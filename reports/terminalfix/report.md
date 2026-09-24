---
title: "TerminalFix: ClickFix variant delivers a reverse tunnel via fake Cloudflare CAPTCHAs"
author: Florian Eder, MSc
date: 2026-09-24
tlp: CLEAR
status: final
---

# TerminalFix: ClickFix variant delivers a reverse tunnel via fake Cloudflare CAPTCHAs

> **TLP:CLEAR** · Author: Florian Eder, MSc · Published: 2026-09-24 · Based on 16 public sources · 30 corroborated / 19 single-source verified claims

## 1. Executive Summary

TerminalFix is a ClickFix variant that Microsoft Threat Intelligence documented on 28 August 2026. Compromised, otherwise legitimate websites show a fake Cloudflare CAPTCHA that copies a PowerShell command to the clipboard and asks the visitor to paste it into Windows Terminal or PowerShell [1][2][3][4]. Unlike the infostealer payloads typical of ClickFix, the chain ends in network-level access: a side-loaded DLL, payloads hidden in PNG images, Active Directory reconnaissance and a Python reverse tunnel over WebSocket/TLS that turns the victim host into a pivot into the internal network [1][4][5][6][7].

In Germany the campaign has a concrete victim. On 4 September 2026 the BSI warned that German institutions had been compromised through TerminalFix and linked the campaign to the group behind the Rhysida ransomware, Vice Spider [5][8]. The same day it tied its work on the Berlin state-network incident to that warning; two Berlin Senate administrations had lost data between 7 and 12 August, and Rhysida published it after Berlin refused to pay [5][7][8][9][10][11][12]. No primary source, however, names TerminalFix as the Berlin entry point explicitly [7].

## 2. Key Judgments

1. **TerminalFix was likely the initial access vector in the August 2026 compromise of two Berlin Senate administrations, but no primary source states this explicitly.** [5][7][8][9][10][11]
   *Confidence: moderate · Likelihood: likely.* The BSI advisory describes a compromised state institution in August whose attack matches TerminalFix, and the BSI's same-day Mastodon post ties its Berlin incident work to that advisory. heise reads this as confirmation. Neither the advisory nor the State of Berlin names Berlin or TerminalFix together, so the link rests on inference.
2. **TerminalFix access is very likely monetised through ransomware and data extortion by the Vice Spider / Rhysida ecosystem rather than through infostealers.** [1][4][5][6][13]
   *Confidence: moderate · Likelihood: very likely.* The chain ends in network access, not credential theft from one host; the BSI reports attempted ransomware and double extortion and ties LoremIpsumLoader in the campaign to the Rhysida group; BlueVoyant independently reports the same group moving to Terminal-based ClickFix lures. Microsoft itself did not observe follow-up activity.
3. **The published hashes and domains will very likely age quickly; detections built on the behaviour - LockScreenContentServer.exe outside SystemApps, pythonw.exe running a tunnel script, a single long-lived WebSocket over 443 - are the more durable control.** [1][2][3][4][5][7][13]
   *Confidence: moderate · Likelihood: very likely.* The operators already switched delivery once after a disruption (signed installers to ClickFix), use a failover domain for payloads and rely on legitimate signed binaries and runtimes whose abuse is visible in behaviour, not in hashes.
4. **German public-sector victims are likely the result of opportunistic, financially motivated targeting rather than a campaign aimed specifically at Germany, so European organisations in all sectors face comparable exposure.** [1][2][3][5][7][12]
   *Confidence: moderate · Likelihood: likely.* Microsoft sees victims across multiple industries, the lure script sat on several hundred websites, the BSI calls the actors opportunistic and financially motivated, and the Rhysida leak site shows no significant German focus.
5. **The Terminal-versus-Run distinction is of limited defensive value; controls and awareness must cover every paste-and-run path, not only the Run dialog.** [1][4][13][14]
   *Confidence: moderate · Likelihood: likely.* Microsoft presents Terminal as the differentiator, while Proofpoint points to Terminal lures since 2024 and BlueVoyant sees another group use the same Terminal paste flow; blocking Win+R alone would leave the TerminalFix path open.

## 3. Timeline

| Date | Event | Sources |
|---|---|---|
| 2026-08-07 | Between 7 and 12 August 2026 the bulk of the data was exfiltrated from two Berlin Senate administrations (Urban Development, Building and Housing; Mobility, Transport, Climate Protection and Environment). | [9][10][11] |
| 2026-08-14 | On 14 August 2026 the IT systems of both Senate administrations were disconnected from the Berlin state network as a precaution. | [7][10] |
| 2026-08-17 | The State of Berlin disclosed an ICT incident in its state network and set up an ICT emergency crisis team; the State Criminal Police Office, the public prosecutor and the BSI are involved. | [7][15] |
| 2026-08-28 | Microsoft Threat Intelligence published its analysis of TerminalFix, a ClickFix variant that uses compromised websites with fake Cloudflare CAPTCHA overlays and targets organisations across multiple industries. | [1][2][3] |
| 2026-08-28 | The attackers tried to extort the State of Berlin, which refused to pay. | [9][10][11] |
| 2026-09-01 | The Senate administration confirmed that passwords for some specialist applications had also been exfiltrated; all 12,000 systems of the State of Berlin were being checked. | [11] |
| 2026-09-04 | The attackers published the copied data on the darknet on 4 September 2026; heise reports about 1.44 million files and a 5.8 TB data package. | [7][10][12] |
| 2026-09-04 | The BSI published advisory BITS-B 2026-287419-1032 on TerminalFix with criticality 2/yellow (measures must be taken promptly), and on the same day stated on Mastodon that it is closely involved in the Berlin incident response, linking the post to the TerminalFix advisory. | [5][8] |

## 4. Threat Overview

### Background

ClickFix lures have so far mostly led to single-host infostealer infections. Microsoft positions TerminalFix as different in two ways: victims are sent to Windows Terminal or PowerShell instead of the Run dialog, which lets longer scripts run reliably, and the payload aims at the network rather than the host [1][4][5][6]. The BSI calls it a further development of ClickFix that can reach deeper into networks and affect entire organisations [1][4][5][6].

Not everyone accepts the new label. Proofpoint's Tommy Madjar points out that Terminal-based lures go back to early ClickFix activity in 2024 and that Microsoft's example does not stand out from known clusters such as ClearFake or ErrTraffic; Malwarebytes likewise notes that steganography alone is not new for ClickFix [6][14].

### Distribution & Initial Access

The lure is a fake Cloudflare Turnstile check shown on top of a compromised website. Clicking it silently copies a PowerShell command to the clipboard; the page then tells the user to open Windows Terminal or PowerShell and paste it [1][5][6]. According to the BSI, victims reach these sites through phishing or social engineering, or simply because they visit them regularly (watering hole), and the CAPTCHA JavaScript appeared on several hundred websites in historical data [5].

### Infection Chain

The pasted command pretends to be a Cloudflare verification, downloads a ZIP archive, unpacks it to C:\ProgramData and prints a fake success message [1][3][5]. The archive contains the signed Windows binary LockScreenContentServer.exe and a malicious dui70.dll; because Windows loads DLLs from the application directory first, the fake DLL runs inside the signed process [1][2][5][7]. The DLL is unsigned, poses as the Windows DirectUI Engine and decodes its payload in memory [1][4].

The second stage downloads three PNG images, rebuilds an executable and a DLL split across two images from the pixel data, and deletes the images [1][4][5][6]. Persistence is set twice under the name LockScreenContentServer_MuODG5yBM, as an HKCU Run key and as a scheduled task every 60 minutes, and the directory is hidden [1][4][5]. The malware then enumerates domain trusts, domain admins, AD users and computers and pings servers with names typical for domain controllers, databases, backup, gateways and mail [1][4][5]; Microsoft and the BSI both read this as an assessment of how valuable the host is [1][5]. A PowerShell loop that executes commands written to a text file provides a simple command channel [1][3]. Finally the attacker brings a signed embeddable Python runtime from python.org and starts the tunnel script client.py through pythonw.exe without a window [1][2][5].

### Capabilities

The tunnel connects outbound over TLS on port 443, upgrades to a WebSocket and relays arbitrary TCP connections for the operator, SOCKS5-style; it multiplexes many connections over one WebSocket, rotates browser User-Agent strings and can be shut down remotely [1][4][5][6]. Together with the reconnaissance results this makes the host a pivot point to any internal system it can reach [1][4][7]. The BSI adds that commands run inside the Python runtime may not be logged, and that attackers may prepare cloud storage such as Azure and exfiltrate with tools like azcopy [5].

Microsoft did not observe hands-on-keyboard activity after the tunnel in the chain it analysed [1][4]. Reports received by the BSI, however, describe attempted ransomware deployment combined with data theft and extortion [5]. Microsoft's description of the reconnaissance script's language coverage is inconsistent: English, Spanish and German in the detailed analysis, English and Spanish in the overview [1].

### Infrastructure

Microsoft names gitnow[.]dev as the tunnel C2 on port 443, two image hosts with failover (bestsocialmedianewspapper[.]com, offlineupdater[.]com) and one compromised website (linked-log[.]com) [1][3][4]. Because the lure sits on legitimate compromised websites and the tunnel uses ordinary TLS on port 443, most of the chain uses infrastructure that reputation filters do not flag by default [1][4][5][6].

### Victimology

Microsoft sees victims across multiple industries [1][2][3]. In Germany, the BSI was informed in August 2026 of a compromised state institution whose attack matched TerminalFix; the advisory's title speaks of German institutions in the plural [5].

In Berlin, the bulk of the data left two Senate administrations (Urban Development, Building and Housing; Mobility, Transport, Climate Protection and Environment) between 7 and 12 August; the systems were cut off on 14 August and a crisis team was set up on 17 August [7][9][10][11][15]. The attackers demanded a ransom, Rhysida claimed the attack with a demand of 30 Bitcoin, and Berlin refused [9][10][11][12]. Passwords for some specialist applications were among the stolen data, and all 12,000 systems of the state were checked [11]. On 4 September the data was published; the State of Berlin describes it as mainly unstructured files from shared and personal staff directories, including personal data of employees, citizens and companies [7][10][12]. The BSI then warned of personalised phishing disguised as Senate e-mail and of identity theft [12][16].

For Rhysida's leak site overall, the BSI reports that 92 % of listings lead to publication, on average after 11 days, with no particular focus on Germany but a strong one on education and health; public administration ranks in the top five [5][7].

### Attribution

The BSI assesses the campaign as opportunistic and purely financially motivated, with no link to state actors [5][12]. Reports it received show LoremIpsumLoader (aka AxolotLoader) in the campaign, a malware the BSI attributes to the group behind Rhysida; it names that group Vice Spider (aka Vice Society, DEV-0832, Vanilla Tempest), active since 2021 and using Rhysida almost exclusively since June 2023 [5]. BlueVoyant, as reported by Dark Reading, independently observed the Lorem Ipsum operators move to ClickFix lures on compromised WordPress sites in late May 2026 - including a paste into Windows Terminal - and links them to the same group under the name Rapid Brigantine [13].

The Berlin link is weaker than the headlines suggest. The BSI advisory speaks only of "a state institution"; on the same day, the BSI's Mastodon post tied its Berlin incident work to the TerminalFix advisory, which heise reads as confirmation that TerminalFix was Rhysida's entry point [5][7][8]. The State of Berlin has not named the attack vector.

## 5. MITRE ATT&CK Mapping

The table combines Microsoft's own mapping with techniques from the BSI advisory. T1204.004 (Malicious Copy and Paste) is the precise technique for the paste step; Microsoft maps it as T1204.002 [1][5][6]. Spearphishing link (T1566.002) and exfiltration to cloud storage (T1567.002) come from the BSI's description of possible steps, not from Microsoft's observed chain [5].

| Tactic | Technique | Evidence |
|---|---|---|
| Initial Access | [T1189](https://attack.mitre.org/techniques/T1189) Drive-by Compromise | [1][5][6] |
| Initial Access | [T1566.002](https://attack.mitre.org/techniques/T1566/002) Phishing: Spearphishing Link | [5] |
| Execution, Persistence, Privilege Escalation | [T1053.005](https://attack.mitre.org/techniques/T1053/005) Scheduled Task/Job: Scheduled Task | [1][4][5] |
| Execution | [T1059.001](https://attack.mitre.org/techniques/T1059/001) Command and Scripting Interpreter: PowerShell | [1][3][5] |
| Execution | [T1204.002](https://attack.mitre.org/techniques/T1204/002) User Execution: Malicious File | [1] |
| Execution | [T1204.004](https://attack.mitre.org/techniques/T1204/004) User Execution: Malicious Copy and Paste | [1][5][6] |
| Execution, Stealth | [T1574.001](https://attack.mitre.org/techniques/T1574/001) Hijack Execution Flow: DLL <br>*cited as T1574.002 (revoked, now T1574.001)* | [1][2][5][7] |
| Persistence, Privilege Escalation | [T1547.001](https://attack.mitre.org/techniques/T1547/001) Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder | [1][4][5] |
| Stealth | [T1027.003](https://attack.mitre.org/techniques/T1027/003) Obfuscated Files or Information: Steganography | [1][4][5][6] |
| Stealth | [T1036.005](https://attack.mitre.org/techniques/T1036/005) Masquerading: Match Legitimate Resource Name or Location | [1][2][5][7] |
| Stealth | [T1070.004](https://attack.mitre.org/techniques/T1070/004) Indicator Removal: File Deletion | [1][4][5][6] |
| Stealth | [T1140](https://attack.mitre.org/techniques/T1140) Deobfuscate/Decode Files or Information | [1][4] |
| Stealth | [T1564.001](https://attack.mitre.org/techniques/T1564/001) Hide Artifacts: Hidden Files and Directories | [1][4][5] |
| Discovery | [T1018](https://attack.mitre.org/techniques/T1018) Remote System Discovery | [1][4][5] |
| Discovery | [T1069.002](https://attack.mitre.org/techniques/T1069/002) Permission Groups Discovery: Domain Groups | [1][4][5] |
| Discovery | [T1082](https://attack.mitre.org/techniques/T1082) System Information Discovery | [1][4][5] |
| Discovery | [T1087.002](https://attack.mitre.org/techniques/T1087/002) Account Discovery: Domain Account | [1][4][5] |
| Discovery | [T1482](https://attack.mitre.org/techniques/T1482) Domain Trust Discovery | [1][4][5] |
| Command And Control | [T1071.001](https://attack.mitre.org/techniques/T1071/001) Application Layer Protocol: Web Protocols | [1][4][5] |
| Command And Control | [T1090](https://attack.mitre.org/techniques/T1090) Proxy | [1][4][5] |
| Command And Control | [T1105](https://attack.mitre.org/techniques/T1105) Ingress Tool Transfer | [1][2][4][5][6] |
| Command And Control | [T1572](https://attack.mitre.org/techniques/T1572) Protocol Tunneling | [1][4][5] |
| Exfiltration | [T1567.002](https://attack.mitre.org/techniques/T1567/002) Exfiltration Over Web Service: Exfiltration to Cloud Storage | [5] |

Mapped to ATT&CK Enterprise v19.2. Navigator layer: [`attack_layer.json`](./attack_layer.json)

## 6. Relevance: European organisations across all sectors; DACH public authorities

*Analyst assessment.*

**For European organisations in any sector** the entry point is ordinary web browsing, not a vulnerability. The lure sits on legitimate, compromised websites, so URL reputation and "known bad domain" lists do little at the first step. What protects is the user not pasting a command, and the endpoint noticing when a signed Windows binary suddenly loads a DLL from C:\ProgramData. Developers and administrators routinely paste install commands into a terminal, so "we blocked Win+R" is not a sufficient answer [14]. Because the tunnel exposes the internal network rather than a single host, one click can become a ransomware incident; the Berlin case shows what that looks like in practice.

**For DACH public authorities** the Berlin incident is the reference case: six days of exfiltration before the systems were cut off, a public ransom demand, and publication of staff directories with personal data of employees, citizens and companies [7][9][10][11]. Authorities have three structural exposures: staff names and structures are partly public, which makes the initial lure easy to tailor; passwords stored on shared or personal drives turn one compromised workstation into access to specialist applications - in Berlin, passwords for specialist applications were among the stolen data [11]; and a leak creates a second wave of targeted phishing that uses real internal documents as pretext. Rhysida's leak site shows no particular focus on Germany, so this is opportunistic crime, not espionage - but the BSI's figure that nine in ten listings end in publication means refusal to pay has to be planned for in advance.

**Regulatory impact:** a TerminalFix compromise with data theft will usually trigger GDPR Art. 33 notification to the supervisory authority and, as in Berlin, Art. 34 information of affected persons. For entities in scope of NIS2 (in Austria the NISG 2026, applicable from 1 October 2026; in Germany the NIS2 implementation act), a compromise that provides network-level access to an attacker may constitute a reportable significant incident, depending on the thresholds.

## 7. Detection & Mitigation

**Prevent the paste.** Tell staff explicitly that no real CAPTCHA asks them to open Terminal, PowerShell or Run and paste something. Where it is workable, restrict PowerShell for standard users (AppLocker / App Control for Business) and enable PowerShell script block logging and Constrained Language Mode, as both Microsoft and the BSI recommend [1][3][5][14]. Do not rely on blocking Win+R alone: TerminalFix deliberately avoids it.

**Hunt for the chain.** The durable signals are behavioural: LockScreenContentServer.exe running from anywhere other than C:\Windows\SystemApps [1][2][4], a scheduled task or Run key named LockScreenContentServer_* [1][4][5], and pythonw.exe running a script called client.py [1][2][5]. An untested Microsoft Defender XDR (KQL) starting point that complements Microsoft's own hunting queries:

```kql
// TerminalFix - behavioural hunt (untested idea, tune before use)
union
  ( DeviceProcessEvents    // signed lock-screen binary outside its home = sideloading host
    | where FileName =~ "LockScreenContentServer.exe"
        and not(FolderPath startswith @"C:\Windows\SystemApps\") ),
  ( DeviceRegistryEvents   // Run-key persistence under the masquerading name
    | where RegistryKey has @"\CurrentVersion\Run"
        and RegistryValueName startswith "LockScreenContentServer_" ),
  ( DeviceEvents           // scheduled-task persistence under the same name
    | where ActionType == "ScheduledTaskCreated"
        and AdditionalFields has "LockScreenContentServer_" ),
  ( DeviceProcessEvents    // windowless Python launching the tunnel script
    | where FileName =~ "pythonw.exe" and ProcessCommandLine has "client.py" )
| project Timestamp, DeviceName, ActionType, FileName, FolderPath, ProcessCommandLine,
          RegistryKey, RegistryValueName, InitiatingProcessFileName, InitiatingProcessCommandLine
```

On the network side, look for a single long-lived TLS/WebSocket session from a workstation to one rarely seen domain, started by pythonw.exe. Microsoft Defender detections to watch for include Trojan:Win32/TermFix, Trojan:Win32/Posilod and Trojan:Python/Indigo.SA [1].

**Response.** Treat an affected host as an entry point into the network, not as an isolated machine: investigate for lateral movement, rotate every credential reachable from it including domain admin accounts, and check shared drives the user could reach for sensitive data and password files [1][4][5]. Commands run through the tunnel may not appear in local logs, so network and identity telemetry matter more than host logs [5].

- Proofpoint's Madjar considers broad Run/Terminal restrictions rarely practical and recommends user training, blocking malicious sites and detecting suspicious execution instead. [14]

## 8. Outlook

*Analyst assessment.*

The paste-and-run technique is cheap, needs no exploit and no code-signing certificate, and has already absorbed one disruption: the Lorem Ipsum operators moved to ClickFix after losing their signing supply [13]. Expect new lure themes (browser updates, meeting fixes, CAPTCHAs), new compromised websites and a replacement for the LockScreenContentServer/dui70 pair once detections catch up; the tunnel concept - a signed runtime plus a small script over TLS 443 - is easy to rebuild. For Germany, two follow-on risks remain after the Berlin leak: targeted phishing that reuses the published documents as pretext, and further public bodies reached through the same lures [12][16]. Watch for an official statement from the State of Berlin on the attack vector, which would settle the TerminalFix link, and for BSI updates to the advisory.


## 9. Indicators of Compromise

IOCs are defanged. Shown are indicators reported by a primary source, an intel database or at least two independent sources. Verdicts come from automated enrichment (ThreatFox, URLhaus, MalwareBazaar, VirusTotal) and reflect the enrichment date – IOCs age quickly, re-validate before blocking.

### DOMAIN

| Indicator | Verdict | VT | Labels | Reported by |
|---|---|---|---|---|
| `gitnow[.]dev` | malicious (high) | 18/91 |  | [1][2][3][4] |
| `bestsocialmedianewspapper[.]com` | malicious (high) | 19/91 |  | [1][2][3] |
| `offlineupdater[.]com` | malicious (high) | 19/91 |  | [1][2][3] |

### SHA256

| Indicator | Verdict | VT | Labels | Reported by |
|---|---|---|---|---|
| `026478003fe354134c03acf6890e7d3b153ba08a836eca42350db48f213872ab` | unknown |  |  | [1][2] |
| `032b529fac61e550f5dc9489686f519b82d64625fa05a8d9ecf8ba8be9b2ad22` | unknown |  |  | [1][2] |
| `18c2090e8a0ae0568af9b87e59eaf8270f23d2909600ed9db91a9444fd8b278f` | unknown |  |  | [1][2] |
| `342df92235c9dec81203b837addaa38bb85b64b4a48fe71b5303ca86d991991e` | unknown |  |  | [1][2] |
| `5d43abf5c36ea203176d3300ff14af27b4be81810ad2679b3a62b255e3d6e1c8` | malicious (high) | 48/75 | trojan.dllhijack/loader | [1][2] |
| `9a7b4dcd51d9251c177d323d6aaecdfc86674f69bc1af048dc872926d22aaa24` | malicious (high) | 48/75 | trojan.dllhijack/loader | [1][2] |
| `b8d107800403b9197e5b7609ceacd8e4cac1b0f9a1d156e6dacd6c3f7794b36a` | malicious (high) | 33/74 | trojan.python/indigo | [1][2] |
| `ba77feed86bcda49308746421bdc684a432dd5d68c363975b2a3c6831bda3f07` | malicious (high) | 46/75 | trojan.dllhijack/loader | [1][2] |
| `df8221a933b38284ebdcb8bffc2df62123c9f5b5f421dd0b070e13e668b3eabf` | malicious (high) | 38/75 | trojan.dllhijack/loader | [1][2] |
| `eb1b4be34d05b394fb74efdeb95faecd1d1963be6ecc1b9db2b4757b491f01f0` | malicious (high) | 47/75 | trojan.dllhijack/loader | [1][2] |
| `ededeacf30e493dd632d477fe770ba419aa2848f685ea049381a0a8d2cc3e84d` | unknown |  |  | [1][2] |

### URL

| Indicator | Verdict | VT | Labels | Reported by |
|---|---|---|---|---|
| `hxxps://linked-log[.]com/` | malicious (high) | 20/91 |  | [1][2] |

Machine-readable: [`iocs.csv`](./iocs.csv)

## 10. Intelligence Gaps

- No primary source explicitly names TerminalFix as the initial access vector in Berlin. The BSI advisory speaks of "a state institution"; the State of Berlin names neither the vector nor Rhysida; the link rests on the BSI Mastodon post and heise's reading of it (see J1).
- The advisory title speaks of German institutions in the plural; which other institutions were affected is not public.
- The BlueVoyant analyses on Lorem Ipsum / Rapid Brigantine (referenced by the BSI) blocked automated retrieval (HTTP 403); that link relies on Dark Reading's summary and was not checked against the original.
- No public IOC overlap links Microsoft's TerminalFix samples to LoremIpsumLoader or to the Berlin intrusion; whether the Berlin chain used the same files and infrastructure is unknown.
- Whether ransomware was actually deployed or executed in Berlin has not been stated by any source; the BSI speaks of attempts in reports it received.
- How the Berlin victim reached the compromised website (phishing e-mail, search, watering hole) is not documented by an official source.
- Number of victims and of compromised websites currently serving the lure is unknown; the BSI only refers to several hundred websites in historical data.
- Microsoft is inconsistent on the locale coverage of the reconnaissance script (English/Spanish vs. English/Spanish/German, C25).
- abuse.ch (ThreatFox, MalwareBazaar) had no entries for the tag TerminalFix at collection time.
- The Register article (S9) could not be extracted and was not used.

## 11. Sources

1. Microsoft Threat Intelligence – "TerminalFix campaign deploys a reverse tunnel through multistage intrusion", 2026-08-28. <https://www.microsoft.com/en-us/security/blog/2026/08/28/terminalfix-campaign-deploys-reverse-tunnel-through-multistage-intrusion/> (accessed 2026-09-24; reliability B, primary)
2. RH-ISAC – "TerminalFix Campaign Targets Multiple Critical Sectors with ClickFix Variant", 2026-09-01. <https://rhisac.org/threat-intelligence/terminalfix-campaign-targets-multiple-critical-sectors-with-clickfix-variant/> (accessed 2026-09-24; reliability B, secondary)
3. The Hacker News – "TerminalFix Uses Fake Cloudflare CAPTCHAs to Deploy Reverse-Tunnel Backdoor", 2026-08-30. <https://thehackernews.com/2026/08/terminalfix-uses-fake-cloudflare.html> (accessed 2026-09-24; reliability C, secondary)
4. BleepingComputer – "Microsoft warns of TerminalFix attacks deploying reverse tunnels", 2026-08-31. <https://www.bleepingcomputer.com/news/security/microsoft-warns-of-terminalfix-attacks-deploying-reverse-tunnels/> (accessed 2026-09-24; reliability B, secondary)
5. BSI – "Deutsche Institutionen über TerminalFix-Kampagne kompromittiert (BITS-B Nr. 2026-287419-1032)", 2026-09-04. <https://www.bsi.bund.de/SharedDocs/Cybersicherheitswarnungen/DE/2026/2026-287419-1032.pdf?__blob=publicationFile&v=2> (accessed 2026-09-24; reliability A, government)
6. Malwarebytes – "TerminalFix looks like ClickFix, but delivers a very different payload", 2026-09-01. <https://www.malwarebytes.com/blog/news/2026/09/terminalfix-looks-like-clickfix-but-delivers-a-very-different-payload> (accessed 2026-09-24; reliability B, primary)
7. heise online – "BSI explains first attack vector on Berlin authorities", 2026-09-07. <https://www.heise.de/en/news/BSI-explains-first-attack-vector-on-Berlin-authorities-11444212.html> (accessed 2026-09-24; reliability B, secondary)
8. BSI – "BSI on Mastodon: involvement in the Berlin incident response and TerminalFix advisory", 2026-09-04. <https://social.bund.de/@bsi/117212729947889443> (accessed 2026-09-24; reliability A, government)
9. Land Berlin – "Der Regierende Bürgermeister Kai Wegner und Innensenatorin Iris Spranger: „Das Land Berlin lässt sich nicht erpressen“", 2026-08-28. <https://www.berlin.de/rbmskzl/aktuelles/pressemitteilungen/2026/pressemitteilung.1708208.php> (accessed 2026-09-24; reliability B, government)
10. Land Berlin – "Cyberangriff auf das Landesnetz: Informationen für Bürgerinnen und Bürger", 2026-09-14. <https://www.berlin.de/cyberangriff/informationen-fuer-buergerinnen-und-buerger/> (accessed 2026-09-24; reliability B, government)
11. heise online – "Berlin: Passwords exfiltrated, 12,000 systems scanned", 2026-09-01. <https://www.heise.de/en/news/Berlin-Passwords-exfiltrated-12-000-systems-scanned-11437104.html> (accessed 2026-09-24; reliability B, secondary)
12. heise online – "BSI warnt nach Daten-Leak vor erhöhter Cyber-Bedrohung", 2026-09-05. <https://www.heise.de/news/BSI-warnt-nach-Daten-Leak-vor-erhoehter-Cyber-Bedrohung-11442510.html> (accessed 2026-09-24; reliability B, secondary)
13. Dark Reading – "'Lorem Ipsum' Malware Pivots to ClickFix Delivery", 2026-06-16. <https://www.darkreading.com/cyberattacks-data-breaches/lorem-ipsum-malware-clickfix-delivery> (accessed 2026-09-24; reliability B, secondary)
14. Dark Reading – "'TerminalFix' Campaign Uses PowerShell for Enterprise Attacks", 2026-08-31. <https://www.darkreading.com/threat-intelligence/terminalfix-campaign-weaponizes-powershell-enterprise-attacks> (accessed 2026-09-24; reliability B, secondary)
15. Land Berlin – "IKT-Vorfall im Landesnetz Berlin", 2026-08-17. <https://www.berlin.de/rbmskzl/aktuelles/pressemitteilungen/2026/pressemitteilung.1703898.php> (accessed 2026-09-24; reliability B, government)
16. BSI – "BSI on Mastodon: follow-up attack risks after the Berlin Senate data theft", 2026-09-13. <https://social.bund.de/@bsi/117262792504611874> (accessed 2026-09-24; reliability A, government)

---

### Appendix: Methodology

This report was produced with a self-developed Python toolchain (not published). It archives every source with its publisher, fetch date and SHA-256 fingerprint; extracts IOCs, ATT&CK technique IDs and CVEs from the source texts; enriches IOCs via abuse.ch and VirusTotal; and maps revoked ATT&CK IDs to their current successors. Every factual statement is recorded with at least one verbatim quote, and the toolchain automatically checks each quote against the archived source text – a statement whose quote cannot be found is not included. Statements are marked *corroborated* when two independent publishers support them. Citations, the ATT&CK table and the IOC tables are generated from these verified records; the analysis is reviewed by the author before publication. Sections 6–8 contain analyst assessments. Source reliability uses the Admiralty scale (A = completely reliable … F = cannot be judged).

**Estimative language:** almost certainly (95–99 %) · very likely (80–95 %) · likely (55–80 %) · roughly even chance (45–55 %) · unlikely (20–45 %) · very unlikely (5–20 %). **Confidence** reflects source quality and corroboration: *high* = multiple independent reliable sources, *moderate* = credible but limited corroboration, *low* = single or questionable source.