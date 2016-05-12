---
layout: post
title: "Historical Job Logs"
modified:
categories: blog
excerpt:
tags: [DSC]
share: true
image:
  feature:
---

DSC persists the job details on disk in a specific location. This blog explains how to retrieve the contents at a later point.

This [blog](http://nanalakshmanan.com/blog/DSC-get-job-details-post-reboot/) explains how to setup DSC so you can obtain results of a configuration run post a reboot. That is valuable when you want to capture the information post a reboot during a configuration run. However, there are cases when you may want to obtain the results that happened in the past or let the configuration run post a reboot (*without a client connected*) and obtain the results later.

DSC persists all of the log information as JSON files in its configuration status folder based on the job id. A job id is unique for a configuration run, consistency check, etc. You can look at all of the job ids from the configuration status object (returned by ```Get-DscConfigurationStatus``` cmdlet)

```PowerShell
Get-DscConfigurationStatus | Get-Member

```

```
   TypeName: 
Microsoft.Management.Infrastructure.CimInstance#root/Microsoft/Windows/DesiredStateConfiguration/MSFT_DSCConfigurationStatus

Name                       MemberType   Definition                                                                                      
----                       ----------   ----------                                                                                      
Clone                      Method       System.Object ICloneable.Clone()                                                                
Dispose                    Method       void Dispose(), void IDisposable.Dispose()                                                      
Equals                     Method       bool Equals(System.Object obj)                                                                  
GetCimSessionComputerName  Method       string GetCimSessionComputerName()                                                              
GetCimSessionInstanceId    Method       guid GetCimSessionInstanceId()                                                                  
GetHashCode                Method       int GetHashCode()                                                                               
GetObjectData              Method       void GetObjectData(System.Runtime.Serialization.SerializationInfo info, System.Runtime.Serial...
GetType                    Method       type GetType()                                                                                  
DurationInSeconds          Property     uint32 DurationInSeconds {get;}                                                                 
Error                      Property     string Error {get;}                                                                             
HostName                   Property     string HostName {get;}                                                                          
IPV4Addresses              Property     string[] IPV4Addresses {get;}                                                                   
IPV6Addresses              Property     string[] IPV6Addresses {get;}                                                                   
JobID                      Property     string JobID {get;}                                                                             
LCMVersion                 Property     string LCMVersion {get;}                                                                        
Locale                     Property     string Locale {get;}                                                                            
MACAddresses               Property     string[] MACAddresses {get;}                                                                    
MetaConfiguration          Property     CimInstance#Instance MetaConfiguration {get;}                                                   
MetaData                   Property     string MetaData {get;}                                                                          
Mode                       Property     string Mode {get;}                                                                              
NumberOfResources          Property     uint32 NumberOfResources {get;}                                                                 
PSComputerName             Property     string PSComputerName {get;}                                                                    
RebootRequested            Property     bool RebootRequested {get;}                                                                     
ResourcesInDesiredState    Property     CimInstance#InstanceArray ResourcesInDesiredState {get;}                                        
ResourcesNotInDesiredState Property     CimInstance#InstanceArray ResourcesNotInDesiredState {get;}                                     
StartDate                  Property     CimInstance#DateTime StartDate {get;}                                                           
Status                     Property     string Status {get;}                                                                            
Type                       Property     string Type {get;}                                                                              
ToString                   ScriptMethod System.Object ToString();                                                                       

```

Here is a way to look at all job ids

```PowerShell
Get-DscConfigurationStatus -All | Format-Table Status, StartDate, Type, JobId -AutoSize

```

```
Status  StartDate            Type    JobId                                 
------  ---------            ----    -----                                 
Success 5/12/2016 6:35:05 AM Initial {535A36CD-1846-11E6-AD61-8C4019F5FC80}
Success 5/12/2016 6:34:47 AM Initial {4B1F0CB6-1846-11E6-AD61-8C4019F5FC80}

```

Once you obtain the job id of a specific job you are interested in, you can locate the json files corresponding to it from the configuration status folder

```PowerShell
$jobid = '{535A36CD-1846-11E6-AD61-8C4019F5FC80}'
dir "$env:windir\system32\configuration\configurationstatus\$jobid*.json"

```

```
    Directory: C:\Windows\system32\configuration\configurationstatus


Mode                LastWriteTime         Length Name                                                                                   
----                -------------         ------ ----                                                                                   
-a----        5/12/2016   6:35 AM           2508 {535A36CD-1846-11E6-AD61-8C4019F5FC80}-0.details.json                                  

```

```0``` indicates the initial run. The suffix gets incremented post every reboot (*```1``` after the first reboot and so on). 

The json file has all the contents of the job logs

```PowerShell
$Contents = Get-Content .\'{535A36CD-1846-11E6-AD61-8C4019F5FC80}-0.details.json' -Raw -Encoding Unicode
ConvertFrom-Json $contents
```

```
time                         type    message                                                                                                     
----                         ----    -------                                                                                                     
2016-05-12T06:35:05.455-7:00 verbose [NANA-TOUCH]: LCM:  [ Start  Set      ]                                                                     
2016-05-12T06:35:05.503-7:00 verbose [NANA-TOUCH]: LCM:  [ Start  Resource ]  [[File]f]                                                          
2016-05-12T06:35:05.503-7:00 verbose [NANA-TOUCH]: LCM:  [ Start  Test     ]  [[File]f]                                                          
2016-05-12T06:35:05.507-7:00 verbose [NANA-TOUCH]:                            [[File]f] The destination object was found and no action is requ...
2016-05-12T06:35:05.507-7:00 verbose [NANA-TOUCH]: LCM:  [ End    Test     ]  [[File]f]  in 0.0040 seconds.                                      
2016-05-12T06:35:05.507-7:00 verbose [NANA-TOUCH]: LCM:  [ Skip   Set      ]  [[File]f]                                                          
2016-05-12T06:35:05.507-7:00 verbose [NANA-TOUCH]: LCM:  [ End    Resource ]  [[File]f]                                                          
2016-05-12T06:35:05.511-7:00 verbose [NANA-TOUCH]: LCM:  [ End    Set      ]                                                                     
2016-05-12T06:35:05.511-7:00 verbose [NANA-TOUCH]: LCM:  [ End    Set      ]    in  0.0560 seconds.    
```