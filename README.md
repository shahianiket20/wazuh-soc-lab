# Home SOC Lab: Attack Detection and Incident Investigation with Wazuh

A self-contained Security Operations Center (SOC) lab built on a single laptop, using **Wazuh** (open-source SIEM/XDR) to monitor a Linux endpoint while it is deliberately attacked from a separate machine. The project covers the full detection lifecycle: deploy → attack → detect → investigate → report.


## What this project demonstrates

- Deploying a Wazuh SIEM server and connecting a monitored endpoint
- Simulating realistic attacks (SSH brute force, port scanning, file tampering, privilege escalation)
- Reading and investigating SIEM alerts, mapped to MITRE ATT&CK
- Identifying a real detection gap and attempting a custom detection rule to close it
- Writing SOC-style incident reports

## Lab architecture

```
Kali Linux (Attacker)              Wazuh Server
192.168.64.11                       192.168.64.21
     |                                    ^
     |   SSH brute force (Hydra)          |
     |   Nmap port scan                   | logs / alerts
     |                                    |
     v                                    |
Ubuntu Endpoint  ------------------------->
endpoint1 / 192.168.64.22      (Wazuh agent forwards logs)
```

All three VMs run under **UTM (QEMU)** on an Apple Silicon Mac, on an isolated virtual network. All attacks were run only against machines owned and controlled by the author.

## Tools used

| Tool | Purpose |
|---|---|
| UTM (QEMU) | Virtualization on macOS/ARM64 |
| Ubuntu Server 22.04 (ARM64) | OS for the Wazuh server and endpoint |
| Wazuh 4.14 | SIEM — log collection, alerting, dashboard |
| Kali Linux | Attacker machine |
| Hydra | SSH brute-force testing |
| Nmap | Port scanning / reconnaissance |
| UFW | Host firewall logging (for the custom detection rule) |

## Results summary

| Test | Attack | Result |
|---|---|---|
| T1 | SSH Brute Force (Hydra) | **Detected** — rule 5763, MITRE T1110 |
| T2 | Nmap Port Scan | **Not detected** by default rules — documented gap |
| T3 | File Integrity Monitoring (create/modify/delete) | **Detected** — all 3 actions logged |
| T4 | Privilege Escalation (sudo) | **Detected** — rule 5402, full command logging |

### Detection gap: Nmap port scan

Wazuh's default rules don't fire on a plain port scan (no auth failure or file event is generated). I enabled UFW firewall logging and wrote a custom frequency-based rule ([`local_rules.xml`](./local_rules.xml)) to flag 8+ blocked connections from one source within 10 seconds, mapped to MITRE T1046. The underlying log events were confirmed (via `wazuh-logtest`) to be collected and decoded correctly, but the rule did not reliably fire in live testing. Documented as an open item — a dedicated network IDS (e.g. Suricata) would be the more appropriate fix in a production setting.

## Repository contents

```
├── README.md
├── Home_SOC_Lab_Wazuh_Report.docx   # Full write-up: methodology, testing, incident reports
├── local_rules.xml                  # Custom Wazuh detection rule (port scan attempt)
└── screenshots/                     # Evidence for each attack and alert
```

## Full report

See [`Home_SOC_Lab_Wazuh_Report.docx`](./Home_SOC_Lab_Wazuh_Report.pdf) for the complete write-up, including methodology, architecture, incident reports, and lessons learned (including real troubleshooting: package install failures, duplicate VM MAC addresses, and the custom rule debugging process).

## Author

**Aniket Shahi** 
