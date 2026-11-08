# Splunk detection queries

Starting-point searches for the lab. Run them against the `endpoint` index and set the time range to cover your attack.

> [!IMPORTANT]
> These are written from the event formats the lab produces but must be **validated against your own data**. Field
> names can differ with your Splunk version and add-ons. Confirm each query returns your test activity and save a
> screenshot into [`../evidence/`](../evidence/README.md). A detection you have not seen fire is only a hypothesis.

## 1. Failed logons by source (T1110)

Sources with five or more failed logons.

```spl
index=endpoint source="WinEventLog:Security" EventCode=4625
| stats count AS failures, dc(Account_Name) AS distinct_accounts, earliest(_time) AS first_seen, latest(_time) AS last_seen BY Source_Network_Address
| where failures >= 5
| convert ctime(first_seen) ctime(last_seen)
| sort - failures
```

## 2. Password guessing that succeeded (T1110.001 followed by valid logon)

A source that failed repeatedly and then logged on successfully. This is the high-signal version of query 1.

```spl
index=endpoint source="WinEventLog:Security" (EventCode=4625 OR EventCode=4624) Source_Network_Address=*
| where Source_Network_Address!="-" AND Source_Network_Address!="::1" AND Source_Network_Address!="127.0.0.1"
| stats count(eval(EventCode=4625)) AS failures, count(eval(EventCode=4624)) AS successes, earliest(_time) AS first_seen, latest(_time) AS last_seen BY Source_Network_Address
| where failures >= 5 AND successes >= 1
| convert ctime(first_seen) ctime(last_seen)
```

Expected in the lab: a single row for `192.168.10.250`.

## 3. New local account created (T1136.001)

```spl
index=endpoint source="WinEventLog:Security" EventCode=4720
| table _time host Account_Name
```

`Account_Name` appears twice in this event (who created it, and the new account), so check both values.

## 4. `net user /add` seen in process creation (T1136.001)

Parses the Sysmon XML directly with `rex`, so it works without a Windows add-on.

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" "<EventID>1</EventID>"
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]*)</Data>"
| rex field=_raw "<Data Name='CommandLine'>(?<CommandLine>[^<]*)</Data>"
| search CommandLine="*user*/add*"
| table _time host Image CommandLine
```

## 5. Suspicious PowerShell launches (T1059.001)

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" "<EventID>1</EventID>"
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]*)</Data>"
| rex field=_raw "<Data Name='CommandLine'>(?<CommandLine>[^<]*)</Data>"
| search (Image="*\\powershell.exe" OR Image="*\\pwsh.exe")
        (CommandLine="* -enc*" OR CommandLine="*-ExecutionPolicy Bypass*" OR CommandLine="*DownloadString*" OR CommandLine="*IEX*")
| table _time host Image CommandLine
```

Expect some benign hits from admin scripts in a real environment; in the lab, tune the condition to the exact test you ran.

## 6. Telemetry health check (find visibility gaps)

Which hosts and log sources are actually reporting?

```spl
index=endpoint
| stats count latest(_time) AS last_event BY host, source
| convert ctime(last_event)
| sort host source
```

A missing row, or an old `last_event`, means that source has stopped reporting and every detection built on it is blind.
