Query 1
```
let lookuptime = 30d;
let RareFilesCreated =
DeviceFileEvents
| where ActionType == 'FileCreated'
| where Timestamp >ago(lookuptime)
| where InitiatingProcessFolderPath == @"c:\windows\system32\ntoskrnl.exe"
| summarize count() by SHA1
| where count_ < 3
| distinct SHA1;
DeviceEvents
| where Timestamp >ago(lookuptime)
| where InitiatingProcessFolderPath == @"c:\windows\system32\ntoskrnl.exe"
| where ActionType == @"NamedPipeEvent"
| project DeviceName, NamedPipeTimeStamp = Timestamp, NamedPipeProcess = InitiatingProcessFileName, NamedPipeProcessId = InitiatingProcessId, NamedPipeProcessStartTime = InitiatingProcessCreationTime, NamedPipeProcessSHA1 = InitiatingProcessSHA1, FileOperation=extractjson("$.FileOperation", AdditionalFields, typeof(string)), NamedPipeEnd=extractjson("$.NamedPipeEnd", AdditionalFields, typeof(string)), PipeName=extractjson("$.PipeName", AdditionalFields, typeof(string))
| join kind=leftouter (
DeviceFileEvents
| where Timestamp >ago(lookuptime)
| where InitiatingProcessFolderPath == @"c:\windows\system32\ntoskrnl.exe"
| where ActionType == 'FileCreated' 
| where SHA1 in~ (RareFilesCreated)
| project DeviceName, FileCreationTimestamp = Timestamp, NamedPipeProcess = InitiatingProcessFileName, NamedPipeProcessId = InitiatingProcessId, NamedPipeProcessStartTime = InitiatingProcessCreationTime, NamedPipeProcessSHA1 = InitiatingProcessSHA1, FileCreated = FileName, FileCreatedSHA1 = SHA1, FileCreatedFolder = FolderPath
) on NamedPipeProcessId, NamedPipeProcessSHA1, NamedPipeProcessStartTime
| project-away NamedPipeProcessId1, NamedPipeProcessSHA11, NamedPipeProcessStartTime1
| join kind=leftouter (
DeviceProcessEvents
| where Timestamp >ago(lookuptime)
| where InitiatingProcessFileName =~ "services.exe"
| where SHA1 in~ (RareFilesCreated)
| project DeviceName, FileCreated = FileName, FileCreatedSHA1 = SHA1, FileCreatedFolder = FolderPath, StartedProcessCommandLine = ProcessCommandLine, StartedProcessName = FileName, StartedProcessSHA1 = SHA1, StartedProcessParent = InitiatingProcessFileName, StartedProcessTimestamp = Timestamp
) on FileCreated, FileCreatedSHA1, FileCreatedFolder
| where StartedProcessTimestamp between (NamedPipeTimeStamp .. (NamedPipeTimeStamp+1m))
| project-away  FileCreated1, FileCreatedSHA11, NamedPipeProcess1, DeviceName1, DeviceName2, FileCreatedSHA11
| summarize NamedPipes = make_set(PipeName), StartedProcessTimestamps = make_set(StartedProcessTimestamp), NamedPipeTimeStamps = make_set(NamedPipeTimeStamp) by DeviceName, NamedPipeProcess, NamedPipeProcessId, NamedPipeProcessSHA1, FileCreated, FileCreatedSHA1, FileCreatedFolder, StartedProcessCommandLine, StartedProcessName, StartedProcessSHA1, StartedProcessParent
```

Query 2

```
# WMIexec
let LookupTime = 30d;
let GetRareWMIProcessLaunches = materialize (
DeviceEvents
| where Timestamp > ago(LookupTime)
| where ActionType == @"ProcessCreatedUsingWmiQuery"
| where isnotempty(FileName)
| summarize count() by SHA1, InitiatingProcessCommandLine
| where count_ > 5 | distinct SHA1); 
DeviceEvents 
| where Timestamp > ago(LookupTime)
| where ActionType == @"ProcessCreatedUsingWmiQuery"
| where SHA1 in~ (GetRareWMIProcessLaunches)
| where isnotempty(FileName)
| project DeviceName, WMIProcessLaunchTimestmap = Timestamp, ProcessLaunchedByWMI = tolower(FileName), ProcessLaunchedByWMICommandLine = tolower(ProcessCommandLine), ProcessLaunchedByWMICreationTime = ProcessCreationTime, ProcessLaunchedByWMISHA1 = tolower(SHA1), ProcessLaunchedByWMIID = ProcessId, InitiatingProcessFileName, InitiatingProcessCommandLine, InitiatingProcessParentCreationTime, InitiatingProcessParentFileName
| join kind=leftouter (
DeviceProcessEvents
| where Timestamp > ago(LookupTime)
| where InitiatingProcessSHA1 in~ (GetRareWMIProcessLaunches)
|project DeviceName, ChildProcessTimestamp = Timestamp, ProcessLaunchedByWMI = tolower(InitiatingProcessFileName), ProcessLaunchedByWMICommandLine = tolower(InitiatingProcessCommandLine), ProcessLaunchedByWMICreationTime = InitiatingProcessCreationTime, ProcessLaunchedByWMISHA1 = tolower(InitiatingProcessSHA1), ProcessLaunchedByWMIID = InitiatingProcessId, WMIchild = FileName, WMIChildCommandline = ProcessCommandLine
) on DeviceName, ProcessLaunchedByWMI, ProcessLaunchedByWMICommandLine, ProcessLaunchedByWMISHA1, ProcessLaunchedByWMIID
| join kind=leftouter (
DeviceNetworkEvents
| where Timestamp > ago(LookupTime)
| where InitiatingProcessSHA1 in~ (GetRareWMIProcessLaunches)
|project DeviceName, ChildProcessTimestamp = Timestamp, ProcessLaunchedByWMI = tolower(InitiatingProcessFileName), ProcessLaunchedByWMICommandLine = tolower(InitiatingProcessCommandLine), ProcessLaunchedByWMICreationTime = InitiatingProcessCreationTime, ProcessLaunchedByWMISHA1 = tolower(InitiatingProcessSHA1), ProcessLaunchedByWMIID = InitiatingProcessId, WMIProcessRemoteIP = RemoteIP, WMIProcessRemoteURL = RemoteUrl
) on DeviceName, ProcessLaunchedByWMI, ProcessLaunchedByWMICommandLine, ProcessLaunchedByWMISHA1, ProcessLaunchedByWMIID
| where isnotempty(WMIProcessRemoteIP) or isnotempty(WMIchild)
| summarize ConnectedAddresses = make_set(WMIProcessRemoteIP), ConnectedURLs = make_set(WMIProcessRemoteURL), LaunchedProcessNames = make_set(WMIchild), LaunchedProcessCmdlines = make_set(WMIChildCommandline) by DeviceName, ProcessLaunchedByWMI, ProcessLaunchedByWMICommandLine, ProcessLaunchedByWMICreationTime, ProcessLaunchedByWMISHA1, ProcessLaunchedByWMIID


## WMIpersist
let LookupTime = 30d; 
DeviceEvents 
| where Timestamp > ago(LookupTime) 
| where ActionType == "WmiBindEventFilterToConsumer" 
| where AdditionalFields contains "ActiveScriptEventConsumer" 
| extend Consumer = extractjson("$.Consumer", AdditionalFields, typeof(string)),ESS = extractjson("$.ESS", AdditionalFields, typeof(string)), Namespace = extractjson("$.Namespace", AdditionalFields, typeof(string)), PossibleCause = extractjson("$.PossibleCause", AdditionalFields, typeof(string)) 
| extend ScriptText = extract(@'\ScriptText = (.*;)',1,PossibleCause), ScriptingEngine = extract(@'\ScriptingEngine = (.*;)',1,PossibleCause) 
| project-reorder Timestamp, DeviceName, Consumer, Namespace, ScriptingEngine, ScriptText


##DcomExec
let LookupTime = 30d;
DeviceNetworkEvents
| where Timestamp > ago(LookupTime)
| where InitiatingProcessFileName =~ "explorer.exe"
| where ActionType == 'InboundConnectionAccepted' 
| project InboundConnTimestamp = Timestamp, DeviceName, InboundConnectionToExplorer = RemoteIP, InitiatingProcessFileName, InitiatingProcessCreationTime, InitiatingProcessId
| join kind=leftouter (
DeviceProcessEvents
| where Timestamp > ago(LookupTime)
| where InitiatingProcessFileName =~ "explorer.exe"
| project ProcessStartTimestamp = Timestamp, DeviceName, StartedProcessCmdline = tolower(ProcessCommandLine), StartedProcessCreationTime = ProcessCreationTime, StartedProcessId = ProcessId, StartedProcessFileName = tolower(FileName), StartedProcessFolderPath = tolower(FolderPath), InitiatingProcessFileName, InitiatingProcessCreationTime, InitiatingProcessId
) on DeviceName, InitiatingProcessFileName, InitiatingProcessCreationTime, InitiatingProcessId
| where ProcessStartTimestamp between (InboundConnTimestamp .. (InboundConnTimestamp + 1m))
| join kind=leftouter ( 
DeviceProcessEvents 
| where Timestamp > ago(LookupTime) 
| where InitiatingProcessParentFileName =~ "explorer.exe"
|project DeviceName, ChildProcessTimestamp = Timestamp, StartedProcessCmdline = tolower(InitiatingProcessCommandLine), StartedProcessCreationTime = InitiatingProcessCreationTime, StartedProcessId = InitiatingProcessId, StartedProcessFileName = tolower(InitiatingProcessFileName), StartedProcessFolderPath = tolower(InitiatingProcessFolderPath), ChildProcessId= ProcessId, ChildProcessName = FileName, ChildProcessCommandLine = ProcessCommandLine 
) on DeviceName, StartedProcessCmdline, StartedProcessCreationTime, StartedProcessId, StartedProcessFileName, StartedProcessFolderPath
| join kind=leftouter ( 
DeviceNetworkEvents 
| where Timestamp > ago(LookupTime) 
| where InitiatingProcessParentFileName =~ "explorer.exe"
|project DeviceName, ChildProcessTimestamp = Timestamp, StartedProcessCmdline = tolower(InitiatingProcessCommandLine), StartedProcessCreationTime = InitiatingProcessCreationTime, StartedProcessId = InitiatingProcessId, StartedProcessFileName = tolower(InitiatingProcessFileName), StartedProcessFolderPath = tolower(InitiatingProcessFolderPath), RemoteIP, RemoteUrl
) on DeviceName, StartedProcessCmdline, StartedProcessCreationTime, StartedProcessId, StartedProcessFileName, StartedProcessFolderPath
| summarize ConnectedAddresses = make_set(RemoteIP), ConnectedUrl = make_set(RemoteUrl), ChildProcesses = make_set(ChildProcessName), ChildProcessCmdlines = make_set(ChildProcessCommandLine) by DeviceName, InitiatingSourceIP = InboundConnectionToExplorer, StartedProcessCmdline, StartedProcessCreationTime, StartedProcessId, StartedProcessFileName, StartedProcessFolderPath, Timestamp = InboundConnTimestamp
```

Query 3
```
DeviceEvents 
| where ActionType == "NamedPipeEvent" 
| where isnotempty(RemoteIP) 
| where AdditionalFields contains "winreg" 
| project DeviceName, Timestamp, RemoteIP, PipeName = extractjson("$.PipeName", AdditionalFields, typeof(string)), RemoteClientsAccess = extractjson("$.RemoteClientsAccess", AdditionalFields, typeof(string)), ShareName = extractjson("$.ShareName", AdditionalFields, typeof(string)), AdditionalFields 
| join ( 
DeviceRegistryEvents 
| where RegistryKey == @'HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\RemoteRegistry' 
| where RegistryValueName == @"Start" 
| where RegistryValueData == @"3" 
| project DeviceName, RegEditTimestamp = Timestamp, ActionType, RegistryKey, RegistryValueData, RegistryValueName, RegistryValueType 
) on DeviceName 
| where RegEditTimestamp < Timestamp 
| project-away DeviceName1


let lookuptime = 30d;
DeviceEvents
| where Timestamp >ago(lookuptime)
| where ActionType == 'NamedPipeEvent' 
| where AdditionalFields contains "atsvc"
| project DeviceName, Timestamp, DeviceId, RemoteIP, PipeName = extractjson("$.PipeName", AdditionalFields, typeof(string)), RemoteClientsAccess = extractjson("$.RemoteClientsAccess", AdditionalFields, typeof(string)), ShareName = extractjson("$.ShareName", AdditionalFields, typeof(string))
| join (
DeviceRegistryEvents
| where Timestamp >ago(lookuptime)
| where ActionType == 'RegistryKeyCreated' 
| where RegistryKey contains @"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\"
| project RegTimestamp = Timestamp, DeviceName, DeviceId, RegistryKey, RegistryValueData, RegistryValueName, RegistryValueType, InitiatingProcessFileName, InitiatingProcessCommandLine, InitiatingProcessId, InitiatingProcessCreationTime
) on DeviceName, DeviceId
| where RegTimestamp between ((Timestamp - 2m) .. (Timestamp + 2m))
| join (
DeviceProcessEvents
| where Timestamp >ago(lookuptime)
| where InitiatingProcessFileName =~ "svchost.exe"
| where InitiatingProcessCommandLine contains "Schedule"
| project DeviceId, DeviceName, FileName, ProcessCommandLine, ProcessId, ProcessStartTimestamp = Timestamp, InitiatingProcessFileName, InitiatingProcessCommandLine, InitiatingProcessId, InitiatingProcessCreationTime
) on DeviceName, DeviceId, InitiatingProcessFileName, InitiatingProcessCommandLine, InitiatingProcessId, InitiatingProcessCreationTime
| where ProcessStartTimestamp between ( Timestamp .. (Timestamp +2m))
| project DeviceName, Timestamp, StartedProcess = FileName, StartedProcessCommandLine = ProcessCommandLine, StartedProcessId = ProcessId, InitiatingProcessFileName, InitiatingProcessCommandLine, InitiatingProcessId, InitiatingProcessCreationTime, RegistryKey, RemoteIP, PipeName
```
