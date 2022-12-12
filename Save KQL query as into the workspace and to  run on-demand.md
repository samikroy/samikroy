Demo query

```
SigninLogs
| summarize Total = count(), FailedSignIns = countif(ResultType !=0),
SuccessfulSignIns = countif(ResultType ==0) by UserPrincipalName
| extend ["Failed %"] = (FailedSignIns/Total) * 100
| extend ["Success %"] = (FailedSignIns/Total) * 100
```


Actual Query 

```
SigninLogs
| summarize Total = count(), FailedSignIns = countif(ResultType !=0),
SuccessfulSignIns = countif(ResultType ==0) by UserPrincipalName
| extend ["Failed %"] = (FailedSignIns * 1.00/Total) * 100
| extend ["Success %"] = (FailedSignIns * 1.00/Total) * 100
```
