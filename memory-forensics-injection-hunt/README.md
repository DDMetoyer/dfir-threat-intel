# Hunting Process Injection in a Windows Memory Image

**Devante Metoyer · October 2024**

A worked methodology for triaging a Windows 8.1 x64 memory capture with **Volatility 2** (run from Kali under WSL). The goal: explain an unexpected outbound connection, find the process that was injected, and identify the process that did the injecting.

> Specific hosts, PIDs and addresses from the original image are left out on purpose. The value is in the workflow, which applies to any capture.

## 1. Network triage: `netscan`

```bash
volatility -f <image>.dmp --profile=Win8SP1x64 netscan -v > netscan.txt
```

`netscan` lists every TCP/UDP endpoint that was live or recently closed at capture time, together with its owning PID and process name. Dumping to a file and searching it is faster than scrolling.

What to look for:
- **Established connections to unusual remote ports.** Port 4444 (the Metasploit default) is a classic red flag.
- **Script hosts holding network sockets.** In this image, an established session to a high remote port was owned by `powershell.exe`, which is worth pivoting on immediately.
- **Inbound sessions on 3389.** An established connection on the host's RDP port, owned by `svchost.exe` (which hosts the Remote Desktop service), shows the machine *received* a remote desktop session. It also gives you the host's internal IP, which recurs as the local address across sessions.

## 2. Injected code: `malfind`

```bash
volatility -f <image>.dmp --profile=Win8SP1x64 malfind -p <suspect PID>
```

`malfind` flags memory regions that are executable but not backed by a file on disk, which is the footprint of injected code. In the suspect PowerShell process, one flagged region **started with the `MZ` signature**, meaning a full PE image had been written into the process's memory. That is far more suspicious than a few bytes of shellcode.

## 3. Confirming the hook: `apihooks`

```bash
volatility -f <image>.dmp --profile=Win8SP1x64 apihooks -p <suspect PID>
```

One of the `malfind` addresses appeared in the `apihooks` output as the destination of an **inline/trampoline hook**. That confirms the injected image was actively intercepting API calls, and the output identifies the base address of the module the hook jumps into.

**Read the hook mode carefully.** `apihooks` reports `Hook mode: Usermode` or `Kernelmode`. An `<unknown>` hooking module in a *user-mode* hook (seen in another case, against `wsmprovhost.exe`, the WinRM plugin host) only affects that user-space process. It says nothing about a kernel rootkit. Mixing those up overstates the compromise.

## 4. Attribution inside the host: `ldrmodules`

```bash
volatility -f <image>.dmp --profile=Win8SP1x64 ldrmodules | grep <module base>
```

`ldrmodules` cross-checks the three loader lists in each process's PEB against the VAD tree, so it shows which processes have a given module mapped, including ones that were unlinked to hide. Searching for the hooked module's base address returned **two processes**: the victim PowerShell process and one other. A second, unrelated process that has the same unbacked module mapped is the most likely injector, and that is the next process to dump and analyze.

## Workflow summary

| Step | Plugin | Question it answers |
|---|---|---|
| 1 | `netscan` | Which process owns the suspicious connection? Was there inbound remote access? |
| 2 | `malfind` | Does that process contain executable, non-file-backed memory (e.g. an `MZ` header)? |
| 3 | `apihooks` | Is that region hooking APIs, and in user mode or kernel mode? |
| 4 | `ldrmodules` | Which other process shares the injected module, and therefore probably injected it? |

## Defensive takeaways

- Alert on script interpreters (`powershell.exe`, `wscript.exe`) making outbound connections to non-standard ports.
- Collect memory early in an incident: injected PE images and hooks often never touch disk.
- Enable PowerShell script-block logging and constrained language mode, and restrict RDP exposure, to cut off the two access paths seen here.

`Volatility 2` · `Windows memory forensics` · `process injection` · `API hooking` · `incident response`
