---
title: "Site Outage Post Mortem: October 2024"
date: 2024-10-08T08:14:57+01:00
draft: true
description: "Reflecting on a Badly Timed Site Outage"
keywords: ["Django", "Django-Redis", "Heroku", "Redis", "TLS", "Outage"]
tags: ["Django"]
---

> What caused a website that I run to have an outage at the worst possible time, how did I recover, and would could I done better?

### Setting the Scene

As I have previously blogged ([here](../nearly-real-time-updates-django-prediction-game) & [here](../django-caching-pattern-redis)), I run an NFL prediction game which is written in Python using the Django Web Framework with some elements of VueJS. I also use [Celery](https://docs.celeryq.dev/en/main/getting-started/introduction.html) in this stack for both asynchronous tasks and scheduled tasks - so things like sending confirmation emails and weekly jobs which pull in game results and score users' predictions respectively. The site is hosted on Heroku, a PaaS provider, which works well for my needs.

### Why Heroku?

As a "one man band", I want to spend my efforts concentrating on adding features and improving functionality of the site, for me that's really rewarding to know the fruits of my labour can be enjoyed in the real world by our player base. This means I don't want to have to spend time managing things like server patching, database maintenance, managing TLS certificate lifecycles etc. With Heroku, that's all abstracted away from me meaning that I can rest assured that my small project is in safe hands and won't go down if I forget to do something like patching a zeroday vulnerability.

### The Outage | An Intro

The above abstraction is great but there are still certain activities that need to be undertaken - the Heroku-20 stack (basesd on Ubuntu 20.04) is deprecated, for example, so I will need to upgrade to Heroku-24 (Ubuntu 24.04), which will undoubtedly bring about dependency challenges.

One such activity that recently needed action was as a result of a [change in the way Heroku Redis URLs worked](https://devcenter.heroku.com/changelog-items/2923). Heroku provides a managed Redis service which your app can plug in to by simply reading the environment variable `REDIS_URL`. This URL was previously a non TLS URL, but was being changed to a TLS URL.

You could argue that this change wasn't necessary because the Heroku workers are hosted in AWS, as are the managed Redis instances, so I can't imagine there's any public traffic between the two but I suppose it's feasible that something could read the in-flight data between the worker and Redis, depending on how the VPCs are configured.

### Lesson 1: Don't Get Complacent

The linked article from Heroku states that the URLs would change on 30th Sept 2024. When this came and went, without issue, I assumed there wouldn't be anything to mitigate. This logic seemed sound because when env vars are updated in Heroku (including, I assume, the Redis URL var), workers restart so the new variables can take affect. As I'd seen no issue on or shortly after 30th Sept, I assumed all was well. This obviously wasn't the case.

### The Outage | The Trigger

At 16:37 on Thursday 3rd October, Heroku Redis updated itself for my app. I'd never seen this kind of self-initiated update before. It's now on release 7.2.4 and was on 7.2.something before, so I can only assume it was a vital patch. Anyhow, this update _did_ trigger a restart of my workers which caused the website to break and display the below image to users