SigninLogs
| summarize arg_max(TimeGenerated, ResultType), FailedLogin = countif(ResultType != 0) by UserPrincipalName, bin(TimeGenerated,1h)
| where ResultType  == 0
| render columnchart with (kind=stacked , ymin=4)




SigninLogs
| summarize arg_max(TimeGenerated, ResultType), FailedLogin = countif(ResultType != 0) by UserPrincipalName



SigninLogs
| summarize arg_max(TimeGenerated, ResultType), FailedLogin = countif(ResultType != 0) by UserPrincipalName, bin(TimeGenerated,1h)


SigninLogs
| summarize arg_max(TimeGenerated, ResultType), FailedLogin = countif(ResultType != 0) by UserPrincipalName, bin(TimeGenerated,1h)
| where ResultType  == 0

SigninLogs
| summarize arg_max(TimeGenerated, ResultType), FailedLogin = countif(ResultType != 0) by UserPrincipalName, bin(TimeGenerated,1h)
| where ResultType  == 0
| render columnchart with (kind=stacked , ymin=4)
