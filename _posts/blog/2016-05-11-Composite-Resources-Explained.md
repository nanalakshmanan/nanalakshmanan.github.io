---
layout: post
title: "Composite Resources Explained"
modified:
categories: blog
excerpt:
tags: [DSC]
share: true
image:
  feature:
---

Composite configurations come in handy in a lot of scenarios and provide a layer of abstraction on resources. This blog explains composite resources in PowerShell DSC.

Any configuration in DSC can be exposed as a resource for other configurations to consume. This configuration is called a composite resource. The parameters of the configuration becomes the properties of the resource. Packaging the configuration in a specific way exposes the configuration as a resource. 

Composite resources are identified as such when resources are discovered

```PowerShell
Get-DscResource -Name WindowsFeatureSet

```

```
ImplementedAs   Name                      ModuleName                     Version    Properties                                        
-------------   ----                      ----------                     -------    ----------                                        
Composite       WindowsFeatureSet         PSDesiredStateConfiguration    1.1        {DependsOn, Name, Ensure, Source...
```

They can be used in a configuration like any other resource

```PowerShell

configuration test
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    WindowsFeatureSet x
    {
        Name   = 'FS-FileServer', 'Hyper-V'
        Ensure = 'Present'
    }
}

```

Though composite resources can be used in configuration, on compilation, the composite resource - which is a configuration in itself - gets compiled to its resources. If we compile the above configuration, this is how the contents of the configuration file looks like

```
/*
@TargetNode='localhost'
@GeneratedBy=nanalakshmanan
@GenerationDate=05/11/2016 11:35:42
@GenerationHost=NANA-TOUCH
*/

instance of MSFT_RoleResource as $MSFT_RoleResource1ref
{
 ResourceID = "[WindowsFeature]Resource0::[WindowsFeatureSet]x";
 Ensure = "Present";
 SourceInfo = "::2::1::WindowsFeature";
 Name = "FS-FileServer";
 ModuleName = "PSDesiredStateConfiguration";
 ModuleVersion = "1.0";
 ConfigurationName = "test";
};

instance of MSFT_RoleResource as $MSFT_RoleResource2ref
{
 ResourceID = "[WindowsFeature]Resource1::[WindowsFeatureSet]x";
 Ensure = "Present";
 SourceInfo = "::8::1::WindowsFeature";
 Name = "Hyper-V";
 ModuleName = "PSDesiredStateConfiguration";
 ModuleVersion = "1.0";
 ConfigurationName = "test";
};

instance of OMI_ConfigurationDocument
{
 Version="2.0.0";
 MinimumCompatibleVersion = "1.0.0";
 CompatibleVersionAdditionalProperties= {"Omi_BaseResource:ConfigurationName"};
 Author="nanalakshmanan";
 GenerationDate="05/11/2016 11:35:42";
 GenerationHost="NANA-TOUCH";
 Name="test";
}

```

As you can see, the composite configuration WindowsFeatureSet got compiled and you can find the underlying resource **WindowsFeature**.

Since a composite resource is a configuration, it never gets loaded by the LCM. (*A configuration needs to be compiled and the resources it calls are the ones that the LCM needs to load*). This is why an ```Invoke-DscResource``` on a composite resource will not work and you should call it on the underlying resource. 

