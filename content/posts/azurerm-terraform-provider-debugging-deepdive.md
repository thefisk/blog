---
title: "Azurerm Terraform Provider Debugging Deepdive"
date: 2022-08-18T08:02:26+01:00
draft: false
description: "Debugging an issue in the AzureRM Terraform provider: a step-by-step guide"
keywords: ["Debugging", "Go", "Golang", "Terraform", "AzureRM"]
tags: ["Terraform", "Go", "Debugging", "Azure"]
---

_In this post I'll walk through how I found the root cause of an issue within the AzureRM Terraform provider that had been open on Github for 7 months and why the length of an open issue doesn't necessarily correlate to its complexity_

> For details on how to set up a debugging environment for the AzureRM Terraform Provider, check out my [last post](../debugging-azurerm-terraform-provider)

The below was produced using AzureRM provider v3.15.1

---
### Setting the Scene

At my place of work we deploy all of our Azure infrastructure through Terraform where possible; I don't think I'm breaking any trade secrets when I say that.  We regularly deploy virtual machines and virtual machine scale sets (VMSS) in this way.  In Azure though, there are two orchestration modes for VMSS deployments; Uniform and Flexible.  One of the main differences between the two is the type of VM instance that Azure deploys as part of the scale set.  In the case of Uniform orchestration, Azure deploys a VM resource of type `Microsoft.compute/virtualmachinescalesets/virtualmachines` whereas for Flexible, it deploys a VM of type `Microsoft.compute/virtualmachines`.  A big benefit of the latter is that this is the same type as is deployed for a standard VM and so any automation that works with normal VMs should work fine with Flexible VM instances.

In terms of Terraform deployments, you can use resource type [azurerm_linux_virtual_machine_scale_set](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/linux_virtual_machine_scale_set) (or equivalent Windows resource) for a traditional Unform deployment, or [azurerm_orchestrated_virtual_machine_scale_set](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/orchestrated_virtual_machine_scale_set) for a Flexible deployment.

However, when I looked at the viability of deploying a Flexible VMSS I hit a blocker with the provider.  When the `source_image_id` param was set, an error was returned by Terraform: - 

`creating Orchestrated Virtual Machine Scale Set: (Name "example2-VMSS" / Resource Group "rg-dev"): compute.VirtualMachineScaleSetsClient#CreateOrUpdate: Failure sending request: StatusCode=400 -- Original Error: Code="InvalidParameter" Message="Cannot specify user image overrides for a disk already defined in the specified image reference." Target="osDisk"`

But leaving the `source_image_id` param off didn't work, and neither did removing the `os_disk` block.  So what gives?

---
### Initial Investigation

My first port of call, as always, was Google (other search engines are available) and I found the issue had already been raised on the AzureRM provider GitHub account: [Issue #14820](https://github.com/hashicorp/terraform-provider-azurerm/issues/14820).  This was raised in January 2022 and I was working on my viability study above in late July, so it had been outstanding for well over 6 months. 

In March, GitHub user evandeworp added [this](https://github.com/hashicorp/terraform-provider-azurerm/issues/14820#issuecomment-1059605863) comment which seemed to highlight an important point; the provider didn't seem to be sending the imageReference.id to Azure's Management API.

---
### Inspecting for Myself

Following the steps laid out in my [previous post](../debugging-azurerm-terraform-provider) I set up an environment to debug the issue.  The first thing I did when debugging was to verify _evandeworp's_ comment about missing data in the PUT request.  In the terminal in which the provider was running, I noticed (among a wall of text) something that looked like an HTTP request as below: -

{{< gist thefisk b4b8942694a4aea58e5e81110f760f96 >}}

Removing the headers and all forward slashes from the request revealed the much more digestable JSON body below.  storageProfile, on line 21, is indeed missing the imageReference object, despite it being included in the .tf file.

{{< gist thefisk c7fc43ec4285a03b491401ed36c5aca0 >}}

---
### Source Code Structure

To try and find out what Terraform is doing, I needed to set some breakpoints, so the first task was to get a loose handle on the structure of the source code.  Our AzureRM resources can be found in [/internal/services](https://github.com/hashicorp/terraform-provider-azurerm/tree/main/internal/services).  Within /services, services are grouped by _type_, so Storage Accounts are under /storage, AppGateways are under /network.  Virtual Machine Scale Sets are under /compute - so that's where we can find the magic sauce for our Flexible VMSSs.

There are two files of interest in here, [orchestrated_virtual_machine_scale_set_resource.go](https://github.com/hashicorp/terraform-provider-azurerm/blob/5fd32b3b3cf8a4a1891dee79e8890a125a2f36ce/internal/services/compute/orchestrated_virtual_machine_scale_set_resource.go) and [orchestrated_virtual_machine_scale_set.go](https://github.com/hashicorp/terraform-provider-azurerm/blob/5fd32b3b3cf8a4a1891dee79e8890a125a2f36ce/internal/services/compute/orchestrated_virtual_machine_scale_set.go).  The first is where the data that Terraform sends to the provider is inspected and turned into requests to send to Azure to either Create, Read, Update, or Delete via the [Azure SDK for Go](https://github.com/hashicorp/terraform-provider-azurerm/tree/main/vendor/github.com/Azure/azure-sdk-for-go), and the second contains various helper functions.

Within [orchestrated_virtual_machine_scale_set_resource.go](https://github.com/hashicorp/terraform-provider-azurerm/blob/5fd32b3b3cf8a4a1891dee79e8890a125a2f36ce/internal/services/compute/orchestrated_virtual_machine_scale_set_resource.go) there's a function called `resourceOrchestratedVirtualMachineScaleSetCreate` on [line 207]((https://github.com/hashicorp/terraform-provider-azurerm/blob/5fd32b3b3cf8a4a1891dee79e8890a125a2f36ce/internal/services/compute/orchestrated_virtual_machine_scale_set_resource.go#L207)).  That seems like a sensible place to start looking.

---
### resourceOrchestratedVirtualMachineScaleSetCreate()

resourceOrchestratedVirtualMachineScaleSetCreate() takes two arguments, 'd' takes a pointer to a type of `pluginsdk.ResourceData`, while 'meta' will accept types of `interface{}`.  Essentially, d is where the contents from our Terraform config are fed into the function.  I'm not a huge fan of Go's preference for single letter variables as function arguments, but that's just me.

On [line 233](https://github.com/hashicorp/terraform-provider-azurerm/blob/5fd32b3b3cf8a4a1891dee79e8890a125a2f36ce/internal/services/compute/orchestrated_virtual_machine_scale_set_resource.go#L233), our provider starts building up a variable for our resource called `props`, this is an instance of type `compute.VirtualMachineScaleSet` which itself is a Go `struct`.  A struct is _sort_ of like a class in that it is a defined list of property names and value types, but seeing as Go is _not_ object orientated, there are no constructors or methods attached to a struct.  But in plain English, it's a blueprint for _'a thing'_.

![Props](/img/Props_01.png)

>#### _"Hmmm...those 'Location' and 'Tags' properties look familiar...they're in the JSON body..."_

Similarly, on [line 251](https://github.com/hashicorp/terraform-provider-azurerm/blob/5fd32b3b3cf8a4a1891dee79e8890a125a2f36ce/internal/services/compute/orchestrated_virtual_machine_scale_set_resource.go#L251) a variable called `virtualMachineProfile` is created.  This is based on the `compute.VirtualMachineScaleSetVMProfile` struct (blueprint).

![virtualMachineProfile](/img/VirtualMachineProfile.png)

>#### 💡 _Looking at the incorrect JSON the provider sent out [above](#inspecting-for-myself), the missing ' imageReference' property should be nested within virtualMachineProfile!_

So these variables, `props`, `virtualMachineProfile`, and more, are building up the JSON body we're going to submit to the Azure management API.  We're on the right path.

---
### So where is our imageReference?

It looks like the imageReference property should be nested within virtualMachineProfile\StorageProfile and on [line 306](https://github.com/hashicorp/terraform-provider-azurerm/blob/5fd32b3b3cf8a4a1891dee79e8890a125a2f36ce/internal/services/compute/orchestrated_virtual_machine_scale_set_resource.go#L306) we can see the provider setting this property to whatever is stored in the `sourceImageReference` variable.

In fact we can see below where the provider reads these properties from `d`, and passes them to a function for processing before setting the result on line 306...

![Setting Image Reference](/img/Image_Reference_01.png)

It looks like line 305 is a good candidate for a breakpoint!

---
### expandOrchestratedSourceImageReference()

With a breakpoint set on line 305, the call to the expandOrchestratedSourceImageReference function, we can see that the sourceImageId successfully populated in line 304: -

![sourceImageID is present](/img/Breakpoint_01.png)

OK, so let's _'Step Into'_ `expandOrchestratedSourceImageReference()` to see what the function is doing.

As we can see on line 305 two images up, the provider passes in `sourceImageReferenceRaw` as the first argument, and `sourceImageId` as the second.  As always, when jumping into functions, it's important to make a note of that.

---
### expandOrchestratedSourceImageReference()

This function lives within the file 'orchestrated_virtual_machine_scale_set.go' at [line 1338](https://github.com/hashicorp/terraform-provider-azurerm/blob/5fd32b3b3cf8a4a1891dee79e8890a125a2f36ce/internal/services/compute/orchestrated_virtual_machine_scale_set.go#L1338).  The arguments it takes are used internally as `referenceInput` and `imageId`.

So, `referenceInput` is what is passed through as `sourceImageReferenceRaw` and `imageId` maps to `sourceImageId`.

Something immediately jumped out to me at this point.  Looking at the first `if` statement, it will return `nil` if `referenceInput`(`sourceImageReferenceRaw`) is blank.

Now, given that you _have_ to provide _either_ a `source_image_reference` or a `source_image_id` in your Terraform config, it stands to reason that if you add a `source_image_id`, your `source_image_reference` value will always be blank.  So it seems likely that if you provide a `source_image_id`, the provider will never actually use it.

![Broken Function](/img/ExpandImageRef.png)

Continuing to use _Step Into_ confirms that line 1340 is reached and the function exits and returns `nil`.

Continuing a couple of lines further confirms that virtualMachineProfile.StorageProfile.ImageReference is indeed blank.  It should have been set in line 306 back in our ...resource.go file, as shown a few images up.

![Blank Image Reference](/img/Blank_Image_Reference.png)

---
### Comparing Behaviour with Other Resource Types

Given that these config parameters (virtualMachineProfile.StorageProfile) are common among other resource types, I thought I'd look to see how this functionality was implemented in them.

I chose the [azurerm_linux_virtual_machine_scale_set](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/linux_virtual_machine_scale_set) resource as it's more or less the same resource (it's a uniform orchestration VMSS as opposed to a flexible VMSS).  

The resource's create function is held in [linux_virtual_machine_scale_set_resource.go](https://github.com/hashicorp/terraform-provider-azurerm/blob/5fd32b3b3cf8a4a1891dee79e8890a125a2f36ce/internal/services/compute/linux_virtual_machine_scale_set_resource.go) and we can see a call to a similar, but differently named function, `expandSourceImageReference()`, on [line 566](https://github.com/hashicorp/terraform-provider-azurerm/blob/5fd32b3b3cf8a4a1891dee79e8890a125a2f36ce/internal/services/compute/linux_virtual_machine_scale_set_resource.go#L566).  This function is clearly written differently to expandOrchestratedSourceImageReference() though as it returns two things (we can see this from the assignment): -

`sourceImageReference, err := expandSourceImageReference(sourceImageReferenceRaw, sourceImageId)`

This function is held on line [401 of shared_schema.go](https://github.com/hashicorp/terraform-provider-azurerm/blob/5fd32b3b3cf8a4a1891dee79e8890a125a2f36ce/internal/services/compute/shared_schema.go#L401) and handles cases of blank `source_image_id` and `source_image_reference` much better, only returning an error if both are blank: -

![expandSourceImageReference](/img/ExpandSourceImageReference.png)

This feels like the correct way to populate the `sourceImageReference`.

---
### Testing a Fix

It seemed to me that the sensible thing to do would be to replace the faulty function call within _orchestrated_virtual_machine_scale_set_resource.go_ with a call to `expandSourceImageReference()` within _shared_schema.go_  and include the same error handling as we saw for the Linux VMSS resource (remember, the original function call only returned an ImageReference type, not an error as well).

I did this on commit [19cb84f of my fork](https://github.com/hashicorp/terraform-provider-azurerm/commit/19cb84f2f0c43fd26ee98fb65c7b27e9d1bb16a0) of the provider repo, recompiled the source code, started it back up in debug mode, and as if by magic, Terraform was able to populate the missing object for the JSON request body, and created my Flexible VMSS without a hitch!

---
### End Result

I added a detailed write-up of my findings on the GitHub issue [here](https://github.com/hashicorp/terraform-provider-azurerm/issues/14820#issuecomment-1197993877) and opened a Pull Request to try and get the fix live.  One of the maintainers at Hashicorp replied, ratifying the fix, and added it into a larger PR for VMSSs as [commit ed6a95d](https://github.com/hashicorp/terraform-provider-azurerm/commit/ed6a95d66237685a65adfe44e522b351eef96cb9), which also removes the redundant `expandOrchestratedSourceImageReference()` function.

---
### In Summary

Like I said at the outset, the amount of time an issue has been open doesn't necessarily correlate to its complexity, and in this instance the bug was pretty easy to track down.  So why not crack open the source code and get involved?  The issue had been open on GitHub for over half a year and all in all it probably took me an hour or so of debugging it to find root cause and create a fix.  In fact, putting this post together took a lot longer!  I'm sure not all issues will be this easy to resolve, but it's a great way to get to know how Terraform providers work and have some fun getting your hands dirty.