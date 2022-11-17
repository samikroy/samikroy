Solution 1: Accessing Powetshell process with an exception to UserSid & UserNames.

Initial Solution

```
// If these numbers increse, recomend using a watchlist.
let ExludedSids = dynamic(["S-1-5-18","S-1-5-19"]);
let ExcludeUserNames = dynamic(["NT AUTHORITY\\LOCAL SERVICE","NT AUTHORITY\\LOCAL SERVICE1"]);
SecurityEvent
// Channel can be better filter.
| where Channel == ""
| where EventID in (4103, 4104,400, 403)
| where SubjectUserSid !in (ExludedSids)
or UserPrincipalName !in (ExcludeUserNames)
| where EventData contains "get and service"
or EventData contains "start and service"
or EventData contains "new and service"
or EventData contains "set and service"
| extend UserName = UserPrincipalName
```

Additional Thoughts

- Configure MMA / AMA Agents to Security Events (All Events) to the desired Log Analytics Workspace.
- Configure the analytics with the desired fequency.
- Confire Entity Mapping
- Configure Alert Name as {{UserPrincipalName}} to easier identification of alerts.
![image](https://user-images.githubusercontent.com/20562985/202419439-d9d74e4a-da51-461c-968a-ff8a17ebae1b.png)

<hr>

Solution 2: Unsuccessful Logins with an exception to UserSid & UserNames.

```
// If these numbers increse, recomend using a watchlist.
let ExludedSids = dynamic(["S-1-5-18","S-1-5-19"]);
let ExcludeUserNames = dynamic(["ANONYMOUS LOGON","SYSTEM"]);
SecurityEvent
//either uncomment or can be configured in analytics | every 10 min with 10 min of lookback data.
//| where TimeGenerated > ago(10m) 
| where EventID in (4625, 4776, 4771)
| where LogonType in (2,10)
| where UserPrincipalName !in (ExcludeUserNames) or SubjectUserSid !in (ExludedSids)
| parse EventData with * "<Data Name=\"Status\">" Status "</Data>" *
| where Status in ("<<xC0000064>>","<<0xC000006A>>")
| summarize FailedLoginCount=count() by UserPrincipalName
// either uncomment or use greater than 5 to create alerts in analytics.
//| where FailedLoginCount > 4
```
Additional Thoughts

- Configure MMA / AMA Agents to Security Events (All Events) to the desired Log Analytics Workspace.
- Configure the analytics with the desired fequency. E.g. - Run every 10 min with 10 min of lookback data.
- Confire Entity Mapping
- Configure Alert Name as {{UserPrincipalName}} to easier identification of alerts.
![image](https://user-images.githubusercontent.com/20562985/202419439-d9d74e4a-da51-461c-968a-ff8a17ebae1b.png)


<hr>
Contact Info
<hr>

<a href="https://github.com/samikroy">Github Profile</a>
<br>
<a href="https://www.linkedin.com/in/roysamik/">Linkedin Profile</a>
