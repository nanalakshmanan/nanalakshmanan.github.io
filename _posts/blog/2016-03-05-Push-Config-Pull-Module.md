---
layout: post
title: "Setting up a Simple File share based Module Repository for DSC configurations"
modified:
categories: blog
excerpt:
tags: [DSC]
share: true
---

This blog explains how to set up a simple file share based repository for DSC to install required modules for a configuration.

In PowerShell 5.0, DSC can pull down modules as required from a repository even when a configuration is pushed (*delivered via Start-DscConfiguration*). This is especially useful in scenarios when you do not want to setup a pull server. This method also eliminates the module deployment problem in the ```Push``` mode.

Let us walk through the whole process using a simple example. Here is a simple configuration that uses a resource downloaded from the [PowerShell gallery](http://www.powershellgallery.com/)

```PowerShell
configuration Sample
{
    Import-DscResource -ModuleName xPSDesiredStateConfiguration

    xArchive SampleArchive
    {
        Path             = 'D:\Nana\TempContents'
        Destination      = 'D:\Nana\official\temp\tempcontents.zip'
        CompressionLevel = 'Fastest'
        DestinationType  = 'File'
    }
}

```

The above configuration requires the ```xPSDesiredStateConfiguration``` module on the target node in order to enact the configuration. 

In PowerShell 4.0, if the configuration ```RefreshMode``` is ```Push``` then the module needs to be available in the node (*in case of pull, it can pull down the module*). Let us see how this can be simplified in PowerShell 5.0

**Setting up File Share**
The first step in the process is setting up the file share with anonymous access. We can setup the file share using the following cmdlet

```PowerShell
New-SmbShare -Name DSCShare -Path D:\Nana\Official\DscShare -Description 'File share for DSC modules' -ReadAccess everyone -Verbose

```

**Making Modules Available**
Once we have the share setup, we need to ensure that the modules are packaged and made available as required by the DSC pull protocol. These are easily automatable steps and we can see how to achieve the same

1. If you are downloading modules from the PowerShell Gallery (like in this case), save the module to a disk location

```PowerShell
Find-Module xPSDesiredStateConfiguration | Save-Module -Path D:\Nana\Official\DownloadedModules -Verbose
```

2. Then we need to zip this module as required by DSC pull. Here are a few requirements on the zip structure
   * The zip file should be named <ModuleName_Version>.zip. So in our case it needs to be xPSDesiredStateConfiguration_3.7.0.0.zip
   * The contents of the module should be at the root level. That simply means the zip file cannot contain a folder 3.7.0.0 but should contain all the contents within it

We can achieve the above using the following PowerShell cmdlet 

```PowerShell
Compress-Archive -Path D:\Nana\Official\DownloadedModules\xPSDesiredStateConfiguration\3.7.0.0\* -DestinationPath D:\Nana\Official\DscShare\xPSDesiredStateConfiguration_3.7.0.0.zip -Verbose
```

3. Once we have the zip file created, we then need to create its checksum

```PowerShell
New-DscChecksum -Path D:\Nana\Official\DscShare\xPSDesiredStateConfiguration_3.7.0.0.zip -OutPath D:\Nana\Official\DscShare -Verbose
```

**Setting up Target Node to Pull Modules**
Now that we have setup the file share we need to let DSC know that it needs to pull these modules from the share. No surpise that we will do this using the LocalConfigurationManager settings

```PowerShell
[DscLocalConfigurationManager()]
configuration PushToPull
{
    Settings
    {
        RefreshMode = 'Push'
        # A configuration Id needs to be specified, known bug
        ConfigurationID = '3a15d863-bd25-432c-9e45-9199afecde91'
    }

    ResourceRepositoryShare FileShare
    {
        SourcePath = '\\nana-hyperv2\dscshare\'
    }
}
```

We need to then compile and deliver this to the target node

```PowerShell

PushToPull -OutputPath D:\Nana\Official\Temp\pushtopull -Verbose

```

```
    Directory: D:\Nana\Official\Temp\pushtopull


Mode                LastWriteTime         Length Name                                                      
----                -------------         ------ ----                                                      
-a----         3/5/2016  12:54 PM           1818 localhost.meta.mof 
```

```PowerShell
Set-DscLocalConfigurationManager -Path D:\Nana\Official\Temp\pushtopull -Verbose
```

```
VERBOSE: Performing the operation "Start-DscConfiguration: SendMetaConfigurationApply" on target "MSFT_DSCLo
calConfigurationManager".
VERBOSE: Perform operation 'Invoke CimMethod' with following parameters, ''methodName' = SendMetaConfigurationApply,'className' = MSFT_DSCLocalConfigurationManager,'namespaceName' = root/Microsoft/Windows/DesiredStateConfiguration'.
VERBOSE: An LCM method call arrived from computer NANA-TOUCH with user sid S-1-5-21-2127521184-1604012920-1887927527-2581743.
VERBOSE: [NANA-TOUCH]: LCM:  [ Start  Set      ]
VERBOSE: [NANA-TOUCH]: LCM:  [ Start  Resource ]  [MSFT_DSCMetaConfiguration]
VERBOSE: [NANA-TOUCH]: LCM:  [ Start  Set      ]  [MSFT_DSCMetaConfiguration]
VERBOSE: [NANA-TOUCH]: LCM:  [ End    Set      ]  [MSFT_DSCMetaConfiguration]  in 0.0400 seconds.
VERBOSE: [NANA-TOUCH]: LCM:  [ End    Resource ]  [MSFT_DSCMetaConfiguration]
VERBOSE: [NANA-TOUCH]: LCM:  [ End    Set      ]
VERBOSE: [NANA-TOUCH]: LCM:  [ End    Set      ]    in  0.0890 seconds.
VERBOSE: Operation 'Invoke CimMethod' complete.
VERBOSE: Set-DscLocalConfigurationManager finished in 0.277 seconds.

```

**Testing pulling down Modules**
Now that we have everything setup, let us set our setup using the sample configuration at hand.

**Compile the sample in a node that has the xPSDesiredStateConfiguration module**

```PowerShell
Start-DscConfiguration -Path D:\Nana\Official\Temp\sample -Wait -Force -Verbose

```

```
VERBOSE: Perform operation 'Invoke CimMethod' with following parameters, ''methodName' = SendConfigurationApply,'className' = MSFT_DSCLocalConfigurationManager,'namespaceName' = root/Microsoft/Windows/DesiredStateConfiguration'.
VERBOSE: An LCM method call arrived from computer NANA-TOUCH with user sid S-1-5-21-2127521184-1604012920-1887927527-2581743.
VERBOSE: [NANA-TOUCH]: LCM:  [ Start  Set      ]
VERBOSE: [NANA-TOUCH]:                            [DSCEngine] Module xPSDesiredStateConfiguration_3.7.0.0.zip downloaded.
VERBOSE: [NANA-TOUCH]:                            [DSCEngine] Resource xPSDesiredStateConfiguration version 3.7.0.0 installed.
VERBOSE: [NANA-TOUCH]: LCM:  [ Start  Resource ]  [[xArchive]SampleArchive]
VERBOSE: [NANA-TOUCH]: LCM:  [ Start  Test     ]  [[xArchive]SampleArchive]
VERBOSE: [NANA-TOUCH]:                            [[xArchive]SampleArchive] Test Result True
VERBOSE: [NANA-TOUCH]: LCM:  [ End    Test     ]  [[xArchive]SampleArchive]  in 1.0780 seconds.
VERBOSE: [NANA-TOUCH]: LCM:  [ Skip   Set      ]  [[xArchive]SampleArchive]
VERBOSE: [NANA-TOUCH]: LCM:  [ End    Resource ]  [[xArchive]SampleArchive]
VERBOSE: [NANA-TOUCH]: LCM:  [ End    Set      ]
VERBOSE: [NANA-TOUCH]: LCM:  [ End    Set      ]    in  11.7140 seconds.
VERBOSE: Operation 'Invoke CimMethod' complete.
VERBOSE: Time taken for configuration job to complete is 12.068 seconds
```