---
title: "Storage Account Public Access: Terraform Oddity"
date: 2022-02-17T20:27:34Z
draft: false
tags: ["Terraform", "Azure"]
description: "Odd behaviour with the AzureRM Terraform provider"
keywords: ["Azure", "Terraform", "Storage Accounts", "AzureRM", "Blob Public Access"]
---

> When things don't go quite as you expect

### Come on Terraform, play nicely

Terraform is a fantastic tool for managing Infrastructure as Code. It's pretty much the industry standard and allows for things like cloud provider agnostic code.  Usually, if there's an issue with some Terraform behaviour, I find the source of the problem lies between the keyboard and the chair (user error). However, a recent experience led me to find that this isn't _always_ the case.

---
### Setting the Scene

It's best practice to make your Azure Storage Accounts as secure as you can and one of the ways you can do this is to disable Public Blob Access (unless your solution requires it, of course). Doing so will prevent any new containers being created with 'Blob' public access level, but also render existing containers unreachable over the internet; Azure will return an [HTTP 409](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/409), which may or may not be the best status code for the job...anyway...

---
### The strange behaviour

To disable Blob public access within Terraform is simply a case of finding your [AzureRM_Storage_Account](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/storage_account) and setting allow_blob_public_access to false as shown in line 8 below: -

{{< gist thefisk 941f0884ff6530ec1246bf97187949e3 >}}

However, in my real world situation, the setting didn't want to apply. I noticed something I'd never seen before as well; after running a Terraform Apply, and inspecting the state file, the Storage Account resource showed as having the setting set to false, as per my Terraform configuration. This was in conflict with the actual resource though, which had Blob public access still enabled!  Furthermore, a subsequent Terraform Plan/Apply seemed to think everything was hunky-dory. What gives?

> Enabled in Azure portal
![Storage Account showing Blob public access enabled](/img/resource_overview.png)

> Disabled in Terraform state
![Terraform State Show, showing Blob public access disabled](/img/tf_state_show.png)

---
### The (probable) answer

When comparing Storage Accounts that did and did not exhibit this behaviour, I noticed a difference in their properties when looking at their JSON representation. The 'bad' Storage Accounts were missing the allowBlobPublicAccess property altogether as can be seen below: -

> Good Storage Account
![Storage Account with allowBlobPublicAccess property](/img/prop_present.png)

> Bad Storage Account
![Storage Account with allowBlobPublicAccess missing](/img/prop_missing.png)

To fix this, it's simply a case of changing the Blob public access setting within the Portal. This will create the property which Terraform can then accurately read and manage.

My assumption here is that this will likely be an issue that only affects older Storage Accounts as I suspect an old API version may not have explicitly set this property.  It may not have even existed at the time the Storage Accounts were created.  What seems to be going on though, is that Terraform is inferring that *its* default behaviour (Blob public access set to disabled) is the behaviour represented by a lack of property on the resource. Either that, or the AzureRM provider is mishandling the missing property when it scans the resource.

This is something that (hopefully) won't come up very often but I wanted to take the time to write about it because it's a little different to the norm and it might help someone, somewhere.

---