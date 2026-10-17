# Lab explanation

This note explains the ideas behind the lab and how to read what it produced. The step-by-step record with screenshots is in the [report](../report/).

## 1. The problem

A CISO wants to know when sensitive files on employee workstations are changed without authorisation. Windows can log some of this locally, but a person with administrator rights can clear those logs, and nobody is looking at every machine. The answer is to send file-change events from each machine to a central system that analysts watch: a **SIEM** (Security Information and Event Management).

## 2. Wazuh in one picture

| Component | Job | In this lab |
|---|---|---|
| **Agent** | Runs on the monitored machine, watches files and logs, sends events to the manager | Windows 11 host, agent `win-endpoint-sadeeq` |
| **Manager** | Receives events, decodes them, applies rules, raises alerts | `wazuh-manager` on the Ubuntu VM |
| **Indexer** | Stores and searches alerts (OpenSearch based) | `wazuh-indexer` |
| **Dashboard** | Web interface for analysts | `wazuh-dashboard` at `https://192.168.48.132` |

The agent talks to the manager over TCP 1514 (events, encrypted) and registers over TCP 1515. The analyst only ever uses the Dashboard (TCP 443).

## 3. File Integrity Monitoring (FIM)

FIM is Wazuh's `syscheck` module. It works like this:

1. **Baseline.** At start-up the agent scans the configured paths and records, for every file, its size, permissions, owner, modification time and cryptographic hashes (MD5, SHA-1, SHA-256).
2. **Detection.** Later, if any of those values differ from the baseline, an event is sent. A new file is `added`, a changed one is `modified`, a removed one is `deleted`.
3. **Scheduled or real-time.** By default syscheck rescans on a timer (here every 43200 s, 12 hours). With `realtime="yes"` the agent also uses the operating system's change notifications, so events arrive within about a second. That is why the test alerts appeared almost immediately.
4. **Diff.** With `report_changes="yes"` the agent keeps the previous text content, so the alert can show exactly which lines were added or removed.

The line added in this lab:

```xml
<directories realtime="yes" check_all="yes" report_changes="yes">C:\SensitiveFiles</directories>
```

## 4. Reading an alert

An alert is a JSON document. For `confidential_report.txt` the useful parts are:

- **What happened:** `syscheck.event` (`added`, `modified`, `deleted`) and `syscheck.path`.
- **How we know:** `syscheck.mode` is `realtime`, which tells you the watcher saw it, not a later scan.
- **What exactly changed:** `syscheck.changed_attributes`, the before/after hashes and sizes, and `syscheck.diff`.
- **How serious:** `rule.id` and `rule.level`. Rule 554 (added) is level 5; rules 550 (modified) and 553 (deleted) are level 7.
- **Context:** `rule.mitre` maps the event to ATT&CK (550 to T1565.001 Stored Data Manipulation; 553 to T1070.004 File Deletion and T1485 Data Destruction) and the compliance fields (PCI DSS 11.5, HIPAA, NIST 800-53 SI.7, GDPR) tell an auditor why FIM matters.

### What the four test alerts show

| Alert | What to notice |
|---|---|
| 1. added (554) | The baseline for the new file: 45 bytes plus its three hashes. |
| 2. modified (550) | Only `mtime` changed and the diff says "No content changes were found". This came from an accidental repeat of the create command. Same content, new modification time. A good example of an alert that looks alarming but is low risk. |
| 3. modified (550) | Size 45 to 88 bytes, all three hashes changed, diff shows `Unauthorized modification during SOC test`. This is the real modification. |
| 4. deleted (553) | The last known size and hashes, so you can prove which version was removed. |

One limit: `uname_after` shows the file's owner, not who changed it. To record the responsible user and process you would enable `whodata` on the directory.

## 5. The six phases, and why each exists

| Phase | Why it matters |
|---|---|
| 1. Deploy Wazuh | You cannot analyse events without a manager, an index and a UI. |
| 2. Dashboard access | Proves the whole stack, not just the manager, is working. |
| 3. Agent registration | The endpoint is invisible until its agent is enrolled and shows **active**. |
| 4. FIM configuration | Without a monitored path there is nothing to detect. The log line `Monitoring path ... realtime` is the proof that it applied. |
| 5. Generate events | A controlled create, modify, delete gives known "ground truth" to compare with the alerts. |
| 6. Investigation | Turning alerts into decisions: triage, false positives, ransomware patterns, and why central monitoring beats local logs. |

## 6. The four analysis questions in one line each

1. **Triage:** read `changed_attributes`, hashes and diff to tell a content change from a metadata-only change, check timing and neighbouring alerts, then escalate or close.
2. **False positives:** busy locations (Temp folders, caches, Windows Update folders, volatile registry keys) flood FIM; narrow paths and use `<ignore>`, `<registry_ignore>` and `restrict`. This lab saw registry noise (rules 751 and 597) first-hand.
3. **Ransomware:** look for bursts of rule 550 across many files, new files with odd extensions, and deletions, all in real time, and correlate with a frequency rule (illustrated in [`config/`](../config/local_rules_ransomware_example.xml)).
4. **Wazuh vs Windows Event Logs:** off-host storage that an attacker cannot easily clear, hashes and diffs in one alert, severity and framework tags, one console for many machines.

## 7. Timeline of the practical (16 October 2026, Africa/Lagos)

| Time | Event |
|---|---|
| 09:12:20 | Agent 003 registered |
| 09:23:13 | Agent restarted; FIM path applied |
| 09:25:27 | FIM baseline scan ended |
| 09:25:33 | Real-time monitoring started |
| 09:26:32 | Test file created |
| 09:27:00 | Create command repeated by mistake |
| 09:27:12 | Line appended |
| 09:27:31 | File deleted |
