

Heartbeat
| where TimeGenerated > ago(1h)
| where Computer contains "dc"
| summarize arg_max(OSType, TimeGenerated) by Computer
