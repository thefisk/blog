---
title: "Django Caching Pattern for Redis"
date: 2022-10-12T07:59:34+01:00
draft: false
description: "Why I cache certain data from Django in Redis and the pattern I tend to use for consistent results"
keywords: ["Django", "Django Web Framework", "Caching", "Cache", "Redis"]
tags: ["Django", "Redis"]
---

> Caching data which doesn't often change can bring massive performance benefits - let's look at how we can do this effectively in Django

### Why Redis?

[Redis](https://redis.io/) is an open-source in-memory database/caching solution and is pretty much seen as the defacto product in this space nowadays.  Services like AWS and Azure offer native services that provide Redis for consumers in ElastiCache and Azure Cache respectively; Heroku has their own Redis service (which piggybacks the AWS offering) too.  So it's very widely available as a service and extremely well-documented.

As of Django v4.0, Redis is offered as a native cache backend as well, though prior to v4.0 you can just use the [Django-Redis](https://github.com/jazzband/django-redis) package.

All of the above is to say that Redis is tried, tested, and readily available for easy adoption within Django.

---
### Why Cache?

The primary purpose of caching data is to reduce expensive (in compute terms) queries to databases and speed up performance.

In terms of Django, there are multiple 'levels' of caching you can perform.

The largest component you can cache (outside of the entire site, which isn't great for a dynamic site) is an [entire View](https://docs.djangoproject.com/en/4.1/topics/cache/#the-per-view-cache).  This means whenever a View is requested, any context processing and template rendering is skipped and a cached response is returned.  This is great for static pages but not ideal for any components which return user-specific context.

One step down from View caching is [caching template fragments](https://docs.djangoproject.com/en/4.1/topics/cache/#template-fragment-caching).  I actually implement this for header and footer sections of sites because the content tends to be static.  Using these cached 'fragments' of a templated page allows the main portion to still be generated dynamically, but means no compute time is wasted on the generation of identical headers and footers.

The smallest thing you may wish to cache though are individual bits of data that are sent to your front-end as context and that's what I wanted to discuss in this post.

----
### Setting the Scene

In my [Pigskin Predictor](https://github.com/thefisk/pigskinpredictor) project I score users' NFL predictions based on the outcomes of the games.  This is done on a weekly basis where I retrieve scores from an ESPN API and store them in my own database table.

I offer a few different scoretables, one of which, the 'Enhanced' scoretable, shows a whole host of data - including each user's average weekly scores, their season's high and low scores, percentage of correct predictions...the list goes on.

This data is sourced from 3 tables (User, ScoreWeek, and ScoreSeason) and is enumerated by running a `for` loop on all entries in the ScoreSeason table (there is one entry per active user) - so it takes a little while for all of this data to be returned.

The data is sent to the front-end as a python dictionary, which the template turns into a JSON object with the [json-script filter](https://docs.djangoproject.com/en/4.1/ref/templates/builtins/#json-script), so it can be displayed in a VueJS component which allows users to sort on any columns they wish, amongst other things.

> ##### "Because this data has been pulled from the database and serialized, and it **doesn't change** for a week, that makes it a prime candidate for caching!" {#fiskquotestrong .fiskblockquotehighlighted}

---
### The Pattern

What I like to do when caching data to be used as context, is: -
- Firstly use cache.get() to try and retrieve the cached data;
  - This works great because cache.get() doesn't error if the cache doesn't exist, instead it returns `None` by default - [source](https://github.com/django/django/blob/004f985b918d5ea36fbed9b050459dd22edaf396/django/core/cache/backends/base.py#L142)
- If it doesn't exist, grab the data from the database and populate a list/dictionary and then;
- Save this to the above named cache
- Then whenever the relevant View is called, it will always check for a cached entry first and create one if it doesn't exist

This can all be seen in the below gist, which is a snippet taken from the example discussed above

{{< gist thefisk ed3febf9516ef133f60f09ace7664adf >}}

One other thing I like to do, is to clear all of my caches when the app restarts.  This is done by adding an AppConfig for the application in question and add a custom ready() function as shown [here](https://github.com/thefisk/pigskinpredictor/blob/cc21bcf50570e1e56037680b06a6b6758df75b2b/predictor/apps.py#L10)

---
### Summary

So, there we have it - a quick snippet on how and why I implement caching with Redis in Django.  It saves time when serving content and isn't super-complicated to implement.  It's all just a case of knowing what to cache and for how long.