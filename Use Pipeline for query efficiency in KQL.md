Query 1

```
Usage
| where TimeGenerated >ago(24h) and DataType == "SigninLogs"
```

Query 2
```
Usage
| where TimeGenerated >ago(24h) 
| where DataType == "SigninLogs"
```
