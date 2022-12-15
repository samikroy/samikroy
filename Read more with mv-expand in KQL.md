SigninLogs
| take 1
| mv-expand ConditionalAccessPolicies
| extend displayName_=  ConditionalAccessPolicies.displayName
| extend conditionsSatisfied_ = tostring(ConditionalAccessPolicies.conditionsSatisfied)
| extend enforcedSessionControls_ = ConditionalAccessPolicies.enforcedSessionControls
| project displayName_, conditionsSatisfied_, enforcedSessionControls_
| mv-expand enforcedSessionControls_
