---
title: "Domino's Breach - Dishonesty or Incompetence?"
date: 2026-10-14T12:45:27+01:00
draft: false
description: "A review of a recent security disclosure of Domino's UK & Ireland and its accuracy"
keywords: ["security", "hack", "breach", "account", "domino's"]
tags: ["security", "news"]
---

_I received a rather interesting email this week from Domino's UK & Ireland about an account breach. But all is not as it seems_

## Intro

At 16:16 BST on Tuesday my phone pinged to say I had an email. Nothing unusual there. However, when I opened it my intrigue was piqued more than usual. Apparently my Domino's account credentials had found their way into the hands of nefarious actors. But don't worry, it's not actually a Domino's breach...or so I was told.

TL;DR - I don't believe Domino's. Read on to find why...

{{< details summary="Click for Email Screenshot" >}}
![Domino's Email](/img/dominos_email.png)
{{< /details >}}

___
## Quick Check-Up

So - why don't I believe them? Let's review my set-up...

### How Do I Manage My Logins?

I've previously detailed in this [blog post](../taking-back-my-inbox-2025) how I now manage my logins. But in summary, as well as using the long established best practice of a unique password for each site, I also use a unique email address (or alias).

One of the benefits of doing this is that it means I have a 1:1 relationship between login ID and service. This means that [credential stuffing attacks](https://community.owasp.org/attacks/Credential_stuffing) should be impossible.

On top of that, the source of breaches can be tracked even when they're not reported and that I can consequently disable that email alias if I need to - say it starts receiving all sorts of spam; I can just cut it off at the source.

### Where Are These Stored?

This is a critical factor - your logins are only as good as where they're stored. If you're going to have unique aliases and unique passwords, you're not going to be able to remember them - that's impossible. So a secure vault is a necessity.

Personally, I use Proton Pass as I'm a paying subscriber and it integrates well with browsers and mobile OSs, and I trust Proton's encryption and ecosystem more than I trust most.

An extra factor to this is that my Proton login is never used for anything other than logging into the account. It's not used to send or receive email (so attempted nefarious logins to my Proton account should be nil).

On top of that, I have 3 YubiKeys set up to access the account - and I haven't lost any of them 😉

All of that is to say, I don't believe it's likely that my Domino's credentials were stolen from my vault.


### Have I (actually) Been Pwned?

Let's do a quick check on [haveibeenpwned?](https://haveibeenpwned.com)

My Domino's login is `dominos-[random-6-digits]@subdomain.mycustomdomain.com` (it's not _this_ domain 😉) - and remember, this has only been used to log in to Domino's.

And my *previous* password is `ftD%hRA5r*uybEHq&6QS` - 20 random digits I have never used anywhere else.

> #### Sends cat to HaveIBeenPwned to check for me... {.fiskblockquotelight}

<img src="/img/cat_thinking.gif" alt="Thinking" class="memegif">

> #### Ah! As I suspected - no, neither of these show up on HaveIBeenPwned {.fiskblockquotelight}

---
## Upshot

So, given all of the above, I feel very confident in saying that my credentials for my Domino's login were not found in another site's breach.

Which raises the question - are Domino's being dishonest or just incompetent?

That might sound facetious, but I'm genuinely curious. If my account was accessed by someone other than me, then the only reasonable source of those details is Domino's themselves - i.e. they're being economical with the truth.

If Domino's itself _wasn't_ breached then my credentials almost certainly _haven't_ been accessed - and so the email was either sent in error, or __really__ badly worded* - i.e. they're incompetent.

So again, clearly something is wrong here, whether intentionally or not.

###### *It is possible that maybe they could have taken the stance that they would like everyone to change their password as a precaution - but that's not what the email says. {.smaller}


### The Broader Picture

I'm very comfortable that my own set up is secure and that I won't personally face any further issues as a result of, well, whatever did or didn't happen at Domino's. I'd encourage others to consider a similar strategy for managing logins.

But the bigger point to make is that companies really need to do better at handling and disclosing these things. There's something not quite right with this one, and that's not really good enough in 2026.