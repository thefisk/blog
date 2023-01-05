---
title: "Secrets in a Post Soft Delete World"
date: 2023-01-05T07:07:51Z
draft: false
description: "Looking at Key Vault Secrets deployed via Terraform now that Soft Delete is being enabled on all Key Vaults"
keywords: ["Terraform", "Azure", "AzureRM", "Key Vaults", "PowerShell"]
tags: ["Terraform", "Azure"]
---

_Here, I'll look at some potential issues Azure Terraform users might find themselves in when deploying Key Vault secrets via Terraform with the advent of enforced Soft Delete and ways to manage this_

### Intro

In July last year, Microsoft [announced](https://learn.microsoft.com/en-us/azure/key-vault/general/soft-delete-change) that Soft Delete would be enabled on all Key Vaults going forward.  This is a positive move that protects consumers and their data, however, when it comes to idempotency it is not without issue.

---
### Terraform's Default Behaviour

By default, Terraform (by which I mean the AzureRM provider) will reinstate a Key Vault it is asked to deploy if it is found to have been soft deleted.  This makes sense as it follows the thought process that, "Well, I used to have a Key Vault with _this name_ in _this resource group_ in _this subscription_...and that one appears to have been deleted before, so the best thing to do is to recreate it."

##### So far, so good

---
### The Pitfall

With this autorecovery by default comes a potential issue.  When a Key Vault is recovered from Soft Delete, its secrets are also restored.  This means that in the below example configuration, `azurerm_key_vault_secret.my_secret` would fail to deploy because it already exists and this error could potentially break your deployment pipelines.

{{< gist thefisk 9fd745ef5f2a7325e502bab6f5b98cc2 >}}

---
### Terraform Ways Around This...
##### ...and why they might not be the best thing to do

Terraform, again AzureRM, provides us with a few handy options to customise this behaviour to suit our needs.

1. Firstly, you can set the `purge_soft_delete_on_destroy` flag in a provider features block.
   * This will purge the Key Vault altogether upon a Terraform destroy, however this is quite heavy handed and may well go against your data protection strategy.

2. Conversely, you can set the `recover_soft_deleted_key_vaults` flag in a features block.
   * As mentioned above, this behaviour defaults to true but it can be explictly set to false.  Again, this may not be the best thing to do if the Key Vault contains valuable secrets that you wish to recover.

3. Thirdly, you can set `purge_soft_deleted_secrets_on_destroy` to true, again in a features block.
   * Similar to the first option, this seems excessive and if you're keeping the Key Vault in Soft Delete but purging the secrets, there's really little value in this (unless you have keys and/or certificats in there)

4. Finally, and following the above pattern, you can set `recover_soft_deleted_secrets` to false.
   * This defaults to true, but again here you're losing the benefits of Soft Delete.

---
### Real World Issue

Obviously the hope is that if any Key Vaults were accidentally deleted in your subscriptions, you could just go in to the portal and recover them through the remarkably easy recovery process.  However, consider the below...

The gold standard of a well managed cloud infrastructure is the ability to redeploy all of your infrastructure at the drop of hat - from your codebase, without issue.  That's one of the main reasons for using Terraform, after all.  One way to add confidence in your ability to do so is through automated testing - spin up an environment on a schedule, tear it down afterwards, and confirm there are no errors.

Looking at an environment which has an `azurerm_key_vault_secret` as part of its codebase, the above automation would fail with default settings causing your `terraform apply` to error.  And in reality, the options to purge secrets/key vaults and/or not recover either aren't viable for production environments where those could be the only reliable source of those secrets.

---
### Ways Around These Shortcomings

One quick and dirty way to fix this would be to add a step to your automation to purge either the secret or the Key Vault after running `terraform destroy` in your testing pipeline.  That will get the automated spin-up & tear-down to work, but the trade off is accepting a fix-forward approach in a real-life recovery situation.

Another way would be to replace the `azurerm_key_vault_secret` with a `null_resource` using a `local-exec` provisioner to run a script which firstly checks for the presence of the secret in question, and then adds it if it doesn't exist.  Of course, you'd have to parameterise the script as we're talking about reusable code.  This might be the most sensible approach at the moment and means the above automation would continue to work, while a greenfield deployment would also work in spite of Soft Delete restoring the secret.  However, the trade off is moving away from simple to read and reuse Terraform, to adding complexity.  Not ideal.

---
### In an Ideal World

Personally, I think with the advent of Soft Delete on all Key Vaults, the AzureRM provider should have an option to import on discovery.  If it wants to deploy a `azurerm_key_vault_secret` resource but a matching key:value pair exists already, rather than return an error it would be nice if it gave you the option to import the secret.  Maybe I'll raise an issue on the provider's GitHub page, or even start looking into it myself.

---