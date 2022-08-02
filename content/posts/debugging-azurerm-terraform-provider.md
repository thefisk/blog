---
title: "Debugging the Azurerm Terraform Provider"
date: 2022-08-01T19:07:56+01:00
draft: false
tags: ["Terraform", "Azure", "Go"]
---

> Debugging can be hard, but often the hardest part is getting your debugging environment set up right to begin with!

It took me a while to get my environment where it needed to be to debug the AzureRM Terraform provider, so in this post I'll run through all of the steps needed and save you the hassle of piecing together lots of random VSCode/Go/Terraform debugging articles that I went through to get to the end goal.  I use VSCode for everything so this guide is specifically written with that environment in mind.

### Step One: Go

Terraform, and Terraform providers, are written in Go.  So the first thing you'll need is at least a rudimentary understanding of the language.  Assuming that is in place, you'll need to install Go on your system.  I won't go into detail on how to do that, but it's pretty straightforward on any system.

---
### Step Two: Delve

[Delve](https://github.com/go-delve/delve) is a debugger for Go.  We'll be using this to run the provider in a special way to allow our IDE to piggy-back on to the running process so we can set breakpoints and inspect variables.  With a recent version of Go installed (>= 1.1.6), installing Delve is a simple case of running the below.

`go install github.com/go-delve/delve/cmd/dlv@latest`

---
### Step Three: Compile the Provider

Firstly, we'll need to clone the [AzureRM repo](https://github.com/hashicorp/terraform-provider-azurerm/) locally (kind of a pre-req for any debugging).  Optionally, you can set the flag 'debuggable' to default to _true_ by changing the value on [line 18](https://github.com/hashicorp/terraform-provider-azurerm/blob/da8cd028f437022963fb009478a9ebbd3ec06e3a/main.go#L18) of main.go.

Once that is done, it will need to be compiled with some special flags.  To do so, make sure you're in the root directory of the source code and run ther below command.

`go build -gcflags="all=-N -l"`

-N will disable optimisations, while -l (lower case L) will disable inlining (the compile-time process of moving function code to the calling function).  Essentially these options will ensure that the compiled code is unaltered and matches the source code.  This is crucial for debugging.

---
### Step Four: Create a Debug Config

Debugging can be done in numerous ways.  With compiled code though, we can either launch a process or attach to a running process.  As we'll be using Delve to initialise the process, we'll want to do the latter. Within VSCode you'll need to create a config file which allows for this to happen.  There are a myriad of options you can add to your launch.json file, but I have found the below to work best.

{{< gist thefisk a3d9fddbc05daa80a399a6069450429d >}}

By specifying a target TCP port, we can just run Delve (in step 6) with the same port as a param and know they should connect fine.  I had some issues with apiVersion 1, so I'm using v2 here.  showLog and trace just adds some output on the debug process to the Debug Console within VSCode.

remotePath needs to be path to the executable.  Because my VSCode workspace is open at the root of the GitHub repository, and that's where the compiled binary is, `${workspaceFolder}` works great.

---
### Step Five: Ensure Env Vars are Present

Go installations seem notorious for missing some env vars which Delve will rely on.  To ensure these are present, check and set these with the below (commands for Linux).

`export GOPATH=$HOME/go`\
`export PATH=$PATH:$GOPATH/bin`

Obviously in the case of the first one, this should be the location of your Go installation.

---
### Step Six: Launch the Debugger

With all of the above in place, we should be ready to do some debugging.  The first thing to do is run Delve in headless mode, remembering to have it listen on TCP port 36283 to match our VSCode attachment target.  To do this, run the below from the root of the project.

`dlv exec --listen=127.0.0.1:36283 --api-version=2 --headless ./terraform-provider-azurerm -- -debuggable`

The debuggable flag matches that on line 18 that we saw in step three, so if you compiled the provider with this set to true, you should be able to omit this.  Either way, specifying it won't hurt.

That should produce an output similar to below

`API server listening at: 127.0.0.1:36283`

Next, within the Run and Debug pane in VSCode, click the play button on our Connect to Server config

We should see some output like the below

`{"@level":"debug","@message":"plugin address","@timestamp":"2022-08-02T08:40:56.630654+01:00","address":"/tmp/plugin839766935","network":"unix"}
Provider started. To attach Terraform CLI, set the TF_REATTACH_PROVIDERS environment variable with the following:`\
`TF_REATTACH_PROVIDERS='{"registry.terraform.io/hashicorp/azurerm":{"Protocol":"grpc","ProtocolVersion":5,"Pid":8869,"Test":true,"Addr":{"Network":"unix","String":"/tmp/plugin839766935"}}}'`

---
### Step Seven: Tell Terraform to use Local Provider

Now that we have our own compiled provider running locally with a debugger attached, we need to tell Terraform to use it, as opposed to the standard Registry version.

This is simply a case of setting the TF_REATTACH_PROVIDERS env var that Delve helpfully output for us above.  Copy the _entire line_ including the final apostrophe/single quote and place after an `export` command (for Linux) and you should be good to go.

When you next run a Terraform init and plan/apply, Terraform will use your local provider and you'll see a nice wall of output flow through the console in which Delve is running.

You're free to set breakpoints as you see fit and have a good old inspect of what Terraform is doing behind the scenes.

---
### Have Fun

That's it really.  I wanted to create this guide because to get to this point I had to piece together information from all across the internet and somehow pick out the relevant bits.  There was a lot of trial and error involved.  Hopefully this guide will help someone out there.

---