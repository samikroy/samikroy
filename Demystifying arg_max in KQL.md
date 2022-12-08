This session inclused 

1. Get only the latest with selective columns.

```
Heartbeat
| summarize arg_max(TimeGenerated,Computer)

```
2. Get only the latest with all columns.

```
Heartbeat
| summarize arg_max(TimeGenerated,*)

```

3. Summarize with selective columns to get the latest version.

```
Heartbeat
| where TimeGenerated > ago(1h)
| where Computer contains "dc"
| summarize arg_max(OSType, TimeGenerated) by Computer
```

4. Summarize with all columns to get the latest version.

```
Heartbeat
| where TimeGenerated > ago(1h)
| where Computer contains "dc"
| summarize arg_max(OSType, *) by Computer
```

