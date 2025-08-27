---
title: "Goodbye Heroku, Hello Appliku"
date: 2025-08-27T17:30:00+01:00
draft: true
description: "Migrating away from Heroku to Appliku"
keywords: ["Django", "Heroku", "Appliku"]
tags: ["Django"]
---

> After 5+ years of Heroku hosting, PigskinPredictor is moving to a new home with thanks to Appliku

### Why Move Away From a Tried and Tested Solution?

I've been a fairly happy Heroku customer for nearly 6 years but it's not without its challenges for a small project like mine - namely cost!  Prior to November 2022, Heroku had a free tier - which meant I was able to spin up test instances of my project from a feature branch and have others access and test things out as if it were live. That all went away, which was a massive shame.

On top of this, each worker process (in Heroku speak, "Dynos") carried individual monthly costs.  So by having a web server process _and_ a Celery worker, that meant twice the dyno costs - had I separated out Celery Beat (as is best practice), that would have incurred a third cost.  That's before you take into account costs for Redis and PostGres...so you can see why, for small projects, costs can quickly escalate.

What I will see in Heroku's defence, is that their PostGres plans were actually quite competitive and one of the reasons I stuck with them after the free tier was killed.

### Self Hosting is a Nightmare Though

As mentioned before, one of the benefits of a PaaS offering like Heroku is that it abstracts server management away from you, which is an absolute God send. But self-hosting is undeniably the more cost effective solution. So how do you reconcile these two things? Can you self-host but keep deployments simple and abstract away the management?

### Enter Appliku

[Appliku](https://appliku.com/) is something I discovered this year and tried out as a potential candidate to migrate away from Heroku with. Appliku is _not_ a hosting service. Instead it's a "bring your own server and we'll handle the rest...service".

Put simply, you rent a [VPS](https://en.wikipedia.org/wiki/Virtual_private_server), add it in to Appliku with an SSH key, and then use Appliku to manage deployments to the server.  I personally use [Hetzner](https://www.hetzner.com/), a German cloud company, with my 4 vCPU, 8GB RAM, 80GB Disk VPS costing < €8/month.

What this means in practice is that you can deploy your application (or applications) to your server, complete with database and as many individual services as you like and they all run in one neat Docker Compose environment.

In fact, there's a very easy-to-use Postgres Import panel so you can move your valuable database across in a matter of seconds.

Appliku also handles your TLS certificates for you, configures your clients' SSH keys on your server, provides server hardware metrics, allows for easy database backups...the list goes on.

One of the features I love is that you can define your application in YAML ([doc](https://appliku.com/guides/yml/)) - so all of your Docker services, your Cronjobs, your Database versioning, Env Vars can be defined and changes tracked in your project's source code - this means that when you deploy that test app from a feature branch, you can guarantee it'll be configured _exactly_ like production.

### Why I'm Sold on Appliku

On top of the above technical reasons, I've found the support through Discord to be fantastic - rarely having to wait more than a few minutes to get a response and solution to a query.

As a solo developer, I love that I can leverage Appliku to overcome those cost barriers I previously faced while keeping management effort to a bare minimum.  I now have Celery and Beat components running separately, but that's just the start.  If I want to spin up extra services I can do so at zero cost.  All the while _massively_ reducing latency as these services are all running alongside each other.  I can also now deploy test instances to the same server for no extra cost.

### I'm a Happy Bunny

There's a definite learning curve to both configuring your project and getting used to managing it slightly differently, but so far, I haven't seen any tangible downsides to be honest.

As a final note (and without wanting to get too political) it's nice to be able to support a smaller European project rather than fill the pockets of an American Goliath.