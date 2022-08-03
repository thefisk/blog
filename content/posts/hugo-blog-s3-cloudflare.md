---
title: "Creating a Hugo blog with S3 and Cloudflare"
date: 2021-12-30T14:31:46Z
draft: false
description: "How I created this blog with Hugo, S3, and Cloudflare"
keywords: ["Hugo", "Amazon S3", "AWS", "Blog", "Cloudflare", "DNS"]
tags: ["Hugo"]
---
> How I made this blog and why I made some of the choices I did...

When creating this blog, I came across many conflicting articles, particularly related to how to set up CI/CD for publishing updates to S3 and how best to use a CDN. I wanted to use this post to jot down what I ended up doing, and why.

I won't delve into why I chose Hugo in too much detail but if you've found this page you're likely considering it too and, as of December 2021, it's proved a good choice for me for managing a simple static blog. If you're familiar with Markdown and things like templating in Jekyll, Django etc, the learning curve isn't too steep.

---
### S3 + CDN

I'm personally not a huge fan of Amazon, but S3 offers ridiculously cheap storage which you can programmatically deploy to quite easily. A lot of guides I had read mentioned using CloudFront (AWS CDN) with S3 and that configuring it can take some work and incur some extra costs. However, I went with Cloudflare because they offer a very generous free tier that is perfect for small volume sites like this. It's easy to set up and just requires some domain ownership verification and adding the Cloudflare NS servers in your registrar's portal to let Cloudflare manage your DNS.

---
### DNS

Now, I wanted to host this site at _just_ nathanfisk.co.uk, i.e. without a preceding 'www' (it's not 1998, people). One thing to note is that when hosting a static site on S3 your bucket name needs to match your site's URL. I had read some blog posts that mentioned doing all sorts of things, like having two identical buckets (one with www. at the beginning), setting up crazy forwarders because you can't set a CNAME record at the root domain...

Luckily, none of this madness was needed because with Cloudflare, all I had to do was add a CNAME record with an @ symbol as the name and the S3 bucket website endpoint as the target and **bingo!** my DNS resolved perfectly.

---
### Automated Deployment

This blog is hosted on a public Git repository on GitHub and as such, I have included a YAML file that will run a Hugo build process when an update is pushed to the main branch, and then deploy the resulting static files to my S3 bucket. I did have a few minor issues getting this to work which I have added comments in the YAML file below to address. A big one though which showed up in a lot of Google searches was caused by having my selected theme as a submodule in Git. By default, the checkout process won't add submodules so a lot of the files simply weren't generated. This was fixed by adding the setting on line 19.

{{< gist thefisk 7432eb4d478d90a34127571692da8ae0 >}}

---
### TLS

To get TLS working when hosting on S3, you have to use Cloudflare's Flexible option, as shown below.  'Full' is not possible because you can't host a certificate on S3.  But as we're using Cloudflare as a CDN, the end user's session is always encrypted and, as for my very small static site, it honestly doesn't need end-to-end encryption between Cloudflare and S3.

![Cloudflare TLS Settings](/img/cloudflare_tls.png)

---
### Extra Bits

There are some extra things like making sure the S3 is publicly accessible and locking it down to the Cloudflare IP addresses, but all of that stuff is readily available at sources far more reputable than this.

---
### The Result

So, now it's all up and running, posting updates is as easy as creating a new markdown page and pushing up to GitHub. I can rest safe in the knowledge that the site should be nice and fast, thanks to Hugo and Cloudflare, and extremely easy on the wallet. :blush:

---