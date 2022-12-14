let lookback = 1d;
//print lookback
let SignIns = SigninLogs
| where TimeGenerated >ago(lookback);
let SignInUsers = SigninLogs
| where TimeGenerated >ago(lookback)
| distinct UserPrincipalName;
SignIns
| summarize countif(UserPrincipalName in (SignInUsers))
