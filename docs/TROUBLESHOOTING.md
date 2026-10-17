# Troubleshooting notes

Problems met during the practical, with the fix that worked. Screenshots are in `evidence/` (files 20 to 24).

## VMware: "Cannot connect the virtual device sata0:1" / "Unable to find a medium containing a live file system"

The virtual CD/DVD drive was not attached to the Ubuntu ISO.

1. Power the VM off.
2. VM > Settings > CD/DVD (SATA): choose **Use ISO image file**, browse to the ISO, tick **Connect at power on**.
3. Power on again.

If it still fails, verify the ISO checksum against the one published with the release.

## Forgotten password on a self-built VM

Recovery mode through GRUB (Advanced options > recovery mode > root shell > `passwd <user>`) is the usual route. In my case the recovery boot only showed a blank screen. Since nothing was installed on the VM yet, I switched to the lab VM supplied by ICDFA. Write the password down when you create a VM.

## Dashboard unreachable ("took too long to respond") or `ens33` has no address

Symptoms: `ip a` shows `ens33` `state DOWN`, or `UP` with only an IPv6 `fe80::` address.

```bash
sudo ip link set ens33 up
sudo dhclient -v ens33
ip a            # expect inet 192.168.48.x/24
```

This is temporary. For a permanent fix configure DHCP (or a static address) for `ens33` in `/etc/netplan/*.yaml` and run `sudo netplan apply`, or give the VM a DHCP reservation. I did not apply the permanent fix in this lab.

Also check on the Windows host that the **VMware NAT Service** and **VMware DHCP Service** are running (`services.msc`).

## `NET START WazuhSvc`: "The service name is invalid"

The MSI installer was still running in the background because the install command uses `/q`. Wait until it finishes, check with `Get-Service WazuhSvc`, then run `NET START WazuhSvc` again.

## `sudo var/ossec/bin/wazuh-control info`: "command not found"

A missing leading slash. Use the absolute path `/var/ossec/bin/wazuh-control`, or read the version with `dpkg -l | grep wazuh`.

## Dashboard flyout error "File not found" when opening a registry event

A display problem in the agent's FIM flyout for registry events. Use **Discover** (`wazuh-alerts-*`) and the JSON view of the alert instead.

## No FIM alerts after configuring the folder

1. The directory must exist before the agent starts: `New-Item -ItemType Directory -Path C:\SensitiveFiles -Force`.
2. Restart the agent after editing `ossec.conf`: `Restart-Service WazuhSvc`.
3. Confirm the path was applied: `Select-String -Path "C:\Program Files (x86)\ossec-agent\ossec.log" -Pattern "sensitivefiles" -SimpleMatch`.
4. Wait for `File integrity monitoring scan ended` before testing.
5. Check the agent is **active** in Endpoints and that the time range in Discover covers the test.
