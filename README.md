# Cyber Threat Intelligence Reports – Florian Eder, MSc

Evidence-first Cyber Threat Intelligence reports on current threats, with a focus on their impact
for organisations in the DACH region.

M.Sc. Business Informatics (Universität Innsbruck) – thesis: *A Comprehensive Analysis of 10 Years
of Cyber Threat Intelligence Research and Practice*.

> ### 📄 Latest report: [TerminalFix – ClickFix variant delivers a reverse tunnel via fake Cloudflare CAPTCHAs](reports/terminalfix/report.md)
>
> - Compromised websites show a fake Cloudflare CAPTCHA and have victims paste a PowerShell command
>   into Windows Terminal instead of the Run dialog (Microsoft Threat Intelligence, 28 August 2026).
> - The chain side-loads a malicious dui70.dll into the signed LockScreenContentServer.exe, pulls
>   payloads hidden in PNG images, maps Active Directory and installs a Python reverse tunnel over
>   TLS 443 that turns the victim into a pivot into the internal network.
> - The BSI (4 September 2026) warns that German institutions were compromised and links the
>   campaign via LoremIpsumLoader to the group behind the Rhysida ransomware (Vice Spider).
> - The BSI ties its work on the Berlin Senate incident to this warning; no primary source names
>   TerminalFix as the Berlin entry point explicitly – the report assesses it as likely.
>
> Based on 16 public sources, 30 of 49 facts corroborated by two or more publishers ·
> [full report](reports/terminalfix/report.md) · [ATT&CK layer](reports/terminalfix/attack_layer.json) · [IOCs](reports/terminalfix/iocs.csv) · [evidence](reports/terminalfix/claims.yaml)

## Reports

| Date | Report | Focus | Status |
|---|---|---|---|
| 2026-09-24 | [TerminalFix: ClickFix variant delivers a reverse tunnel via fake Cloudflare CAPTCHAs](reports/terminalfix/report.md) | ClickFix · DLL sideloading · reverse tunnel · DACH public sector | published |
| 2026-09-24 | [Rapuncel: MaaS infostealer distributed through fake GitHub repositories](reports/rapuncel/report.md) | Infostealer · BYOVD · ATT&CK v19 · IOCs | published |

## Methodology

Reports are produced with a self-developed Python toolchain (not published) that archives the
sources, extracts and enriches IOCs, maps ATT&CK techniques and automatically checks every quoted
piece of evidence against the archived source text before the report is rendered.

- **Evidence first** – every factual statement is backed by a verbatim quote from a cited public
  source; the toolchain verifies each quote against an archived copy of that source. A fact counts as *corroborated* only
  when two independent publishers support it.
- **Source rating** – sources are rated on the Admiralty scale; primary research (vendors, the
  affected brand, CERTs) is preferred, news coverage serves as corroboration.
- **Facts vs. assessment** – key judgments carry a confidence level and estimative language; the
  relevance, detection and outlook sections are marked as analyst assessment.
- **Current ATT&CK** – techniques are mapped to MITRE ATT&CK v19; revoked IDs used in vendor
  reports are replaced by their successors. Each report ships an ATT&CK Navigator layer.
- **IOCs** – defanged, with automated reputation lookups (abuse.ch, VirusTotal) at the date of the
  report. Indicators age quickly; re-validate before blocking.

Each report folder contains `report.md`, `attack_layer.json` (ATT&CK Navigator), `iocs.csv`,
`claims.yaml` (every factual statement with its verbatim quotes) and `sources.csv` (source
list with publisher, reliability rating and SHA-256 fingerprint of the archived copy).

## License

Reports: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) – see [LICENSE](LICENSE).
Short quotations from cited sources remain the property of their respective publishers.

## Disclaimer

Based exclusively on publicly available sources, for educational and defensive purposes.
