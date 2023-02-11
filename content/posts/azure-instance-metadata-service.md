---
title: "Helping Hand: The Azure Instance Metadata Service"
date: 2023-02-11T08:21:17Z
draft: false
description: "An intro to the little-known Azure Instance Metadata Service and a look at how it can be used to help aid automation"
keywords: ["Azure", "Instance", "Metadata", "Service", "Azure Instance Metadata Service", "IMDS", "Tags"]
tags: ["Azure", "Scripting", "Automation"]
---

_In this post I'll give a brief introduction to the Azure Instance Metadata Service and show how it can be used to help aid automation in conjunction with Azure Tags_

### Intro

The Azure Instance Metadata Service (IMDS) is a simple HTTP endpoint that can be accessed from within any Azure VM.  It returns a JSON representation of that machine which includes properties such as the `sku`, `osType`, `resourceId`, `location` (Azure region), and importantly for the purposes of this blog post, `tags`.

The docs can be found [here (Linux)](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/instance-metadata-service?tabs=linux) and [here (Windows)](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service?tabs=windows) - they are extremely similar though.

But the gist is, you can just send an HTTP GET request with the library of your choice to the special address `http://169.254.169.254/metadata/instance`, being sure to pass in the header `Metadata: True`, and the service will return the JSON data mentioned above.

---
### Extending IMDS Use Cases With Tags

Clearly the IMDS payload can be useful to get info about a VM you're logged on to.  Perhaps you're SSH'd on but don't have access to the Azure Portal to check its configuration?

But what if you wanted to use some kind of post-build automation and need a locally running script to easily access some custom data about your environment?

This is where IMDS can really help you out, in my opinion.

One example could be deploying an application into a custom image with a parameterised config setting.  If the image is going to be used in several environments, and you need to hand it a unique value _after_ the VM has been built, this could be solved with the help of IMDS and tags.

---
### Combine With Azure Policy

To get the full benefit of this type of workflow, you could consider creating tags with Azure Policy.

By creating a policy definition which deploys a tag with a unique name and making the tag's value a parameter to set at assignment based on the scope (management group/subscription/resource group) you will have a known, and consistent, tag key to lookup later on.

---
### Boilerplate Scripts

The below scripts can then be used to find the tag values from the Azure Instance Metadata Service (in this case, for a tag called `tag_key`) before pumping the resulting value out wherever it needs to go.  The value of the tag will be held in the `tagvalue/$tagvalue` variable.

#### Python

{{< gist thefisk c95561593218493382dc73ba661359b0 >}}

---
#### PowerShell

{{< gist thefisk 89ed48a76f77ba1a11348ce7851e1459 >}}

---
### Conclusion

The Azure Instance Metadata Service offers an easy way to lookup config data from within a VM, which can be particularly useful if you don't have access to the portal.  You can even use cURL for really quick and easy requests.

When combined with custom tags though, you can use IMDS as a conduit for extra config data you may want to access from within the VM.  It's less hassle than setting up a Key Vault and associated access policies, though obviously if you're looking to retrieve confidential secrets, that would be a better route to take.

---