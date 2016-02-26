---
layout: post
title: "Converging to a Desired State"
modified:
categories: blog
excerpt:
tags: [DSC]
share: true
image:
  feature:
---

This blog explains how DSC attempts to converge to a desired state when a configuration is applied.

When DSC LocalConfigurationManager (LCM aka the DSC agent) is delivered a configuration document, it is staged as **Pending** (*internally LCM stores this document as pending.mof*). This indicates to the LCM that it needs to work on this configuration until the desired state is reached.  Once the desired state is reached, the document is staged as **Current** (*internally LCM moves this document to current.mof and the existing current.mof as previous.mof*). 

A configuration document is a graph with dependencies. When the LCM processes a configuration document, it translates the graph to an ordered list and begins to work on resources starting from the first one. The most important aspect is that this is how the LCM **always** processes a document - whether it is applying a new configuration or checking the consistency of the system against the current configuration or when it is resuming after a reboot. This is what keeps the model simple. For every resource, the LCM first tests if the resource is in the desired state by calling the Test method on the resource (*Test-TargetResource method for mof based resources and Test() method on the class for class based resources*). If the resource is in the desired state, it moves to the next resource. If the resource is not in the desired state, it then calls the Set method to get the resource to the desired state. This is what gives the ability for LCM to effectively resume from where it left off - since the resources already acted upon will not be acted upon again. 

Sometimes the LCM will not be able to get the system to a desired state in one convergence cycle. For example, a ```WindowsFeature``` resource requested a reboot or a source network share for a ```File``` resource was not available. Any time the LCM starts it checks for the presence of a **pending.mof** which indicates that the system has not reached the desired state - in which case the LCM tries to act on the same. This provides the advantage that any reboots as well as transient errors  (*like network share not being available*) are handled and the system reaches the desired state. 

The above also means that LCM keeps attempting to act on a **pending** configuration so long as there is one. The only way to stop the LCM from acting on one is to remove the configuration document. This can be accomplished using the following cmdlet

```PowerShell
 Remove-DscConfigurationDocument -Stage Pending 
```

***Note*** *If the LCM crashes for whatever reason, the DSC timer process will attempt to restart the LCM for up to 3 times. This provides some resiliency against any transient issues in the LCM*

