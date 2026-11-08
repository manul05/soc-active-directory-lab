# Evidence

Screenshots that prove the lab works. Add the files below using exactly these names, then uncomment the **Results**
section in the main [README](../README.md).

| File | What it should show |
| :--- | :--- |
| `01-network-and-vms.png` | VirtualBox with all four VMs on the `AD-Project` NAT network |
| `02-splunk-receiving-9997.png` | Splunk receiving port 9997 enabled |
| `03-telemetry-health.png` | Query 6: hosts and sources reporting into `endpoint` |
| `04-domain-joined.png` | Target-PC reporting domain `aman.test` |
| `05-failed-logons-4625.png` | The burst of 4625 events during the attack |
| `06-success-4624-source-ip.png` | The 4624 event with Source Network Address `192.168.10.250` |
| `07-bruteforce-detection-search.png` | Query 2 returning the Kali source |
| `08-account-creation-detected.png` | Query 3 or 4 showing the Atomic Red Team account creation |

**Before adding any image:** crop or blur anything real (your hostname, real email, real IPs, license keys). Lab
addresses in `192.168.10.0/24` are safe to show.
