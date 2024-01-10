---
title: "Architecting (Nearly) Real Time Updates in a Django based NFL Prediction Game"
date: 2024-01-08T12:21:49Z
draft: false
description: "How I set up (near) real-time score updates on my weekly NFL prediction game"
keywords: ["Django", "NFL", "VueJS", "Vue", "Websockets", "Heroku"]
tags: ["Django"]
---

 _As another NFL season enters the playoff stages and my beloved Chicago Bears once again leave me questioning my life choices, I thought it would be a good time to write a post about how I created a (nearly) live score page for players of a weekly NFL prediction game that I play and, a few years ago, turned into an automated website_

---
### The Preamble

I started playing a weekly prediction game a number of years ago that a friend introduced me to.  It was run by his cousin's colleague and involved emailing in who you thought would win each NFL game on a weekly basis.  Choosing a road team to be your "banker" each week which is where the jeopardy comes in; banker teams can only be chosen once per season and will score you double points if they win, but lose you a ton of points if they don't.  There's also now a 'Joker' feature too to add extra risk/reward!

A bunch of people would email their picks to the admin and he'd record them on a spreadsheet, which must have taken an immeasurable amount of time each week.  So this is why I set up [pigskinpredictor.com](https://pigskinpredictor.com) - to make the process quicker for players and admins alike, with the added benefit of reducing human error.

---
### The Goal

The site isn't anything _super_ complex.  Users register, make their weekly picks, some validations take place on both the client and server sides before those picks are saved, and there are a bunch of [celery](https://docs.celeryq.dev/en/stable/getting-started/introduction.html) tasks which do things like retrieve the results from an ESPN API and kick off our scoring system.  There are fancy scoretables (in VueJS) which can be sorted based on various statistics and those update on a weekly basis.  However, the site wasn't used massively outside of the weekly picks and maybe checking your leaderboard position.

So I wanted to try and add an extra dimension to the site.  Something to make players engage more with the game and add some spice to their Sunday evening watching rituals.  A live Sunday leaderboard, that changed with every touchdown, field goal, PAT, or safety would be the perfect thing.  Seeing how you're comparing with other players during this packed window of games would add that extra dose of excitement.  But how to go about achieving this?

---
### Cost Challenges

For something truly live, I'd need to consider using a pub/sub (publish/subscribe) pattern, most likely over WebSockets.  Now, the site is written in Django and deployed to Heroku.  Heroku has many many benefits over something like a VPS.  Being a PaaS offering, I don't need to worry about keeping my server up to date or scheduling downtime etc.  That's all abstracted away from me, which is how I like it.  Interfacing with PostGres and Redis is ridiculously easy as well.  So for me, running a game for 50-100 people, it does exactly what I need.  However, convenience like this comes at a price.

To run the a pub/sub using [Django Channels](https://channels.readthedocs.io/en/latest/), the Django package that implements WebSockets, would need a dedicated Redis instance and its own interface server running something like [Daphne](https://pypi.org/project/daphne/).  Those extra instances would cost $3 and $7/month respectively, which is roughly £95/year.  Given that we pass running costs onto the players, making sure there's enough left for a healthy prize pot, that's not the kind of extra cost I can bare.

So a _not so live_ option had to be sought.

---
### How to Architect the API Calls and Live Scoring

Knowing I wasn't going to be able to use a pub/sub pattern for pushing live scores, I was left with having to poll for updates periodically.  But this gave me a few different options.

For retrieiving scores, I could: -

- Have the back-end poll the ESPN API for score updates and store them in a Django model for the front-end to poll; or
- Have the front-end poll the ESPN API itself

While for calculating scoring, if I opted for the latter I would _have_ to place the scoring functionality on the front-end; whereas if I went for the first option, I could choose between calculating scoring on the backend or the front-end.

In the end I opted to have the backend poll ESPN periodically.  I figured it was safer to have just one entity poll the free API rather than start to flood it with calls from numerous clients every so often.  The front-end would then poll the backend every so often for updates to the scores.

In terms of calculating each players' points, I opted to do this on the front-end.  In reality, it could take place at either side, but doing so meant I saved space in my PostGres database and reduced the size of the JSON payload when the client polls the server for game score updates.

---
### Finer details

So, how does it all work?

Well, there's a lot of automation.  As I mentioned earlier, I use Celery for a lot of task automation.  For this particular piece of functionality, it's heavily used.  Firstly, on a Saturday lunchtime, it removes last week's games from the `LiveGame` table, then it reads all the games from the Games table which will be played the _next day_ and have a kick-off time earlier than 23:00.  It then places them in the aforementioned table which will hold the lives scores - [here\'s the Django model](https://github.com/thefisk/pigskinpredictor/blob/467f91a6c6b075149c569b2b80adfe418e575960/predictor/models.py#L294).  It features a `State` field so the front end can display completed, in-progress, and upcoming games differently, and an `Updated` boolean field which is used by the front-end to display a nice little flashing animation when a particular game's score changes.

During the Sunday games, [this Celery task](https://github.com/thefisk/pigskinpredictor/blob/467f91a6c6b075149c569b2b80adfe418e575960/predictor/tasks.py#L239) runs every 60 seconds which grabs the latest scores and updates the `LiveGame` entries.

On the front-end, the live scores page is initially fed all of the users' predictions as JSON.  Because getting this data is computationally expensive (slow), the first time someone hits the page, the lovely convenient serialised JSON is actually stored in my main Redis cache to speed up things for everyone.  Subsequent requests then just read from the cache.  The backend also sends through a `jsonuser` object representing the logged in user so that the front-end knows which user to highlight in the table.

For fetching scores from the backend, I use [Axios](https://www.npmjs.com/package/axios) as a client to poll my own API every 60 seconds (which reads from the corresponding table mentioned above).  That client is used as part of a method within my Vue instance.  It firstly [gets the latest scores](https://github.com/thefisk/pigskinpredictor/blob/467f91a6c6b075149c569b2b80adfe418e575960/predictor/templates/predictor/live-scores.html#L209), then iterates over all predictions and [scores them](https://github.com/thefisk/pigskinpredictor/blob/467f91a6c6b075149c569b2b80adfe418e575960/predictor/templates/predictor/live-scores.html#L138C7-L138C7), then updates each user's total, checks the live games states and [reorders if needed](https://github.com/thefisk/pigskinpredictor/blob/467f91a6c6b075149c569b2b80adfe418e575960/predictor/templates/predictor/live-scores.html#L196) (if a new game has kicked off/old one finished etc), and then finally [sorts the score table](https://github.com/thefisk/pigskinpredictor/blob/467f91a6c6b075149c569b2b80adfe418e575960/predictor/templates/predictor/live-scores.html#L200).  Any games with an updated score, get a [brief flash](https://github.com/thefisk/pigskinpredictor/blob/467f91a6c6b075149c569b2b80adfe418e575960/predictor/templates/predictor/live-scores.html#L75) via a CSS class.

---
### (Very) Rough Flow Diagram

![Sunday Live Component Diagram](/img/SundayLive.png)

---
### Conclusion

So, while I wasn't able to implement truly live scores without incurring great expense, I managed to implement a feature that is pretty close to live without incurring extra costs.  By using the back-end as a single caller to the ESPN API I limited the impact my userbase has there and, by using Redis to cache the (quite large) initial JSON payloads, I could speed up the delivery of the page quite significantly.

The end result is fully automated and pleasantly slick, thanks to Vue's nice out of the box animations - when the table updates, users seemlessly slide into their new positions.

It was a good learning experience and something I've been meaning to write up for a long time now.  Hopefully the players all enjoy it as much as I enjoyed making it.