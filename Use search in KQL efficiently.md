
// 1. Simple term search over all unrestricted tables and views of the database in scope
```
search "OfficeActivity"
```
// 2. Like (1), but looking only for records that match both terms
```
search "admin" and ("office" or "azure")
```
// 3. Like (1), but looking only in the TraceEvent table
```
search in (SigninLogs) "admin"
```
// 4. Like (2), but performing a case-sensitive match of all terms
```
search kind=case_sensitive "Admin" and ("SigninLogs" or "OfficeActivity")

search kind=case_sensitive "admin" and ("SigninLogs" or "OfficeActivity")
```
// 5. Searches over all the higher-ups
```
search in (Signin*, OfficeActivity) "admin" or "azure"
```
// 6. A different way to say (7). Prefer to use (7) wh
```
union Signin*, Heartbeat | search "Azure" or "davec" or "Office"
```
// 7. Like (1), but restricting the match to some columns
```
search UserPrincipalName contains "admin" or UserName contains "admin"
```
