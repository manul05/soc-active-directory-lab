[Back to README](../README.md)

# 01. Virtualization Setup

The whole lab runs in VirtualBox on one host.

## Host requirements

| Resource | Minimum |
| :--- | :--- |
| RAM | 16 GB (the four VMs together need about 14 GB if Splunk gets 8 GB, so close other apps) |
| Free disk | 250 GB |
| Software | VirtualBox 7.x |

## Step 1: Create the NAT Network

1. Open VirtualBox and go to **File > Tools > Network Manager** (on older versions: **File > Preferences > Network**).
2. Create a **NAT Network** named `AD-Project`.
3. Set the **IPv4 Prefix** to `192.168.10.0/24`.

## Step 2: Create the virtual machines

Download the ISOs from the vendors' official sites first (Ubuntu Server 22.04, Windows Server 2022, Windows 10). For
Kali, download the **pre-built VirtualBox image** from kali.org instead of an ISO.

| VM | RAM | Disk | Base OS |
| :--- | :-: | :-: | :--- |
| Splunk-Server | 4 to 8 GB (8 GB recommended) | 100 GB | Ubuntu Server 22.04 LTS |
| AD-DC-01 | 4 GB | 50 GB | Windows Server 2022 |
| Target-PC | 4 GB | 50 GB | Windows 10 |
| Attacker-Kali | default | default | Kali Linux pre-built VM |

## Step 3: Attach every VM to the lab network

For **each** of the four VMs: **Settings > Network > Adapter 1**, set **Attached to** to **NAT Network**, and choose
`AD-Project`.

> [!TIP]
> Take a snapshot of each VM right after the OS install (`Machine > Take Snapshot`). Atomic Red Team and the attack
> steps change the machines, and snapshots let you reset to a clean state in seconds.
