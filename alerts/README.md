# FIM alerts (raw)

These four files are the alert records for `c:\sensitivefiles\confidential_report.txt`, exported unmodified from the `wazuh-alerts-4.x-2026.10.16` index (Discover, JSON view).

| File | Event | Rule | Level | Raw timestamp (UTC) |
|---|---|---|---|---|
| `alert_1_added_rule554.json` | added | 554 | 5 | 2026-10-16T08:26:33.247 |
| `alert_2_modified_mtime_only_rule550.json` | modified | 550 | 7 | 2026-10-16T08:27:01.240 |
| `alert_3_modified_content_rule550.json` | modified | 550 | 7 | 2026-10-16T08:27:13.281 |
| `alert_4_deleted_rule553.json` | deleted | 553 | 7 | 2026-10-16T08:27:32.229 |

Dashboard times are UTC+1 (Africa/Lagos), so 08:26:33 UTC is 09:26:33 in the screenshots.

## Fields worth reading first

| Field | Meaning |
|---|---|
| `syscheck.event` | `added`, `modified` or `deleted` |
| `syscheck.mode` | `realtime` (watcher) or `scheduled` (periodic scan) |
| `syscheck.changed_attributes` | what changed, e.g. `["mtime"]` or `["size","mtime","md5","sha1","sha256"]` |
| `syscheck.md5_before` / `md5_after` (also sha1, sha256) | integrity hashes before and after |
| `syscheck.size_before` / `size_after` | file size in bytes |
| `syscheck.diff` | what text was added or removed (needs `report_changes="yes"`) |
| `syscheck.uname_after` | owner of the file, not necessarily who changed it (that needs `whodata`) |
| `rule.id`, `rule.level`, `rule.description` | which rule fired and how severe it is |
| `rule.mitre.*` | mapped ATT&CK technique and tactic |

Quick look with `jq`:

```bash
jq '._source | {time: .timestamp, event: .syscheck.event, rule: .rule.id, level: .rule.level, changed: .syscheck.changed_attributes}' alerts/*.json
```
