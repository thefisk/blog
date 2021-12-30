---
title: "Creating a Hugo blog with S3 and Cloudflare"
date: 2021-12-30T14:31:46Z
draft: true
tags: ["Hugo"]
---
> How I made this blog and why...

When creating this blog, I came across many conflicting articles, particularly related to how to set up CI/CD for publishing updates to S3 and how best to use a CDN. I wanted to use this post to jot down what I ended up doing, and why.

I won't delve into why I chose Hugo in too much detail but if you've found this page you're likely considering it too and, as of December 2021, it's proved a good choice for me for managing a simple static blog. If you're familiar with Markdown and things like templating in Jekyll, Django etc, the learning curve isn't too steep.

### S3 + CDN

I'm personally not a huge fan of Amazon, but S3 offers ridiculously cheap storage which you can programmatically deploy to quite easily. A lot of guides I had read mentioned using CloudFront (AWS CDN) with S3 and that configuring it can take some work and incur some extra costs. However, I went with Cloudflare because they offer a very generous free tier that is perfect for small volume sites like this. It's easy to set up and just requires some domain ownership verification and adding the Cloudflare NS servers in your registrar's portal to let Cloudflare manage your DNS.

### DNS

Now, I wanted to host this site at _just_ nathanfisk.co.uk, i.e. without a preceding 'www' (it's not 1998, people). One thing to note is that when hosting a static site on S3 your bucket name needs to match your site's URL. I had read some blog posts that mentioned doing all sorts of things, like having two identical buckets (one with www. at the beginning), setting up crazy forwarders because you can't set a CNAME record at the root domain...

Luckily, none of this madness was needed because with Cloudflare, all I had to do was add a CNAME record with an @ symbol as the name and the S3 bucket website endpoint as the target and **bingo!** my DNS resolved perfectly.

### Automated Deployment

This blog is hosted on a public Git repository on GitHub and as such, I have included a YAML file that will run a Hugo build process when an update is pushed to the main branch, and then deploy the resulting static files to my S3 bucket.