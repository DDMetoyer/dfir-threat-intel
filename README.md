# DFIR & Threat Intelligence

Digital forensics, incident response and cyber threat intelligence write-ups.

## Contents

### [Fancy Bear (APT28): Threat Intelligence Report](apt28-threat-intel-report/)
An intelligence report on APT28: organizational alignment (GRU Unit 26165), targeting, motives, tactics mapped to the kill chain, and operational indicators for defenders. It ends with an **Analysis of Competing Hypotheses (ACH)** matrix that weighs six candidate intrusion sets (APT28, APT29, Sandworm, APT37, APT33 and eCrime) against seven pieces of evidence from an aerospace intrusion scenario.

### [Hunting Process Injection in a Windows Memory Image](memory-forensics-injection-hunt/)
A Volatility 2 workflow for going from "an unexpected outbound connection" to "this process injected that DLL": `netscan` → `malfind` → `apihooks` → `ldrmodules`. It also covers reading hook output correctly (user mode vs. kernel mode) and spotting an inbound RDP session.

## Skills

`Volatility` · `memory forensics` · `process injection` · `API hooking` · `threat intelligence` · `ACH` · `Cyber Kill Chain` · `indicators of compromise`

## Author

**Devante Metoyer** · [LinkedIn](https://linkedin.com/in/devantedmetoyer) · [GitHub](https://github.com/DDMetoyer)

Text licensed under [CC BY 4.0](LICENSE).
