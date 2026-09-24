# Cyber Threat Intelligence Reports – Florian Eder, MSc

Evidence-first Cyber Threat Intelligence reports on current threats, with a focus on their impact
for organisations in the DACH region.

M.Sc. Business Informatics (Universität Innsbruck) – thesis: *A Comprehensive Analysis of 10 Years
of Cyber Threat Intelligence Research and Practice*.

> ### 📄 Latest report: [Rapuncel – MaaS infostealer distributed through fake GitHub repositories](reports/rapuncel/report.md)
>
> - A fake "LastPass Authenticator" GitHub organisation was one lure of a kit that impersonated
>   at least 40 companies; LastPass itself was not breached.
> - The loader installs **Alinubx.sys**, a driver signed through Microsoft's hardware compatibility
>   program, which kills 145 antivirus and EDR processes from kernel mode.
> - The driver is a renamed copy of the known abusable CcProtect.sys – the rename alone dropped its
>   VirusTotal detections from 7 to 0, and at publication neither variant was on Microsoft's
>   vulnerable driver blocklist.
> - Rapuncel then steals browser credentials (bypassing Chrome/Edge app-bound encryption),
>   crypto wallets, messenger and gaming sessions and Windows Credential Manager contents.
>
> Based on 10 public sources, 36 of 43 facts corroborated by two or more publishers ·
> [full report](reports/rapuncel/report.md) · [ATT&CK layer](reports/rapuncel/attack_layer.json) · [IOCs](reports/rapuncel/iocs.csv)

## Reports

| Date | Report | Focus | Status |
|---|---|---|---|
| 2026-09-24 | [Rapuncel: MaaS infostealer distributed through fake GitHub repositories](reports/rapuncel/report.md) | Infostealer · BYOVD · ATT&CK v19 · IOCs | published |

## Methodology

- **Evidence first** – every factual statement is backed by a verbatim quote from a cited public
  source, checked against an archived copy of that source. A fact counts as *corroborated* only
  when two independent publishers support it.
- **Source rating** – sources are rated on the Admiralty scale; primary research (vendors, the
  affected brand, CERTs) is preferred, news coverage serves as corroboration.
- **Facts vs. assessment** – key judgments carry a confidence level and estimative language; the
  relevance, detection and outlook sections are marked as analyst assessment.
- **Current ATT&CK** – techniques are mapped to MITRE ATT&CK v19; revoked IDs used in vendor
  reports are replaced by their successors. Each report ships an ATT&CK Navigator layer.
- **IOCs** – defanged, with automated reputation lookups (abuse.ch, VirusTotal) at the date of the
  report. Indicators age quickly; re-validate before blocking.

Each report folder contains `report.md`, `attack_layer.json` (ATT&CK Navigator) and `iocs.csv`.

## License

Reports: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) – see [LICENSE](LICENSE).
Short quotations from cited sources remain the property of their respective publishers.

## Disclaimer

Based exclusively on publicly available sources, for educational and defensive purposes.
