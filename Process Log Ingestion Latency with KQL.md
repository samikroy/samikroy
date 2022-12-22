let List_Of_Tables = dynamic(['Usage','BehaviorAnalytics']);
union withsource = DummyTableName *
| summarize 
      ['average E2E IngestionLatency'] = round(avg(todouble(datetime_diff("Second",ingestion_time(),TimeGenerated)) ),2)
    , ['minimun E2E IngestionLatency'] = round(min(todouble(datetime_diff("Second",ingestion_time(),TimeGenerated)) ),2) 
    , ['maximum E2E IngestionLatency'] = round(max(todouble(datetime_diff("Second",ingestion_time(),TimeGenerated)) ),2)
  by TableName = DummyTableName
| sort by ['average E2E IngestionLatency'] desc
| where TableName in (List_Of_Tables)
