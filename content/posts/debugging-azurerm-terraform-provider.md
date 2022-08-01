---
title: "Debugging the Azurerm Terraform Provider"
date: 2022-08-01T19:07:56+01:00
draft: true
---

> Debugging can be hard, but often the hardest part is getting your debugging environment set up right to begin with!

It took me a while to get my environment where it needed to be to debug the AzureRM Terraform provider, so in this post I'll run through all of the steps needed and save you the hassle of piecing together lots of random VSCode/Go/Terraform debugging articles that I went through to get to the end goal.

### Step One: Go

Terraform, and Terraform providers, are written in Go.  So the first thing you'll need is at least a rudimentary understanding of the language.  Assuming that is in place, you'll need to install Go on your system.  I won't go into detail on how to do that, but it's pretty straightforward on any system.