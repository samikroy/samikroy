```
SigninLogs
| summarize ["Login Count for each User"] = count() by ["Domain User Name"] = UserPrincipalName
```

```
SigninLogs
| summarize LoginCount = count() by DomainUserName = UserPrincipalName
```
