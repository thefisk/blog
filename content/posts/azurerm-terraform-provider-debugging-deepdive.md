---
title: "Azurerm Terraform Provider Debugging Deepdive"
date: 2022-08-05T08:02:26+01:00
draft: true
description: "Debugging an issue in the AzureRM Terraform provider: a step-by-step guide"
keywords: ["Debugging", "Go", "Golang", "Terraform", "AzureRM"]
tags: ["Terraform", "Go", "Debugging", "Azure"]
---

_In this post I'll walk through how I found the root cause of an issue within the AzureRM Terraform provider that had been open on Github for 7 months and why the length of an open issue doesn't necessarily correlate to its complexity_

> For details on how to set up a debugging environment for the AzureRM Terraform Provider, check out my [last post](../debugging-azurerm-terraform-provider)

### Setting the Scene

At my place of work we deploy all of our Azure infrastructure through Terraform where possible; I don't think I'm breaking any trade secrets when I say that.  We regularly deploy virtual machines and virtual machine scale sets (VMSS) in this way.  In Azure though, there are two orchestration modes for VMSS deployments; Uniform and Flexible.  One of the main differences between the two is the type of VM instance that Azure deploys as part of the scale set.  In the case of Uniform orchestration, Azure deploys a VM resource of type `Microsoft.compute/virtualmachinescalesets/virtualmachines` whereas for Flexible, it deploys a VM of type `Microsoft.compute/virtualmachines`.  A big benefit of the latter is that this is the same type as is deployed for a standard VM and so any automation that works with normal VMs should work fine with Flexible VM instances.

In terms of Terraform deployments, you can use resource type [azurerm_linux_virtual_machine_scale_set](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/linux_virtual_machine_scale_set) (or equivalent Windows resource) for a traditional Unform deployment, or [azurerm_orchestrated_virtual_machine_scale_set](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/orchestrated_virtual_machine_scale_set) for a Flexible deployment.

However, when I looked at the viability of deploynig a Flexible VMSS I hit a blocker with the provider.  When the `source_image_id` param was set, an error was returned from Terraform

### In Summary

Like I said at the outset, the amount of time an issue has been open doesn't correlate to its complexity, so why not crack open the source code and get involved?