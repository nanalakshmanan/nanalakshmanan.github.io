---
layout: post
title: "Import-DscResource Explained"
modified:
categories: blog
excerpt:
tags: [DSC]
share: true
---

This blog explains the need for Import-DscResource in a configuration and what happens under the covers

When we write a ```configuration``` in DSC, often the first statement is to specify which resources to import and from which modules (*in PowerShell 4.0, the built-in resources did not require this. Starting PowerShell 5.0 this will now write a warning asking the author to use an Import-DscResource statement*)

```PowerShell

Configuration MyServices
{
	Import-DscResource -ModuleName xPSDesiredStateConfiguration
}

```

When a script is parsed and the parser encounters the ```Import-DscResource``` keyword, it then proceeds to identify the resources in the specified modules. In the above piece of code, the parser looks at ```xPSDesiredStateConfiguration``` module and identifies the resources in it. 

It then creates keywords which map to these resources and makes them available within the scope of the configuration block. In our example the following keywords will be generated - ```xArchive, xDscWebService, xGroup, xPackage, xPSEndpoint, xRegistry, xRemoteFile, xService, xWindowsOptionalFeature, xWindowsProcess```

This is what helps an author use those resources in the configuration. They are keywords since they have a special meaning when the configuration is compiled into a mof document. Since they are generated dynamically they are also called as **dynamic keywords**.

Given that these resources are discovered during parse time - or at the time when you are authoring a configuration in ISE - it is important that this operation gets done as quickly as possible. This is why dynamic keywords are not generated for all available resources. This also solves the conflict problem - the same resource defined in multiple module or multiple version of the same module. 

DSC resources can be authored in one of the following ways:
* A script based resource using a schema.mof (*typically used when a resource needs to work with PowerShell 4.0*)
* Class based resource (*new in PowerShell 5.0 and the recommended way*)

A script based resource has the schema.mof file in a specific location relative to the module folder. However a class based resource can be defined in any script file as part of a module. This is why class based resources need to be explicitly exported in ```DscResourcesToExport``` section of the module manifest and be available with the root module - so parsing of PowerShell script files or script module files can be kept to a minimum.
