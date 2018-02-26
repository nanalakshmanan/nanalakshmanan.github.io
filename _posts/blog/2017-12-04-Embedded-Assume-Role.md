---
layout: post
title: "PassRole permissions and Embedded AssumeRole in Automation Documents"
modified:
categories: blog
excerpt:
tags: [AWS,SystemsManager]
share: true
image:
  feature:
---

This blog explains how to embed AssumeRole in Automation documents and not require PassRole permissions.

AWS Systems Manager Automation is used to automate common maintenance and deployment tasks targetting various AWS resources. Automations are defined using Automation Documents. You can view a list of available Automation documents using the following CLI

````sh
aws ssm list-documents --filters 'Key=DocumentType, Values=Automation'                
````

The PowerShell equivalent of the same is

```powershell
$Filter = New-Object Amazon.SimpleSystemsManagement.Model.DocumentKeyValuesFilter                
$Filter.Key='DocumentType' 
$Filter.Values='Automation'
Get-SSMDocumentList -Filter $Filter 
```

In most Automation documents, you will find the following:

```json
"assumeRole": "{% raw %}{{ AutomationAssumeRole }}{% endraw %}",
"parameters": {
  "AutomationAssumeRole": {
    "type": "String",
    "description": "(Optional) The ARN of the role that allows Automation to  perform the actions on your behalf.",
    "default": ""
  }                    
````
```AssumeRole``` essentially is an IAM service role that lets the Automation execution perform actions on AWS resources when the user invoking the same has restricted or no access to the same. Automation expects that you pass the ARN of an IAM role for this parameter.

One aspect to note is that a user should have ```PassRole``` permission to the role being passed. 
The permissions associated with the role will have something like the following

```json
"Version": "2012-10-17",
"Statement": [
  {
    "Action": [
      "iam:PassRole"
    ],
    "Resource": [
      "arn:aws:iam::123456789012:role/Nana"
    ],
    "Effect": "Allow"
  }
]
````
If a user does not ```PassRole``` permission then  it results in the following error

```powershell
One or more errors occurred. (User: arn:aws:iam::123456789012:user/NanaNoPassRole is not authorized to perform: iam:PassRole on resource: arn:aws:iam::123456789012:role/TestAssumeRole)
```

This is by design and to ensure that a user does not pass a role that the user does not have access to. However, it is not required that the ```AssumeRole``` need to be passed as a parameter even though it is very common to do so.

Automation provides the ability to embed the ```AssumeRole```. When embedded in the following format, the user it not required to have ```PassRole``` permissions on the role. (**Note** The user still needs to have permissions to start automation execution using the document)

```json
"assumeRole": "arn:aws:iam::123456789012:role/TestAssumeRole"
```

Automation allows providing Account Id as 
```` 
{% raw %}{{global:ACCOUNT_ID}}{% endraw %}
````. However, if the ```AssumeRole``` is specified in this format, then ```PassRole``` permission is required. 