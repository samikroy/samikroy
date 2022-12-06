

Query 1 : Take 1 from SignIn Logs to verify the data set existence.

```
SigninLogs
| take 1
```

Query 2 : Login failure & success summary by Time Generated

```
SigninLogs
| summarize count(), SuccessSignins = countif(ResultType == 0),FailedSignins = countif(ResultType != 0) by bin(TimeGenerated, 10m)
```
Query 3 : Apply the right visual.

```
SigninLogs
| summarize count(), SuccessSignins = countif(ResultType == 0),
FailedSignins = countif(ResultType != 0),
make_set(UserPrincipalName) by bin(TimeGenerated, 10m)
| where FailedSignins >1 and array_length( set_UserPrincipalName) > 10
| extend Users = tostring(set_UserPrincipalName)
| render columnchart 
```
