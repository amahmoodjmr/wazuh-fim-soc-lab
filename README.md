# Wazuh SIEM and File Integrity Monitoring Lab

Real-time File Integrity Monitoring (FIM) of a Windows endpoint with Wazuh 4.12.0.
SOC technical laboratory for the **ICDFA Nigeria Ethical Hacking Internship**.

| | |
|---|---|
| **Author** | Abubakar Mahmood Muhammad |
| **Practical date** | 16 October 2026 |
| **Role in the scenario** | Level 1 SOC Analyst |
| **Platform** | Wazuh Manager, Indexer, Dashboard and Windows agent, all 4.12.0 |
| **Report** | [`report/`](report/) (Word and PDF) |

## What this lab is about

The scenario: a CISO is worried about unauthorised changes to sensitive files on employee workstations, and about the organisation's limited ability to detect and investigate them. The task is to build and validate a centralised monitoring setup with Wazuh and show that it catches those changes.

In short, an agent on a Windows machine watches a `C:\SensitiveFiles` folder in real time and reports every change to a Wazuh server. A test file is created, modified and deleted, and the resulting alerts are investigated.

A fuller explanation of the ideas (SIEM, FIM, how an alert is read) is in [`docs/LAB_EXPLANATION.md`](docs/LAB_EXPLANATION.md).

## Environment

| Item | Detail |
|---|---|
| Hypervisor | VMware Workstation 17, Windows 11 host with 8 GB RAM |
| Wazuh server | VM `icdfa-server`, Ubuntu 20.04.6 LTS, `192.168.48.132/24` (VMware NAT) |
| Server software | wazuh-manager, wazuh-indexer, wazuh-dashboard 4.12.0-1 |
| Endpoint | Windows 11 Pro (the host), agent `win-endpoint-sadeeq` (ID 003), seen by the manager as `192.168.48.1` |
| Monitored folder | `C:\SensitiveFiles`, `realtime="yes"` |

```
 Windows 11 host (agent 4.12.0)  --1515/1514 TCP-->  Ubuntu VM (Manager + Indexer + Dashboard)
   FIM: C:\SensitiveFiles                              https://192.168.48.132  <-- analyst browser
```

> **Environment note.** The Wazuh server software was already installed on the ICDFA-provided lab VM. I verified the installation (package list and service status) but did not run the Wazuh installer on that VM. My own earlier attempt to build an Ubuntu Server 22.04.5 VM was abandoned, before installing Wazuh on it, after I forgot its password. See the report, Section 11.

## The six phases and what happened

| Phase | Task | Outcome |
|---|---|---|
| 1 | Wazuh deployment | Manager, Indexer, Dashboard confirmed installed (4.12.0-1) and running |
| 2 | Dashboard access | Logged in at `https://192.168.48.132` |
| 3 | Windows agent | MSI install and registration, agent 003 shows **active** |
| 4 | FIM configuration | `C:\SensitiveFiles` added to `<syscheck>` with `realtime="yes"`; confirmed in `ossec.log` |
| 5 | Event generation | File created, overwritten (by mistake), modified, deleted |
| 6 | Investigation | Four alerts analysed; four lab questions answered |

## Key results

Dashboard times are Africa/Lagos (UTC+1); the raw JSON timestamps are UTC.

| Time | Event | Rule | Level | Notes |
|---|---|---|---|---|
| 09:26:33 | added | 554 | 5 | 45-byte file, baseline hashes recorded |
| 09:27:01 | modified | 550 | 7 | `mtime` only (an accidental repeat of the create command), hashes unchanged |
| 09:27:13 | modified | 550 | 7 | size 45 to 88, MD5/SHA-1/SHA-256 changed, diff shows the appended line |
| 09:27:32 | deleted | 553 | 7 | last known hash and size recorded |

Each alert arrived about one second after the action, in `realtime` mode.

## Repository layout

```
report/      The lab report (.docx and .pdf)
evidence/    Numbered screenshots with an index (evidence/README.md)
alerts/      The four FIM alerts exactly as exported from the Wazuh index (JSON)
config/      The ossec.conf FIM entry and an illustrative ransomware rule
docs/        Lab explanation and troubleshooting notes
```

## Reproducing the lab (summary)

1. Wazuh server: install Manager, Indexer and Dashboard on Ubuntu (for a single host, the assisted installer: `curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash wazuh-install.sh -a`), or use a ready-made VM. Check with `dpkg -l | grep wazuh` and `sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard`.
2. Open `https://<server-ip>` and sign in as `admin`.
3. In the Dashboard open **Endpoints summary, Deploy new agent**, choose Windows, enter the server address and an agent name, and run the generated command in an Administrator PowerShell. Then run `NET START WazuhSvc` once the install has finished.
4. Create `C:\SensitiveFiles`, add the line from [`config/ossec_syscheck_snippet.xml`](config/ossec_syscheck_snippet.xml) inside `<syscheck>` of `C:\Program Files (x86)\ossec-agent\ossec.conf`, then `Restart-Service WazuhSvc`.
5. Wait for `File integrity monitoring scan ended` in `ossec.log`, then create, modify and delete a file in the folder.
6. In **Discover** (`wazuh-alerts-*`) search for the file name.

## Limitations

- One endpoint only, so cross-host correlation is not demonstrated.
- `whodata` was not enabled, so alerts do not show which user or process made a change.
- The ransomware rule in `config/` is an illustration. It was not deployed or tested.
- The server lost its DHCP lease twice during the session; I restored it by hand with `dhclient` and did not make the fix permanent.

## Privacy

Passwords and login credentials are not included anywhere in this repository. The registry alert I used as a false-positive example contained a Bluetooth device address, which is left out of the report.

## Licence and use

This repository documents a student lab exercise. Reuse the explanations and configuration freely; the screenshots and report are the author's own work for the ICDFA assessment.
