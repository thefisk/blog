---
title: "Taking Back My Inbox in 2025"
date: 2025-10-29T17:30:00Z
draft: false
description: "How I reorganised, resecured, and took back some email privacy in 2025"
keywords: ["email", "privacy", "proton", "protonmail"]
tags: ["email"]
---

_I've been trying to take back my relience on Google this year and, as a (very) longstanding Gmail user have finally found a new home for my inbox, along with a new strategy for managing email_

---
### The Problems That Needed Fixing

I've been using Gmail for my primary email literally since the day it launched. In fact, I managed to grab my (at the time) preferrd alias @gmail.com through a beta invite in 2004. This has been a blessing and a curse. Of course, having all of my emails from the past 20+ years in one place is super handy, and having an email address that everyone I want to know can easily remember is lovely. But the trade-offs are _massive_ when it comes to **privacy**.

When I say _privacy_, I mean that in two ways.

Firstly, I don't love giving Google access to all that info about me - that's probably the most obvious privacy concern. But the other stems from inevitable hacks and breaches that put some of your details out in the public. Having a single email address registered for every online service, shop, and mailing list makes profiling so much easier, as well as exposes you to guessable logins.

---
### The Solution I Landed On

I now use a paid tier of [Proton](https://proton.me/) for pretty much all of my email. I like the privacy elements when it comes to things like their encryption, removal of trackers, and prompts to remove metadata from attachments etc. But that's only half the story - the real value for me comes from the flexibility that comes with [SimpleLogin](https://simplelogin.io/) and being able to separate out your Proton login from your frequently used email addresses.

---
### Login and Sending Mails

My account with Proton is an address that I never use to send or receive emails - so that means it's never out in the wild for anyone to try and log in to. I use 2FA and physical security keys as well, but those are extra levels of security. I like the peace of mind knowing that my actual logon id is something only I know.

For sending emails to actual people, I use a personal domain which is configured with Proton's MX settings. So that bypasses SimpleLogin and is just managed within Proton. However, I _don't_ use this address to sign up to anything.

---
### Logins and Mailing Lists etc

Now, with SimpleLogin, I use a different personal domain and in fact use 3 subdomains each for a different purpose: -

- One is for my personal things that I care about;
- One is for home things which I share with my wife (think utility bills etc);
- One is for temporary things I probably don't want to keep

Then, within Proton, I have labels applied to each when the recipient matches each subdomain.

Each of these is configured within SimpleLogin to allow the creation of on-the-fly aliases and, for the home subdomain, I make use of SimpleLogin's ability to send aliased emails to multiple mailboxes - one being one of my Proton addresses, and the other being my wife's email address.

This means I can just register for a site with a completely made up email address using my registered subdomain that we share and we'll both get the emails in our true inboxes. Think _mybank@sharedemails.mydomain.com_

---
### Going a Step Further - Suffixes

As an additional piece of security by obscurity, I add a pseudo-random 6-digit suffix to each email address I register using my aliases. So rather than _spotify@myemails.mydomain.com_ I will use something like _spotify-3jas8x@myemails.mydomain.com_

The benefit this gives me is that, if there is a leak at Spotify, in the above example, attackers couldn't infer my other addresses by simply substituting "spotify" for "netflix", or whatever the case may be.

---
### Additional Benefits

It's long been acknowledged that you should use a different password for every site you register for, but by using a different email you also get to take control of your inbox in a much more controlled manner. If an email is leaked and suddenly I find I am receiving a load of spam to my _spotify-3jas8x@myemails.mydomain.com_ address, as well as knowing who sold my email/was hacked, I can kill the spam at the source and disable the alias at the click of a button.

---
### Summary

The process of swapping out all of my registered emails (well, most...is it ever _truly_ done?) took several weeks and was very tedious but the benefits have definitely been worth it for me. I feel much more organised and am very happy to have stoped giving Google so much of my personal data. But overall I just feel a lot happier with how I use email now - the shared subdomain is super useful and makes managing household things so much easier (especially as the logins are stored in a Proton Pass vault which I share with my wife).

So yeah, it's taken a while to land on a setup I'm happy with but I feel like the new setup will hopefully serve me well for the next 20+ years, like Gmail did...before it didn't.