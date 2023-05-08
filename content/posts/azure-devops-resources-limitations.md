---
title: "Azure Devops Resources Limitations"
date: 2023-05-08T16:33:18+01:00
draft: false
description: "Exploring the undocumented limitations on Azure DevOps pipeline resources"
keywords: ["Azure DevOps", "DevOps", "ADO", "AzDo", "Pipeline", "Service Connections", "Checks"]
tags: ["Azure", "DevOps"]
---

_The documentation around Azure DevOps can be somewhat lacking, particularly if you find yourself debugging an issue in production. In this post I'll explain one of the undocumented limitations that took several months for Microsoft to get to the bottom of and caused me a major headache in the process._

### Setting the Scene

Azure DevOps shouldn't need a lengthy introduction at this point in time, so suffice it to say that it's a popular SaaS solution for CI/CD that lets you break deployment pipelines down into stages, jobs, and steps.  How you organise and run your pipelines is up to you - some basic concepts are listed [here](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/key-pipelines-concepts?view=azure-devops).

At my place of work, as a platform team, we deploy platform resources into Azure spoke subscriptions using Azure DevOps.  These deployment pipelines are separated into individual stages based on their _environment_ - dev, test, production.  This allows us to deploy our resources to dev environments without affecting anything higher up the food chain, for example.

On top of this, we make use of [branch control checks](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/approvals?view=azure-devops&tabs=check-pass#branch-control) to ensure we can only deploy from our main branch.  This ensures relevant PR checks have all been completed and that no "rogue code" can be deployed.

A while ago though, we experienced a strange bug in Azure DevOps.  When trying to deploy to our dev spokes (from our main branch) we were greeted with the below error: -

`##[error]The job is using protected resource(s) for which checks have not been evaluated endpoint: [service endpoint name(s)]. For more details, refer to https://aka.ms/pipelinechecks.`

The DevOps GUI seemed to contradict this though, saying checks had passed.  The only way to resolve was to temporarily remove the checks.  So, what gives?

---
### Pipeline Resources

To understand the cause of this, we need to understand what a "resource" is in the world of Azure DevOps.  According to the [MS Docs](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/about-resources?view=azure-devops&tabs=yaml): -

> A resource is anything used by a pipeline that lives outside the pipeline.

There's a handy table [here](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/about-resources?view=azure-devops&tabs=yaml#use-resources-to-enhance-security), but essentially it's things like service endpoints, DevOps environments, repos, agent pools, queues, secure files, variables...

But why is this important?

---
### Root Cause

The below snippet is from someone on the Azure DevOps product group that helps explain what happens each time we run a _stage_, better than I could (emphasis added): -

> Whenever we try to run the stage, there's a pre-evaluation step that we must go through in order to determine whether we should run the stage at all. Part of that step is to evaluate the conditions our customers setup in their YAMLs (as a property of yaml stage snippet) and the other part is the evaluation of the checks. When we evaluate checks, __*we pick up all the resources used in that stage*__ (endpoints, environments, repos, pools, queues, secure files, variables) and take all the checks that were configured for these resources and start running them.

This is done by collating the resources and passing them to a stored procedure as an argument named `@resources`, which is defined as `VARCHAR4000`.  This means the argument has a _hard character limit of 4000 characters_.

Crucially, the stored procedure is sent the resources in a format that includes both the GUID _and_ the name version of the resource.  The below is an example of a service endpoint as it would be seen in the sproc argument (the GUID and names are made up examples): -

`endpoint-8cb58ea4-5127-454c-951c-fb4a8843b4ab2 => endpoint1-name`

Essentially, in our case, we had reached a breaking point where a combination of our naming conventions, number of dev subscription service endpoints and DevOps environments had caused us to breach this undocumented limit.

---
### Overcoming This Limitation

Knowing the character limit, what it is comprised of, and that it is applied before each _stage_ is run are the crucial things to know to make sure you avoid hitting this limit or, if you do, to help resolve the issue it will undoubtedly cause.

Key things to consider are: -

1. Pipeline Structure
   - Try to avoid stages with too many jobs where possible
2. Service Connection Naming Convention
   - The entire name is passed in as an argument that contributes towards the 4000 character limit - try to be concise; &
3. Environment Usage Strategy
   - Do you need one "environment" per job?  Can a single environment per stage work better?  And again, use concise naming conventions.

---
### Final Thoughts

This particular Microsoft support case took far too long to resolve in my opinion (it was a number of months).  The error returned is far too generic and doesn't help end users _or_ Microsoft support staff; hopefully they will address this moving forward.  The fact there is zero documentation out there to help users diagnose this issue or even plan a strategy to build their pipelines around this is also very poor.

So, a frustrating episode but a good learning experience for sure.  Hopefully this post will help someone out there 😊