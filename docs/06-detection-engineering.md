[Back to README](../README.md)

# 06. Detection Engineering in Splunk

Now switch to the defender. Open Splunk at `http://192.168.10.10:8000` and set the time range to cover your attack.

All queries referred to here are copy-ready in [`detections/splunk-queries.md`](../detections/splunk-queries.md).

## 1. Start broad, then narrow

```
index=endpoint tsmith
```

This shows everything that mentions the targeted account, including a visible spike during the attack. It is how you
*find* the activity; the correlation searches below are how you *detect* it automatically.

## 2. What to look for (brute force)

| Event | Meaning | In this lab |
| :-: | :--- | :--- |
| 4625 | Failed logon | A burst of failures for `tsmith`, roughly one per wrong password (retries and network-level authentication can add a few extra) |
| 4624 | Successful logon | One success immediately after the failures |

On the 4624 event read **Source Network Address**. It should be `192.168.10.250`, the Kali machine, which attributes the
attack to its source.

## 3. Turn the hunt into a detection

A person scrolling events is not a control. The correlation query (query 2 in the file) counts failures per source and
only keeps sources that also succeeded:

```
failures >= 5 AND successes >= 1   →   password guessing that worked
```

To make it permanent, run it and choose **Save As > Alert**, schedule it every 5 minutes, and trigger when the number of
results is greater than zero.

## 4. Detect the Atomic Red Team activity

| Technique | Where it shows up | Query |
| :--- | :--- | :--- |
| T1136.001 local account created | Security **4720** and a Sysmon process creation containing `net user ... /add` | 3 and 4 |
| T1059.001 PowerShell | Sysmon process creation for `powershell.exe` with encoded, bypass or download arguments | 5 |

If a technique ran but the matching events are not there, you have found a **visibility gap**. Fix the logging
(Sysmon config, PowerShell logging, the `inputs.conf` stanza) and run the test again. Proving a gap and then closing
it is the most valuable thing this lab teaches.

## 5. MITRE ATT&CK mapping

| Activity | Technique |
| :--- | :--- |
| Dictionary attack against RDP | T1110.001 Password Guessing |
| Logging in over RDP | T1021.001 Remote Desktop Protocol |
| Creating a local account | T1136.001 Create Account: Local Account |
| PowerShell execution | T1059.001 Command and Scripting Interpreter: PowerShell |

## 6. Field names depend on your setup

Splunk extracts Windows fields differently depending on whether the Splunk Add-on for Microsoft Windows is installed.
The queries here use field names the default Windows event format provides and parse Sysmon XML with `rex` so they work
without extra add-ons. Before trusting a query, check what fields you actually have:

```
index=endpoint source="WinEventLog:Security" EventCode=4625 | head 5 | fieldsummary
```

## 7. Ideas to extend this

- Add a dashboard with failed-logon counts over time and top source addresses.
- Detect the same brute force on the domain controller (Kerberos 4768/4771, NTLM 4776).
- Add a Kerberoasting or pass-the-hash simulation and write the matching detection.
