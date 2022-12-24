SigninLogs
| where TimeGenerated >ago(1d)
| where UserPrincipalName contains "samik"

union
SigninLogs,
OfficeActivity
| where * has "5d96e8c1-8ff1-4f32-a131-24084f25c205"
