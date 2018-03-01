---
layout: post
title: "Targetting Automation Executions against Resource Groups"
modified:
categories: blog
excerpt:
tags: [AWS,SystemsManager]
share: true
image:
  feature:
---

This blog explains targetting Systems Manager Automation Executions against Resource Groups.

# Resource Groups

[AWS Systems Manager](https://aws.amazon.com/systems-manager/) introduces the concept of Resource Groups. [Resource Groups](https://aws.amazon.com/systems-manager/features/) are a way to create a logical group of resources. Resource Groups are living queries based on tag conditions. It is a **query** because it represents a filtered set of resource instances (*of same or different resource types*) from your environment based on a set of tag based conditions and it is **living** because the system periodically refreshes this filtered list. 

For instance, if you create a Resource Group representing an Auto Scaling Group, then the Resource group will contain instances in the ASG at any given point in time. Depending on various factors the number of instances in an ASG will vary. This gets reflected in the number of instances in a Resource Group. A more complex example of a resource group can represent all resources in your HR application including EC2 instance, S3 buckets, EBS volumes, etc.

A Resource group can therefore be used to represent all resources associated with a particular workload such as different layers of an application stack, or production versus development environments, etc. 

# Executing Automation targetting Resource Groups
**Target Type**

Automation Documents can be executed against a Resource Group. When a resource group contains resources of different types (EC2 instances, S3 buckets, etc) what exactly does targetting a resource group mean. Let us take a closer look at the same.

Automation Documents now support defining a target type. This implies what type of resources the execution of this document targets. The target type can be viewed using the following AWS CLI

````sh
aws ssm list-documents --filters 'Key=DocumentType, Values=Automation'                
````
Here is a sample

```json
{
  "TargetType": "/AWS::EC2::Instance", 
  "Name": "AWS-TerminateEC2Instance", 
  "PlatformTypes": [
      "Windows", 
      "Linux"
  ], 
  "DocumentVersion": "1", 
  "DocumentType": "Automation", 
  "Owner": "Amazon", 
}
```
The PowerShell equivalent of the same is

```powershell
$Filter = New-Object Amazon.SimpleSystemsManagement.Model.DocumentKeyValuesFilter                
$Filter.Key='DocumentType' 
$Filter.Values='Automation'
Get-SSMDocumentList -Filter $Filter 
```

**Note** ```TargetType``` is not a mandatory parameter and only one target type can be specified for an automation document. 

The values for ```TargetType``` are the same as defined by AWS Cloudformation. All resource type definitions can be found [here](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html) 

**Execution targetting Resource Groups**

When an automation execution is targetted against a resource group, the Automation service will filter the resources from the resource group based on the target type of the automation document. It then applies rate control (*execute against N at a time*) for automation executions against these filtered set of resources. For instance, for the Automation document ```AWS-StopEC2Instance```, the target type is ```/AWS::EC2::Instance```. When this document is executed against a resource group, Automation service will filter all EC2 instances from the resource group, apply rate control and execute the automation.

Note that when no target type is defined on the document, there will be no filtering and the execution will be against all resources in the resource group.

Executing an automation document against a resource group can be invoked using the following AWS CLI

```sh
aws ssm start-automation-execution --document-name AWS-StopEC2Instance --targets Key=ResourceGroup,Values=resource-group-1 --target-parameter-name InstanceId
```

* ```Key=ResourceGroup``` indicates that the targetting is against a Resource Group
* ```Values=resource-group-1``` indicates the value of the resource group that is being targetted
* ```target-parameter-name InstanceId``` indicates the name of the parameter in the Automation document on which the targetting happens

The PowerShell equivalent of the same is below

```powershell
$target = New-Object Amazon.SimpleSystemsManagement.Model.Target                                                                              
$target.Key='ResourceGroup'                                                                                                                   
$target.Values='resource-group-1'

Start-SSMAutomationExecution -DocumentName 'AWS-StopEC2Instance' -Target $target -TargetParameterName 'InstanceId'
```