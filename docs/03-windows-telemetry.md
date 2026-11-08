[Back to README](../README.md)

# 03. Windows Telemetry: Sysmon and the Splunk Universal Forwarder

Do this on **Target-PC** first. Repeat it on **AD-DC-01** after the server is built (it uses the same steps; only the
hostname and IP differ).

## Step 1: Hostname and static IP (Target-PC)

1. Search **Rename this PC** and set the name to `Target-PC`.
2. **Network Status > Change adapter options > Properties > Internet Protocol Version 4**, set:

| Setting | Value |
| :--- | :--- |
| IP address | `192.168.10.100` |
| Subnet mask | `255.255.255.0` |
| Gateway | `192.168.10.1` |
| DNS | `8.8.8.8` (you will change this to the domain controller in [04](04-active-directory.md)) |

For AD-DC-01 use `192.168.10.7` (see [04](04-active-directory.md) for its DNS settings).

## Step 2: Install Sysmon

By default Windows does not log the detail an investigator needs, such as full command lines and parent processes.
Sysmon adds it.

1. Download Sysmon from the
   [Sysinternals page](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) and extract it.
2. Download `sysmonconfig.xml` from the community
   [sysmon-modular](https://github.com/olafhartong/sysmon-modular) project into the same folder.
3. In an **Administrator PowerShell**:
   ```powershell
   .\Sysmon64.exe -accepteula -i sysmonconfig.xml
   ```

## Step 3: Install the Splunk Universal Forwarder

Run the UF `.msi` installer. When it asks for the **Receiving Indexer**, enter `192.168.10.10` and port `9997`.

## Step 4: Tell the forwarder what to send

1. Open Notepad **as Administrator** and create or edit:
   `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`
2. Paste:
   ```ini
   [WinEventLog://Application]
   index = endpoint
   disabled = false

   [WinEventLog://Security]
   index = endpoint
   disabled = false

   [WinEventLog://System]
   index = endpoint
   disabled = false

   [WinEventLog://Microsoft-Windows-Sysmon/Operational]
   index = endpoint
   disabled = false
   renderXml = true
   source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
   ```
3. Restart the **SplunkForwarder** service (`services.msc`, or `Restart-Service SplunkForwarder` in an elevated
   PowerShell). You must restart it after **every** change to `inputs.conf`.

## Step 5: Verify

In Splunk run:

```
index=endpoint | stats count BY host, source
```

You should see rows for `Target-PC` with the Security, System, Application and Sysmon sources. If a source is
missing, that is a visibility gap: check the stanza spelling, the service restart, and that port 9997 is reachable.

> [!NOTE]
> The domain controller's Security log is far busier than a workstation's (every authentication in the domain passes
> through it). Expect much higher volume from AD-DC-01.
