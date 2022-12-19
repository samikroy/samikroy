```
let domainName = "M365x29954847";
SigninLogs
| summarize make_set_if(AppDisplayName, UserPrincipalName contains domainName and isnotempty(AppDisplayName),3)
```
```
let domainName = "M365x29954847";
SigninLogs
| summarize make_set_if(AppDisplayName, UserPrincipalName !contains domainName and isnotempty(AppDisplayName),3)
```
