---
layout: post
title: "New-xSelfSignedDscEncryptionCertificate"
modified:
categories: blog
excerpt:
tags: [DSC]
share: true
---

This blog details the New-xSelfSignedDscEncryptionCertificate for generating self signed certificates to support encrypting credentials in a Configuration

This [blog post](https://blogs.msdn.microsoft.com/powershell/2014/01/31/want-to-secure-credentials-in-windows-powershell-desired-state-configuration/) in the official PowerShell team blog talks about how to secure credentials in DSC. There were some limitations of this feature in WMF 4.0 which have been addressed in WMF 5.0. These are:

* **Ability to encrypt Credentials of any arbitrary length (and not limited to 121 characters)**
  This was required because the only way to store any secret is in the password field of a PSCredential object. Though passwords may not be that long, there were other information which users had to store in the password field which was encountering this limitation.
* **Validating if a appropriate certificate is being used**
  A digital certificate is categorized for use in only certain scenarios depending on how the ```KeyUsage``` and ```EnhancedKeyUsage``` properties have been defined. Previously DSC wasn't validating these and was using any certificate provided for encryption. Now this has also been fixed. The encryption piece of code in DSC now uses the ```Protect-CMSMessage``` cmdlet - which actually enforces these (*which is the right thing to do*). This also means the certificate must meet the following criteria:
	* **KeyUsage** must contain ```KeyEncipherment``` and ```DataEncipherment```
	* **EnhancedKeyUsage** must specify ```DocumentEncryption```


Though this makes the system secure it also means the barrier to adopting these features is slightly high. Certificates are inherently hard to deal with. Add in a few more constraints and it makes it all the more difficult. In order to minimize this barrier to entry, here is a simple PowerShell module which can help generate a self signed certificate that meets all the requirements

```PowerShell
Find-Module -Name xDscUtils -Repository psgallery
```
```
Version    Name                                Type       Repository           Description                                        
-------    ----                                ----       ----------           -----------                                        
0.1        xDSCUtils                           Module     PSGallery            Utilities for DSC   
```

You can install this using 

```PowerShell
Install-Module -Name xDscUtils -Repository PSGallery -Verbose 
```

This module provides the following simple cmdlet for generating a self-signed certificate that will help with encrypting Credentials in DSC

```PowerShell
New-xSelfSignedDscEncryptionCertificate -EmailAddress nanalakshmanan@gmail.com -ExportFilePath D:\MyCerts
```

Hopefully this should minimize the barrier to trying out credential encryption in DSC and limit the use of ```PSDscAllowPlainTextPassword``
