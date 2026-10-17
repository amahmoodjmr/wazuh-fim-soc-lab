# Evidence index

Screenshots from the practical on 16 October 2026. The figure number is the one used in the report.

| File | What it shows | Report figure |
|---|---|---|
| `01_wazuh_packages_dpkg_4.12.0.png` | Installed Wazuh packages on the Ubuntu server (dpkg): dashboard, indexer and manager, all 4.12.0-1 | 3 |
| `02_indexer_dashboard_services_running.png` | wazuh-indexer journal ("Started wazuh-indexer") and wazuh-dashboard active (running) | 4 |
| `03_manager_active_and_dhcp_recovery.png` | wazuh-manager active (running) since 2026-10-16 08:12:13 UTC, with the address 192.168.48.132 held on ens33 | 5 |
| `04_server_ip_192.168.48.132.png` | ens33 UP with the DHCP address 192.168.48.132/24 | 6 |
| `05_icdfa_server_os_ubuntu_20.04.6.png` | Server login banner: hostname icdfa-server, Ubuntu 20.04.6 LTS, kernel 5.4.0-216-generic | 2 |
| `06_dashboard_login_success_overview.png` | Successful login: Wazuh Dashboard overview at https://192.168.48.132 (Agents summary and Last 24 hours alerts) | 7 |
| `07_agent_install_command_msi_4.12.0.png` | Agent install command, followed by NET START WazuhSvc run too early ("The service name is invalid") | 8 |
| `08_wazuh_service_started.png` | NET START WazuhSvc: "The Wazuh service was started successfully" | 9 |
| `09_endpoints_list_agent_003_active.png` | Endpoints list: win-endpoint-sadeeq (ID 003) is active, version 4.12.0, Windows 11 Pro | 10 |
| `10_agent_003_detail_active.png` | Agent 003 detail page: active, 192.168.48.1, Wazuh v4.12.0, cluster node01, group default | 11 |
| `11_ossec_conf_syscheck_realtime_block.png` | ossec.conf: new real-time directory entry inside <syscheck>, directly below <frequency> | 14 |
| `12_sensitivefiles_folder_created.png` | C:\SensitiveFiles created (PowerShell New-Item) | 13 |
| `13_ossec_log_monitoring_path_realtime.png` | ossec.log: c:\sensitivefiles is monitored with report_changes and realtime | 15 |
| `14_ossec_log_agent_connected.png` | ossec.log: agent connected to 192.168.48.132:1514/tcp and is now online (09:23:13 to 09:23:18) | 12 |
| `15_fim_scan_ended_realtime_started.png` | ossec.log: FIM scan ended (09:25:27) and real-time FIM started (09:25:33) | 16 |
| `16a_test_create.png` | Step 1: file created at 09:26:32 | 17 |
| `16b_test_overwrite_modify_delete.png` | Steps 2 to 4: overwrite 09:27:00, append 09:27:12, delete 09:27:31 | 18 |
| `17_fim_recent_events_four_alerts.png` | FIM: Recent events: added (554), modified (550) twice, deleted (553) for c:\sensitivefiles\confidential_report.txt | 19 |
| `18_discover_four_hits_realtime.png` | Discover: 4 hits for the test file, syscheck.mode realtime; the modified event shows size 45 to 88 and the diff line | 20 |
| `19_registry_noise_and_flyout_error.png` | Default registry monitoring produced rule 751 and 597 alerts; opening a registry event raised a dashboard "File not found" flyout error | 21 |
| `20_challenge_cdrom_device_error.png` | Challenge 1: virtual CD/DVD device error in VMware Workstation | 22 |
| `21_challenge_live_filesystem_not_found.png` | Challenge 1: installer could not find the live file system | 23 |
| `22_challenge_dashboard_unreachable.png` | Challenge 4: Dashboard unreachable (ERR_CONNECTION_TIMED_OUT) | 24 |
| `23_challenge_ens33_no_ipv4.png` | Challenge 4: ens33 UP with only an IPv6 link-local address | 25 |
| `24_challenge_service_name_invalid.png` | Challenge 5: NET START WazuhSvc before the agent finished installing | 26 |

## Required evidence from the brief

| Requirement | Files |
|---|---|
| Successful installation of Wazuh components on the Ubuntu server | `01`, `02`, `03` |
| Successful login to the Wazuh Dashboard | `06` |
| Windows agent registered and Active | `09`, `10` |
| SensitiveFiles configured for real-time FIM in ossec.conf | `11`, `13` (and `../config/`) |
| Test events and resulting alerts | `16a`, `16b`, `17`, `18` (and `../alerts/`) |

## Notes

- Agents `001` (Windows-SOC-Lab) and `002` (Windows2) visible in `09` were already registered on the ICDFA server. My agent is `003`, `win-endpoint-sadeeq`.
- Screenshots that showed login credentials were deliberately left out.
- `19` shows the default registry-monitoring noise (rules 751 and 597) and a Dashboard display error.
