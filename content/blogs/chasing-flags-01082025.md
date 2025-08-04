---
title: ChasingFlags Challenges - 01/08/2025
slug: chasing-flags-01082025
description: Web Challenges Walkthrough
longDescription: A brief walkthrough on two interesting web challenges from ChasingFlags CTF held on 01/08/2025 
cardImage: "https://cy00p.github.io/cy00p.webp"
tags: ["express", "git", "python", "sqlite", "scripting", "jwt"]
readTime: 10
featured: true
timestamp: 2025-08-03T19:46:03+00:00
---

I'll try to keep the walkthroughs brief. That being said, in case of any questions, suggestions or improvements, ping me on my X account. Rightio then:

## Vibe Coder
### TLDR
So this was a website, running express in the backend (you could tell from the headers), and a simple login page.
Now if you fuzz a little, you would discover a `/dashboard` path, that obviously wouldn't allow you to access it without being logged in.
One way to go about it is to forge a jwt token cookie, and since jwt signing was disabled, you would login as `admin` with the following fields set in the user data field in the jwt: 
- `username` could be anything i.e. `cat`
- `isAdmin` had to be set to `true`



## Captcha The Flag
#### TLDR
In this challenge, there were three challenges we had to overcome.
1. First, you had to identify a `/debug` path that would allow you to set an ip address such that you can login without solving the captcha.
2. Discover the login page was vulnerable to blind sql injection, but was being protected by a web application firewall
3. Now the hard part, crafting queries that would bypass the WAF and also leak the database



These were great challenges, huge shoutout to [ChasingFlags](https://linkedin.com/company/chasingflags/) and the challenge creators.
There was a ton to learn, and I can't wait to see what they set up in the upcoming challenges.