---
title: "Rapuncel: MaaS infostealer distributed through fake GitHub repositories"
author: Florian Eder, MSc
date: 2026-09-24
tlp: CLEAR
status: final
---

# Rapuncel: MaaS infostealer distributed through fake GitHub repositories

> **TLP:CLEAR** · Author: Florian Eder, MSc · Published: 2026-09-24 · Based on 10 public sources · 36 corroborated / 7 single-source verified claims

## 1. Executive Summary

Rapuncel is a Windows infostealer that LastPass and Delphos Labs uncovered after a fake "LastPass Authenticator" organisation appeared on GitHub. The operation was not limited to LastPass: the same kit impersonated at least 40 companies [1][2][3]. LastPass itself was not breached; only its brand was used [1][3].

What makes the campaign notable is not the stealer but the driver that runs before it. The loader installs Alinubx.sys, a kernel driver signed through Microsoft's hardware compatibility program, which terminates 145 antivirus and EDR processes from kernel mode [1][2][4][5]. The driver is a renamed copy of a known abusable driver, CcProtect.sys, and the rename was enough to drop its VirusTotal detections to zero [1][5][6]. At publication neither variant was on Microsoft's vulnerable driver blocklist [1][5][7].

Once defences are down, Rapuncel collects browser credentials (including Chrome and Edge app-bound encrypted data), cryptocurrency wallets, messenger and gaming sessions, and Windows Credential Manager contents, and uploads them to a single exfiltration server [1][2][3][7]. Researchers place it in the BoryptGrab ecosystem of SEO-poisoned GitHub lures with moderate confidence, without naming an operator [1][3].

## 2. Key Judgments

1. **The operators will very likely continue the campaign with new brands, infrastructure and a renamed or recompiled driver, so the hash and domain IOCs in this report will age quickly.** [1][2][3][5][6][8][9]
   *Confidence: moderate · Likelihood: very likely.* One kit serves 40+ brands, the redirect layer lets the operator swap payload servers at will, infrastructure was maintained after detection, and the driver was already re-identified once (CcProtect -> Alinubx). A parallel BoryptGrab-lineage wave with 292 repositories shows the model scales.
2. **Controls that rely on hashes, file names or reputation (VirusTotal, the Microsoft blocklist, hash-based LOLDrivers rules) will almost certainly miss the next variant of this driver; signer- and lineage-based driver blocking is the control that holds.** [1][5][6]
   *Confidence: high · Likelihood: almost certainly.* A filename and identity swap alone dropped VirusTotal detections from 7 to 0, neither the original nor the renamed driver was blocklisted, and the Microsoft attestation remained valid.
3. **Rapuncel is likely one strand of a broader, Russian-speaking-linked BoryptGrab ecosystem of SEO-poisoned GitHub lures rather than an isolated campaign; attribution to a specific actor is not possible on current evidence.** [1][3][5][7][9][10]
   *Confidence: moderate · Likelihood: likely.* Shared lure brand, identical collection artifacts and fake badge texts connect the waves; the Russian-language indicators come from the related BoryptGrab reporting, not from Rapuncel itself, and the loader is a commercial crypter that several crews can buy.
4. **The kernel-level stage is likely to succeed mainly where users work with local administrator rights; enforcing standard-user accounts sharply reduces, but does not remove, the risk.** [1][2][4][5]
   *Confidence: moderate · Likelihood: likely.* The loader needs elevation (UAC bypass techniques) before it can install the driver, and the researchers state that administrator rights are enough to install and use the trusted driver. Whether any of the three elevation methods works from a standard-user context is not documented.

## 3. Timeline

| Date | Event | Sources |
|---|---|---|
| 2026-08-13 | LastPass TIME identified a fraudulent GitHub organisation impersonating LastPass Authenticator. | [1][3][8] |
| 2026-08-19 | Delphos reported Alinubx.sys to Microsoft (MSRC); MSRC did not classify it as a vulnerability because the driver is not a Microsoft component and redirected the report to the driver submission (blocklist) channel. | [1][8] |
| 2026-08-20 | Neither Alinubx.sys nor the original CcProtect.sys was on Microsoft's vulnerable driver blocklist, and Alinubx.sys had 0/72 VirusTotal detections. | [1][5] |
| 2026-09-10 | The traffic-director endpoint was still live and its content had changed between 27 August and 10 September, showing active maintenance of the campaign. | [1][8] |
| 2026-09-17 | LastPass and Delphos Labs published their joint report; at publication Alinubx.sys was still not on the Microsoft vulnerable driver blocklist. | [1][7] |

## 4. Threat Overview

### Background

Fake GitHub repositories are an established delivery channel for this stealer family. Trend Micro documented BoryptGrab in March 2026, spread through more than a hundred SEO-optimised GitHub repositories, with Russian-language artefacts in the code [7][10]. In July 2026 Arctic Wolf reported a separate wave of at least 292 brand-impersonation repositories delivering a BoryptGrab-lineage stealer, using the same fake "VirusTotal Approved" and "Secure Archive" badges seen in the LastPass lure [1][5][7][8][9].

### Distribution & Initial Access

The lure relies on search engines, not e-mail. Users searching for "LastPass Authenticator download" found a fake GitHub organisation near the top of the results, styled as an official product page [1][5]; a second page for a "macOS LastPass" product was taken down before analysis [1][5]. The download button led to a GitHub Pages portal with fabricated trust badges [1][7][8].

Behind the portal, two further GitHub Pages accounts with custom 404.html pages silently forwarded the victim, abusing GitHub's error handling as a covert redirector [1][7][8]. The final ZIP archives were 148 MB and 127.9 MB, padded with junk DLLs so that size-limited scanners skip them [1][2].

### Infection Chain

The "installer" is Microsoft's own debugger, vsdbg.exe, renamed. When run, it side-loads the malicious vsdbg.dll placed next to it [1][2][3]. The loader attempts three elevation methods and runs as SYSTEM [1][5]; it matches the Cruciferra PUROSANGUE crypter feature set, including process hollowing [1][2].

With SYSTEM rights the loader drops the driver as nvfsflt64.sys and registers it as the service NvFsFilter, posing as an NVIDIA file system filter [1][2]. The malware persists as an auto-start service that, on every boot, kills security products that have restarted and re-runs the stealer [1][2].

### Capabilities

Alinubx.sys exposes the device `\\.\Alinubx`. The loader sends IOCTL 0x222024 with process IDs, and the driver terminates each one with ZwTerminateProcess; the loader carries a hardcoded list of 145 AV/EDR process names [1][2][4]. Delphos states that opening processes with kernel-mode access lets it defeat Protected Process Light [1]; LOLDrivers, however, notes that successful termination of protected processes is not established [4].

The driver is signed through the Microsoft Windows Hardware Compatibility Publisher chain with a March 2023 timestamp [1][5]. It is an identity-swapped CcProtect.sys v1.32 from the Chinese CnCrypt product, a driver already catalogued as a BYOVD process killer with public proof-of-concept code [1][6]. Renaming it cut VirusTotal detections from 7 to 0; by 10 September three engines flagged it [1][5]. The driver also contains rootkit, DLL-injection and traffic-redirection features that need a configuration file the operators did not supply, so they stayed dormant in this deployment [1][2].

The stealer reads saved passwords from browsers and bypasses Chrome/Edge app-bound encryption by injecting a helper DLL that calls the browser's own decryption service [1][2][3]. It also collects wallet files, Discord, Steam and Telegram sessions, Credential Manager contents, documents with names like "password" or "seed", screenshots of every monitor and a system profile [1][2][7]. The primary report gives two different target counts for browsers and wallets [1].

### Infrastructure

The chain is built for rotation. A Cloudflare-fronted traffic director, istatlmenus[.]com, supplies the payload server at runtime, so the operator can swap servers without touching the GitHub lures [1][8]; its content changed between 27 August and 10 September [1][8]. The payload server albinofennel[.]com served lure pages for at least 40 brands from a templated JavaScript kit [1][5]. During the analysis one payload server dropped off DNS while two stayed live, and the terminal redirect domains were parked but still registered [1].

Exfiltration goes to a single IP, 2.26.126[.]50, as a ZIP upload formatted as HTTP "POST /upload" over raw TCP; the loader and driver themselves do not contact external infrastructure [1][2].

### Victimology

Targeting is opportunistic: whoever searches for a lured product and runs the download is a victim. The roughly 40 impersonated brands have not been named, and no victim count has been published [1][7].

### Attribution

No threat actor has been named. Delphos assesses with high confidence that the loader was built with the commercial Cruciferra PUROSANGUE crypter or a close derivative [1][2]. It places the stealer in the BoryptGrab ecosystem with moderate confidence, citing a shared lure brand (passathook-cs2) and identical collection artefacts, but finds the evidence insufficient to call it the same campaign or operator [1][3][10]. The Russian-language indicators in the wider ecosystem come from the BoryptGrab and Arctic Wolf reporting, not from the Rapuncel samples [5][7][9][10].

## 5. MITRE ATT&CK Mapping

The mapping below uses ATT&CK v19. Three rows, T1014 Rootkit, T1055.004 APC injection and T1557 Adversary-in-the-Middle, come from the primary report's list of driver features that are present in code but were not activated in this deployment [1][2]. They describe what the driver can do, not what was observed.

| Tactic | Technique | Evidence |
|---|---|---|
| Resource Development | [T1608.006](https://attack.mitre.org/techniques/T1608/006) Stage Capabilities: SEO Poisoning | [1][5] |
| Execution | [T1204.002](https://attack.mitre.org/techniques/T1204/002) User Execution: Malicious File | [1][2][3] |
| Execution, Stealth | [T1574.001](https://attack.mitre.org/techniques/T1574/001) Hijack Execution Flow: DLL <br>*cited as T1574.002 (revoked, now T1574.001)* | [1][2][3] |
| Persistence, Privilege Escalation | [T1543.003](https://attack.mitre.org/techniques/T1543/003) Create or Modify System Process: Windows Service | [1][2] |
| Privilege Escalation, Stealth | [T1055.004](https://attack.mitre.org/techniques/T1055/004) Process Injection: Asynchronous Procedure Call | [1] |
| Privilege Escalation, Stealth | [T1055.012](https://attack.mitre.org/techniques/T1055/012) Process Injection: Process Hollowing | [1] |
| Privilege Escalation | [T1548.002](https://attack.mitre.org/techniques/T1548/002) Abuse Elevation Control Mechanism: Bypass User Account Control | [1][5] |
| Stealth | [T1014](https://attack.mitre.org/techniques/T1014) Rootkit | [1] |
| Stealth | [T1027.001](https://attack.mitre.org/techniques/T1027/001) Obfuscated Files or Information: Binary Padding | [1][2] |
| Stealth | [T1036.005](https://attack.mitre.org/techniques/T1036/005) Masquerading: Match Legitimate Resource Name or Location | [1][2][3] |
| Defense Impairment | [T1553.002](https://attack.mitre.org/techniques/T1553/002) Subvert Trust Controls: Code Signing | [1][5] |
| Defense Impairment | [T1685](https://attack.mitre.org/techniques/T1685) Disable or Modify Tools <br>*cited as T1562.001 (revoked, now T1685)* | [1][2][4] |
| Credential Access | [T1539](https://attack.mitre.org/techniques/T1539) Steal Web Session Cookie | [1][2][7] |
| Credential Access | [T1555.003](https://attack.mitre.org/techniques/T1555/003) Credentials from Password Stores: Credentials from Web Browsers | [1][2][3] |
| Credential Access | [T1555.004](https://attack.mitre.org/techniques/T1555/004) Credentials from Password Stores: Windows Credential Manager | [1][2][7] |
| Credential Access, Collection | [T1557](https://attack.mitre.org/techniques/T1557) Adversary-in-the-Middle | [1] |
| Discovery | [T1082](https://attack.mitre.org/techniques/T1082) System Information Discovery | [1][2][7] |
| Collection | [T1005](https://attack.mitre.org/techniques/T1005) Data from Local System | [1][2][7] |
| Collection | [T1113](https://attack.mitre.org/techniques/T1113) Screen Capture | [1][2][7] |
| Collection | [T1560](https://attack.mitre.org/techniques/T1560) Archive Collected Data | [1][2] |
| Exfiltration | [T1041](https://attack.mitre.org/techniques/T1041) Exfiltration Over C2 Channel | [1][2] |

Mapped to ATT&CK Enterprise v19.2. Navigator layer: [`attack_layer.json`](./attack_layer.json)

## 6. Relevance: DACH organisations; employees downloading software via search engines

*Analyst assessment.*

**For DACH organisations** the risk is not a targeted attack but ordinary software downloads. Any employee who searches for a tool, whether a password manager, a utility or a game mod, can reach one of the ~40 brand lures. The lure defeats the checks users are trained to make: it sits on github.com, ranks high in search results, and displays "VirusTotal Approved" badges. The driver stage then removes the EDR that most mid-sized organisations rely on as their last line of defence. Because the impersonated brands are unknown, organisations cannot rule out that tools common in German-speaking markets are among them.

The kill chain depends on elevation. Endpoints where users hold local administrator rights, typically on freelancer and contractor laptops, in small IT teams and on developer machines, are at the highest risk. On such a machine the attacker has already stolen browser-stored credentials, including SSO sessions, before any alert could fire.

**For the event and ticketing sector** exposure is above average for three reasons. Production and box-office staff frequently install tools ad hoc on shared or personal laptops at venues, often with admin rights. Ticketing, payment-provider and social-media accounts are typically saved in the browser. A stolen session there can mean fraudulent ticket sales, payout redirection or account takeover on the channels used for customer communication. Seasonal staff are rarely covered by software-installation policies.

Regulatory impact: stolen browser credentials that give access to customer or payment data may trigger GDPR Art. 33 notification duties. For entities in scope of NIS2 (in Austria the NISG 2024, in Germany the NIS2 implementation act), a kernel-level compromise of an endpoint with privileged access is likely a reportable significant incident.

## 7. Detection & Mitigation

**Prevent the driver, not the file.** Enable Microsoft's vulnerable driver blocklist, but do not rely on it: it is hash-based and did not cover this driver [1][5]. Where WDAC / App Control for Business is available, add a deny rule on driver file attributes and signer: ProductName "CnCrypt", OriginalFileName "CcProtect.sys" and "Alinubx.sys", and the Henan Dafeng signer [1][5]. The product name survived the rename, so it covers the lineage rather than a single hash. Remove local administrator rights from standard users; the kernel stage requires elevation.

**Hunt for behaviour.** The durable signals are the NvFsFilter service, writes of nvfsflt64.sys to the drivers folder, handles to the `\\.\Alinubx` device, and a burst of security-process terminations right after a driver load [1][5]. Further indicators are a renamed vsdbg.exe spawning non-Microsoft child processes, PE files with an oversized .reloc section, and the files browser_decryption.log / sends.log in %TEMP% [1]. An untested Microsoft Defender XDR (KQL) starting point:

```kql
// Rapuncel / Alinubx - behavioural hunt (untested idea, tune before use)
union
  ( DeviceProcessEvents   // renamed Microsoft debugger used as fake installer
    | where InitiatingProcessVersionInfoOriginalFileName =~ "vsdbg.exe"
        and InitiatingProcessFileName !~ "vsdbg.exe" ),
  ( DeviceFileEvents      // driver dropped under NVIDIA-looking name
    | where FileName =~ "nvfsflt64.sys" and FolderPath has @"\System32\drivers" ),
  ( DeviceRegistryEvents  // service registration of the masquerading driver
    | where RegistryKey has @"\CurrentControlSet\Services\NvFsFilter" ),
  ( DeviceEvents          // any driver load of the lineage
    | where ActionType == "DriverLoad" and FileName in~ ("nvfsflt64.sys", "alinubx.sys", "ccprotect.sys") )
| project Timestamp, DeviceName, ActionType, FileName, FolderPath, RegistryKey,
          InitiatingProcessFileName, InitiatingProcessFolderPath
```

On endpoints where the EDR sensor was killed, this telemetry may be missing. Absence of alerts is therefore not evidence of a clean host. Check for gaps in sensor heartbeats as well. Proxy and DNS logs are unaffected by the kernel driver: look up the kit domains in the IOC list, and look for raw HTTP-formatted POSTs to 2.26.126[.]50. The researchers themselves advise treating that IP as historical context, not a standalone block rule [1].

**Response.** Treat an affected host as a kernel-level compromise: isolate, image, and rebuild or remediate offline. Rotate all browser-stored credentials, sessions and Credential Manager contents from a clean device, revoke SSO and OAuth sessions, and move any crypto assets [1][3].

**Awareness.** Tell staff explicitly that GitHub is not a download site for commercial software and that "VirusTotal Approved" badges on a download page are a red flag. Provide software through a company portal or winget/Intune so searching is unnecessary.


## 8. Outlook

*Analyst assessment.*

The operators lost one brand lure and some infrastructure, not their kit. The redirect layer and the templated lure server are designed for exactly this situation, and the driver has already been re-identified once. Expect the same pattern with a new driver name, new domains and new brands within weeks rather than months. Two developments would raise the threat level: operators supplying the Alinubx.ccf configuration, which would activate the rootkit and traffic-redirection features, or the stealer being paired with a persistent backdoor as in earlier BoryptGrab waves [1][10]. Defenders should watch for Microsoft adding the CnCrypt lineage to the blocklist and for new LOLDrivers entries sharing the 0x222024 kill primitive.


## 9. Indicators of Compromise

IOCs are defanged. Shown are indicators reported by a primary source, an intel database or at least two independent sources; 12 further candidate(s) mentioned by a single secondary source are only in [`iocs.csv`](./iocs.csv). Verdicts come from automated enrichment (ThreatFox, URLhaus, MalwareBazaar, VirusTotal) and reflect the enrichment date – IOCs age quickly, re-validate before blocking.

### IPV4

| Indicator | Verdict | VT | Labels | Reported by |
|---|---|---|---|---|
| `2[.]26[.]126[.]50` | malicious (high) | 16/91 |  | [1][2] |

### SHA256

| Indicator | Verdict | VT | Labels | Reported by |
|---|---|---|---|---|
| `5f0cfe8357bb52b45068ddbac053e32bc38e6cb5e086746f5402657b0a5cfb1c` | malicious (high) | 7/75 | hacktool.vulndriver/ccprotect | [1][6] |
| `611b3ba687b7f46319a19609605ddfe5225e6d85277d8e923eea3fdb6f7b5b61` | malicious (medium) | 3/75 | vulndriver/cncrypt | [1][4] |
| `1e6c1766ac78d7adfdae71d361cb132d972771897ae9065503b135cb812d7c35` | known, no detections | 0/75 |  | [1] |
| `26db14b956e33f69b3397a36387d32e01eb63613acff91069dc76b6ed7de45a8` | malicious (high) | 30/75 | trojan.driverloader/genericfca | [1] |
| `75018b06c7105a1dca391805d17b402aed35ebd515b92d461236eafbd606cb40` | malicious (high) | 48/75 | trojan.stealer/loregun | [1] |
| `aefbc6e04320e9a0e80f2323f8a897c4fdb222a37b0b87d76e850109decbfadd` | malicious (high) | 50/75 | trojan.stealer/encoder | [1] |
| `ea8c31a86fa785ab514022c278a2f6e571c86aac9283745a96605c44d88382d6` | unknown |  |  | [1] |

### DOMAIN

| Indicator | Verdict | VT | Labels | Reported by |
|---|---|---|---|---|
| `albinofennel[.]com` | malicious (high) | 9/91 |  | [1] |
| `dallikilic54[.]github[.]io` | malicious (medium) | 1/91 |  | [1] |
| `edgarcostartqd[.]github[.]io` | malicious (medium) | 1/91 |  | [1] |
| `hanselarinmusky[.]com` | malicious (high) | 7/91 |  | [1] |
| `icansamyope[.]com` | malicious (high) | 7/91 |  | [1] |
| `istatlmenus[.]com` | malicious (high) | 8/91 |  | [1] |
| `lastpass-authenticator[.]github[.]io` | malicious (medium) | 1/91 |  | [1] |
| `macperformancetools[.]com` | malicious (high) | 8/91 |  | [1] |
| `ryanpresbrey[.]cc` | malicious (high) | 5/91 |  | [1] |
| `zaffersnouty[.]com` | malicious (high) | 14/91 |  | [1] |

Machine-readable: [`iocs.csv`](./iocs.csv)

## 10. Intelligence Gaps

- No victim count is published and the ~40 impersonated brands are not named; it is unknown whether DACH-specific software or brands were among the lures.
- The Delphos Labs companion analysis and the SOCFortress write-up (blocked, HTTP 403) were not stored as sources; technical details rely on the LastPass report, which incorporates the joint findings.
- The list of 145 targeted AV/EDR process names is not published, so it is unknown which products common in DACH organisations are covered.
- PPL bypass: Delphos states the driver can defeat Protected Process Light, while LOLDrivers states that successful termination of protected processes is not established. Not independently tested.
- The LastPass report contains inconsistent target counts (25+/30+ vs. 19+/34+ browsers/wallets).
- Which three elevation methods the loader uses, and whether any works for standard users, is not documented.
- Start date of the Rapuncel wave is unknown (the report says only that the campaign has been running for months).
- Purpose of the Chrome PreferenceMACs registry-key deletion seen in sandbox runs of the terminal domains is unclear.
- abuse.ch (ThreatFox, MalwareBazaar) had no entries for Rapuncel or Alinubx at collection time; hashes come only from the LastPass report and LOLDrivers and were not independently analysed.
- Whether Microsoft has added Alinubx.sys or the CcProtect lineage to the vulnerable driver blocklist since 17 September 2026 was not checked.

## 11. Sources

1. LastPass TIME – "Threat Intel | One Kit, Forty Companies: How a Malware-as-a-Service Platform Used GitHub as a Distribution Network for its Campaign - The LastPass Blog", 2026-09-17. <https://blog.lastpass.com/posts/lastpass-delphos-report-rapuncel-infostealer> (accessed 2026-09-24; reliability B, primary)
2. BleepingComputer – "Fake LastPass Authenticator GitHub repos push new Rapuncel infostealer", 2026-09-18. <https://www.bleepingcomputer.com/news/security/fake-lastpass-authenticator-github-repos-push-new-rapuncel-infostealer/> (accessed 2026-09-24; reliability B, secondary)
3. CyberInsider – "Fake LastPass downloads on GitHub pushed password-stealing malware", 2026-09-18. <https://cyberinsider.com/fake-lastpass-downloads-on-github-pushed-password-stealing-malware/> (accessed 2026-09-24; reliability C, secondary)
4. LOLDrivers – "Alinubx.sys — LOLDrivers", 2026-08-27. <https://www.loldrivers.io/drivers/84a3007a-de5e-4622-bfc5-f05d927c3618/> (accessed 2026-09-24; reliability B, database)
5. The Hacker News – "Fake LastPass Authenticator Installer Abuses Microsoft-Signed Driver to Kill Antivirus and EDR", 2026-09-21. <https://thehackernews.com/2026/09/fake-lastpass-authenticator-installer.html> (accessed 2026-09-24; reliability C, secondary)
6. LOLDrivers – "CcProtect.sys — LOLDrivers", 2026-06-16. <https://www.loldrivers.io/drivers/3e3067b0-3d74-46fe-9f57-1ae3a0293958/> (accessed 2026-09-24; reliability B, database)
7. heise online – "LastPass discovers new infostealer variant disguised as GitHub repo", 2026-09-19. <https://www.heise.de/en/news/LastPass-discovers-new-infostealer-variant-disguised-as-GitHub-repo-11459182.html> (accessed 2026-09-24; reliability B, secondary)
8. Security Affairs – "Fake LastPass on GitHub Led to an Infostealer That Killed 145 Security Tools", 2026-09-23. <https://securityaffairs.com/199577/malware/fake-lastpass-on-github-led-to-an-infostealer-that-killed-145-security-tools.html> (accessed 2026-09-24; reliability C, secondary)
9. Arctic Wolf Labs – "Malicious GitHub Campaign: Fake "Arctic Wolf" and 290+ Brand-Impersonation Repositories Deliver BoryptGrab-Lineage Infostealer - Arctic Wolf", 2026-07-13. <https://arcticwolf.com/resources/blog/fake-github-repositories-deliver-boryptgrab-lineage-infostealer/> (accessed 2026-09-24; reliability B, primary)
10. Trend Micro Research – "New BoryptGrab Stealer Targets Windows Users via Deceptive GitHub Pages", 2026-03-05. <https://www.trendmicro.com/en_us/research/26/c/boryptgrab-stealer-targets-users-via-deceptive-github-pages.html> (accessed 2026-09-24; reliability B, primary)

---

### Appendix: Methodology

Sources were archived with a SHA-256 fingerprint. Every factual statement is backed by at least one verbatim quote that was checked against the archived source text, and statements are marked *corroborated* when two independent publishers support them. Sections 6–8 contain analyst assessments. Source reliability uses the Admiralty scale (A = completely reliable … F = cannot be judged).

**Estimative language:** almost certainly (95–99 %) · very likely (80–95 %) · likely (55–80 %) · roughly even chance (45–55 %) · unlikely (20–45 %) · very unlikely (5–20 %). **Confidence** reflects source quality and corroboration: *high* = multiple independent reliable sources, *moderate* = credible but limited corroboration, *low* = single or questionable source.