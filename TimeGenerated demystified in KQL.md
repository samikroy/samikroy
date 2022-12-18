SigninLogs
//| where TimeGenerated > ago(1d)
//| where TimeGenerated > ago(3d) and TimeGenerated < ago(1d)
//| where TimeGenerated > startofday(ago(3d)) and TimeGenerated < startofday(ago(1d))
//| where TimeGenerated between (startofday(ago(7d)) .. endofday(now()) )
//| where TimeGenerated between ( datetime(2022-12-13) .. datetime(2022-12-16, 20:00) )
//| where TimeGenerated between ( startofday(ago(48h)) .. endofday(ago(1d)) )
| summarize MaxTime = max(TimeGenerated), MinTime = min(TimeGenerated)
| extend Today = startofday(now()),
MaxDate = startofday(MaxTime),
MinDate = startofday(MinTime)
| project Today, MaxDate, MinDate, MaxTime, MinTime
