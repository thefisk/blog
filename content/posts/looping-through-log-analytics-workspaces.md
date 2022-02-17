---
title: "Looping Through Log Analytics Workspaces"
date: 2022-01-19T09:48:30Z
draft: false
tags: ["Azure", "Scripting"]
---
> Using the AZ Monitor Log-Analytics Query command to script queries

### Log Analytics Workspaces and Scripting

Log Analytics Workspaces are great in Azure - they provide a nice repository for your log data which, when combind with [KQL](https://docs.microsoft.com/en-us/sharepoint/dev/general-development/keyword-query-language-kql-syntax-reference), make finding the log data from your valuable Azure resources a cinch.

Often in the world of Azure though, it useful to script things to make your life easier for large and/or repeatable tasks.  The Azure CLI provides a wealth of tools to let you do this but sometimes it's not very intuitive to know what to put as certain arguments, or even how to attain those values.

### The Problem

That was the case when I recently wanted to loop through various Log Analytics Workspaces.  I wanted to use the [az monitor log-analytics query](https://docs.microsoft.com/en-us/cli/azure/monitor/log-analytics?view=azure-cli-latest#az-monitor-log-analytics-query) command but this requires knowing the GUID of every single LAW.  In a large enterprise environment you will likely have naming standards for things like your LAWs and your Resource Groups that you can either provide in a list or enumerate with string interpolation.  But doing so for a GUID is obviously a different kettle of fish.

In the image below, the argument we need to pass in for --workspace is "982a8281-cccd-49bb-aae9-af0a7b7c4806".  But how do we find that, programmatically?

![Log Analytics Workspace Properties](/img/law_properties.png)

### The Solution

Checking the JSON View provides some clues; confusingly it is the 'customerId' of the resource, as shown below: -

{{< gist thefisk 263a73df240109a16e5128161d950508 >}}

With this information, we can find the Workspace ID/customerId by using [az monitor log-analytics workspace show](https://docs.microsoft.com/en-us/cli/azure/monitor/log-analytics/workspace?view=azure-cli-latest#az-monitor-log-analytics-workspace-show) (assuming we have firstly set the active subscription, know the resource group name, and know the LAW name).  This is displayed below on lines 5 and 8, where we get the properties of the Workspace and extract the ID.  Piping to convertfrom-json simply takes the JSON returned data and converts it to a PowerShell object.

{{< gist thefisk a7c269080fe0149ec6bb9b3222453b0b >}}

From there, you can run whatever KQL query/queries you wish.  Again on line 11 I'm piping to convertfrom-json to store the results as a PowerShell object.  On line 14 I add a new property to help me distinguish which subscription each set of results relate to.