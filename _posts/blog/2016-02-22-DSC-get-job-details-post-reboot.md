---
layout: post
title: "Get DSC Job Streams post Reboot"
modified:
categories: blog
excerpt:
tags: [DSC]
share: true
image:
  feature:
---

This blog explains how to get various streams after a DSC node reboots.

When you execute a DSC configuration, sometimes a reboot may be required. The default behavior of DSC is to wait for the user to explicitly reboot and then proceed automatically post reboot.  You can view the defaults using the following cmdlet

```powershell
Get-DscLocalConfigurationManager

ActionAfterReboot              : ContinueConfiguration
AgentId                        : 29C00365-9860-11E5-9F8A-F4B7E2CC0DA1
AllowModuleOverWrite           : False
CertificateID                  : 
ConfigurationDownloadManagers  : {}
ConfigurationID                : 
ConfigurationMode              : ApplyAndMonitor
ConfigurationModeFrequencyMins : 15
Credential                     : 
DebugMode                      : {NONE}
DownloadManagerCustomData      : 
DownloadManagerName            : 
LCMCompatibleVersions          : {1.0, 2.0}
LCMState                       : PendingConfiguration
LCMStateDetail                 : 
LCMVersion                     : 2.0
StatusRetentionTimeInDays      : 10
PartialConfigurations          : 
RebootNodeIfNeeded             : False
RefreshFrequencyMins           : 30
RefreshMode                    : PUSH
ReportManagers                 : {}
ResourceModuleManagers         : {}
PSComputerName                 : 

```

The **ActionAfterReboot** property indicates what DSC would do after a reboot. The **RebootNodeIfNeeded** property indicates what DSC would do when a configuration requires a reboot. 

In cases where you want to obtain the information from the node post a reboot, set DSC to not proceed after reboot. This can be done by using the following meta configuration sample

```powershell
[DscLocalConfigurationManager()]
configuration Settings
{
    Settings
    {
        ActionAfterReboot  = 'StopConfiguration'
        RebootNodeIfNeeded = $false
    }
}
 
```
Then re-apply the existing configuration using the following command

```powershell
Start-DscConfiguration -Wait -UseExisting -Verbose
```


