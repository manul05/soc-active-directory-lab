[Back to README](../README.md)

# 04. Active Directory

A domain controller (DC) stores every user, computer and password hash in the domain and enforces policy on all of
them. Here it turns a set of unrelated VMs into one managed network.

## Part A: Build the domain controller (AD-DC-01)

### Step 1: Static IP

Set the server's IPv4 properties:

| Setting | Value |
| :--- | :--- |
| IP address | `192.168.10.7` |
| Subnet mask | `255.255.255.0` |
| Gateway | `192.168.10.1` |
| Preferred DNS | `127.0.0.1` (the DC resolves its own domain) |
| Alternate DNS | `8.8.8.8` |

### Step 2: Install Active Directory Domain Services

1. **Server Manager > Add Roles and Features**, keep clicking **Next** to **Server Roles**.
2. Tick **Active Directory Domain Services** and install.

### Step 3: Promote the server

1. In Server Manager click the **flag icon** and choose **Promote this server to a domain controller**.
2. Choose **Add a new forest** and enter the root domain `aman.test`.
3. Set a Directory Services Restore Mode (DSRM) password, accept the defaults, and let the server reboot.

### Step 4: Create OUs and users

1. **Tools > Active Directory Users and Computers**.
2. Right-click the domain > **New > Organizational Unit** and create `IT` and `HR` (an OU is just a folder with
   policy attached).
3. Create users in them, for example Jenny Smith (`jsmith`) and Terry Smith (`tsmith`).

> [!WARNING]
> Use throwaway lab passwords that you do not use anywhere else. One of them is deliberately placed in a password
> list during the attack, so treat it as burned.

Now install Sysmon and the forwarder on this machine too ([03](03-windows-telemetry.md)).

## Part B: Join Target-PC to the domain

### Step 1: Point DNS at the DC

Target-PC must resolve `aman.test` through the DC, not Google.

1. **Network Connections > Change adapter options > Properties > IPv4**.
2. Keep the static IP, set **Preferred DNS** to `192.168.10.7`.

### Step 2: Join

1. Search **About your PC > Rename this PC (advanced) > Change**.
2. Under **Member of**, choose **Domain** and enter `aman.test`.
3. At the credentials prompt use a **domain** account: `AMAN\Administrator` and the password you set on the server
   (a local Target-PC account will not work).
4. Reboot when prompted.

### Step 3: Verify

On Target-PC, in PowerShell:

```powershell
(Get-WmiObject Win32_ComputerSystem).Domain     # expect aman.test
nltest /dsgetdc:aman.test                       # expect AD-DC-01
```

## Part C: Allow RDP to Target-PC (attack target)

1. On Target-PC open **Remote Desktop settings** and switch **Enable Remote Desktop** on.
2. **Select users that can remotely access this PC**, add `jsmith` and `tsmith`.
3. From another lab VM, or from the DC, confirm one of them can actually sign in over RDP with the correct password.
   The attack in [05](05-attack-simulation.md) is only meaningful if the legitimate login works first.
