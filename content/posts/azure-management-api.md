---
title: "Azure Management Api | Part One"
date: 2022-03-22T16:59:08Z
draft: false
description: "How to create an Enterprise Application in readiness for use with the Azure Management API"
keywords: ["Azure", "API", "Enterprise Applications"]
tags: ["Azure"]
---

This guide will show you how to set up credentials for using the Azure Management API in order to make some basic read and update requests.  A follow up guide will run through using the API using Postman.

The very first thing to do is to create an App Registration, this will act as the 'user' which will perform the CRUD operations through the Management API.

> ##### Firstly, head to App Registrations in the portal and create a new registration as per below. {.fiskblockquotehighlighted}

![New Registration Screenshot](/img/API_01_New_Reg.png)

---
> ##### Next, give it a suitable name and choose a Tenant scope: - {.fiskblockquotehighlighted}

![Naming the Registration](/img/API_02_Reg_Name.png)

---
> ##### Once created, we'll need to generate a client secret.  Check the screenshot below to do this and give it a suitable name and expiration period. {.fiskblockquotehighlighted}

![New Secret](/img/API_03_Secrets.png)
![Naming New Secret](/img/API_04_New_Secret.png)

---
> ##### Once the secret is created, be sure to **copy the value and save it somewhere**. Once you leave this screen, the value is never displayed again. {#fiskquotestrong .fiskblockquotehighlighted}

![Secret Created](/img/API_05_Secret_Created.png)

---
> ##### You'll also need to take a note of the Application (client) ID from the overview screen; together with the secret value above, these two will be used as the credentials to request access tokens. {.fiskblockquotehighlighted}

![Client ID](/img/API_06_Client_ID.png)

---
> ##### The last stage in the portal is to give your App Registration some permissions. This can be done via the IAM section of most resources by adding a role assignment and you can be as broad or granular as you like (management group, subscription, resource group, individual resource...) {.fiskblockquotehighlighted}

> ##### In the screenshots below I'm giving my new App Registration Owner rights over my subscription. {.fiskblockquotehighlighted}

![IAM Screen](/img/API_07_IAM.png)
![Assigning Role to App Registration](/img/API_08_Role_Assignment.png)

---

At this point we have a working set of credentials that will have permissions to perform CRUD operations on our subscription.

In [part 2](../azure-management-api-2.md), we'll look at using these credentials with Postman as our API client.

_part 2 live as of 30/07/2022_

---