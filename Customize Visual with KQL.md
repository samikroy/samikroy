Query Used

```
OfficeActivity
| summarize Total = count(), Files = countif(OfficeWorkload == "SharePoint" or OfficeWorkload == "Onedrive"), Mails = countif(OfficeWorkload == "Exchange") by bin(TimeGenerated, 24h)
| extend Files = (Files *1.00 / Total)*100,
Mails = (Mails *1.00 / Total)*100
| project-away Total
| extend TimeGenerated = strcat(format_datetime(TimeGenerated,'y-M-d'),'🕒')
| render columnchart with (title="Office Apps Details", xtitle="Time ⏲️", ymin=50,ymax=100,kind=stacked )
```
