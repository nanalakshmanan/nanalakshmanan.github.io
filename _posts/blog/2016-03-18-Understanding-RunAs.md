---
layout: post
title: "Understanding RunAs in DSC"
modified:
categories: blog
excerpt:
tags: [DSC]
share: true
---

DSC runs configurations as the System user. Why? How to change that ?

The DSC Agent, also called the **LocalConfigurationManager (LCM)**, performs any operation in the **LocalSystem** context. There are two main reasons as to why it is done this way:
* Running something in local system context provides the LCM with necessary privileges to configure the system
* LCM can proceed after a system reboot without requiring an interactive user session or saving user credentials (*which is needed to run as a specified user context*)


However the downside of the above is that the LCM cannot perform any operation that requires access to user specific resources like the users profile or registry key or a specific operation that needs to be performed in a user context. This is fine in most of the configuration scenarios. However there are a few that does require access to these resources or require to be run as a specific user (*example: some operations in SharePoint configuration need to run as the SharePoint setup account*).

DSC supports running as a specific user at a resource level. What this means is, it is possible to specify that a particular resource in a configuration needs to be run in a specific user context by specifying the user credentials. This is done by specifying the credentials against the **PsDscRunAsCredential** property which is available for all PowerShell resources (*similar to DependsOn*)

```PowerShell

    xSPManagedAccount xSharePoint 
    { 
        AccountName          = $ServicePoolManagedAccount.UserName 
        Account              = $ServicePoolManagedAccount 
        PsDscRunAsCredential = $SPSetupAccount 
        DependsOn            = $FarmWaitTask 
    } 

```

When a resource is specified to run under a user context, LCM creates a process with the users token (**using PowerShell remoting to the localhost**) and runs the resource within the context of that process. Since LCM has already created the context the resource doesn't have access to the Credentials of the user specified. However the resource can get access to the UserName of the credentials as under:
* Script resource (using schema.mof): $PsDscContext.RunAsUser  
* Class based resource : $global:PsDscContext.RunAsUser  

