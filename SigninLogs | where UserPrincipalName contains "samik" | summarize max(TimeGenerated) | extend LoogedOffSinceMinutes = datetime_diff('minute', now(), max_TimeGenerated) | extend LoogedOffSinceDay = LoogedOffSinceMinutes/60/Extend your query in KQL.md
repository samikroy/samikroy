SigninLogs
| where UserPrincipalName contains "samik"
| summarize max(TimeGenerated)
| extend LoogedOffSinceMinutes = datetime_diff('minute', now(), max_TimeGenerated)
| extend LoogedOffSinceDay = LoogedOffSinceMinutes/60/24




SigninLogs
| where UserPrincipalName contains "samik"
| summarize max(TimeGenerated)
| extend LoogedOffSinceMinutes = datetime_diff('minute', now(), max_TimeGenerated),
LoogedOffSinceDay = datetime_diff('day', now(), max_TimeGenerated)



SigninLogs
| where UserPrincipalName contains "samik"
| summarize max(TimeGenerated)
| extend LoogedOffSince =  datetime_diff('second', now(), max_TimeGenerated)
| extend LoogedOffSinceMinutes = datetime_diff('minute', now(), max_TimeGenerated),
LoogedOffSinceDay = datetime_diff('day', now(), max_TimeGenerated)
