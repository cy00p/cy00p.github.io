---
title: ChasingFlags "Can You Hack it?" Round 1
slug: chasing-flags-01082025
description: Web Challenges Walkthrough
longDescription: A brief walkthrough on two interesting web challenges from ChasingFlags "Can You Hack It?" Round 1 CTF challenge held on 01/08/2025 
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

### Methodology
We are provided with a link, which upon opening, we have this login page:
![landing_page](../../src/assets/01082025/1.png)

If you try to login with any set of credentials, we get an invalid credentials error:
![invalid_creds_error](../../src/assets/01082025/2.png)

When you look at the source of the page, we just have a very simple html file, which loads a generic css file. At this point, it’s good we try to understand the stack running this website. One way to achieve this is to have a look at the raw requests, and see if there’s any unique header. Fortunately, there's one: `X-Powered-By`
![x_powered_by_header](../../src/assets/01082025/3.png)

Now we know we’re running `Express`, which is a JS middleware, and we can now try to do further enumeration. I’ll do a directory fuzz. Something worth noting is that if we try to hit an invalid endpoint, we get an error response that prints out the characters that we provided as the path. 
![404_error](../../src/assets/01082025/4.png)

I initially tested out SSTI and XSS payloads but that was a dead end; input is well sanitized.
So, next we can query for paths using various tools, I’ll use `feroxbuster`:
![enum_paths](../../src/assets/01082025/5.png)

There’s a dashboard, which kind of makes sense, since we’re being asked to authenticate. Now, before I figured out the intended way of solving this challenge, I took a lot of time with what I already knew here. If you try to access the dashboard endpoint, you get redirected to the login page, hence the 302 response. I started poking around with the dashboard, and I first asked chatGPT what kind of authentication Express applications support and the structure of the auth cookies. Here’s the response:
![gpt_response_on_express_cookies](../../src/assets/01082025/6.png)

I tried to forge these cookies, to just prompt a response from the server and see if I’ll get something, and luckily, I got something interesting (previously, the app was handling requests to non-existents paths very well, with a 404 not found page).
![server_crashes](../../src/assets/01082025/7.png)

It revealed the stack trace to us, which shows the underlying directory structure, but most importantly, that a username is required in our token. So I went to [jwt.io](https://jwt.io), and crafted a simple JWT:
![creating_unsigned_jwt](../../src/assets/01082025/8.png)

Feeding this to our website, we get an interesting response - we’re now authenticated, our username is printed out, but there’s also a message saying we’re not admin:
![successful_auth](../../src/assets/01082025/9.png)

At this point, you can tell the dashboard doesn’t reveal anything, since we’re not admin. One could try to set the username to admin and see what comes up, but that wouldn’t work (‘one’ is me, it didn’t work). But, if you’ve interacted with JWTs before, usually you set an admin property in the JWT, and that’s like the normal way these applications work. So we can try adding an admin property in the JWT data field, and see if anything changes. There are many ways to try that, through a role key, admin key, isAdmin key, and the values could be true or 1 . Trying this out, isAdminworks out and we get our flag:
![flag](../../src/assets/01082025/10.png)

### Intended Way?
Express apps are usually quite robust when it comes to directory fuzzing; if an endpoint such as /path is only enabled through GET requests, sending POST requests will return a 404 Not Found i.e.
![express_robustness_on_paths](../../src/assets/01082025/11.png)

So we could have an existing path like `/path/to/existing_path` but trying to `GET /path` will return a 404, which means that our directory fuzzers might not hit anything (I was using `feroxbuster` and `ffuf`).

At some point, I randomly thought of using `nmap` to run some default scripts since it might help fingerprint the application stack running this web server. To my surprise, we got a hidden path:
![exposing_git_dir](../../src/assets/01082025/12.png)

Nmap reveals a `.git` repo. From there, it would be easy to understand what’s going on in our web app. Download the git repo using `git-dumper`
![downloading_git_repo](../../src/assets/01082025/13.png)

Looking at the source code, you would discover hidden credentials in the commit messages i.e.
![creds_in_previous_commit](../../src/assets/01082025/14.png)

From there, you login with those credentials, which are valid but it turns out that `dinesh` isn’t the admin. Looking through the source code, we find that JWT signature verification is disabled, therefore we just need to modify our cookie to have the right header and data fields, as done previously in the first step, and that should allow us to authenticate as admin:
![disabled_jwt_signing](../../src/assets/01082025/15.png)

~THE END~

## Captcha The Flag
#### TLDR
In this challenge, there were three challenges we had to overcome.
1. First, you had to identify a `/debug` path that would allow you to set an ip address such that you can login without solving the captcha.
2. Discover the login page was vulnerable to blind sql injection, but was being protected by a web application firewall
3. Now the hard part, crafting queries that would bypass the WAF and also leak the database

### Methodology
Accessing the endpoint, we're directed to a login page:
![login_page](../../src/assets/01082025/16.png)

If we try to login with random creds but without solving the captcha:
![invalid_captcha_err](../../src/assets/01082025/17.png)

We have to enter our username, password and also solve the captcha, and trying out with random creds, we get an error:
![invalid_creds_err](../../src/assets/01082025/18.png)

At the bottom, we have a create account link (the ‘forgot password’ link is disabled), and we can try to create an account, I’ll use `cy00p:cy00p`:
![create_account](../../src/assets/01082025/19.png)

After registering, we’re redirected to the login page, and we can now try to login with our valid creds:
![login](../../src/assets/01082025/20.png)

This page doesn’t have much, and we’re told that we probably need to login as admin to find some good stuff.
When we look at burpsuite, and view the request headers, the server is running `Werkzeug/3.1.3 Python/3.11.13`, a python server framework.
![werkzeug_header](../../src/assets/01082025/21.png)

At this point, I thought of trying to do some fuzzing and see if we can identify any valid endpoints, I’ll use `feroxbuster`:
We discover the `/debug` path, which is kind of interesting. When we make a GET request to that path, we get our IP address returned to us and some interesting info:
![debug_endpoint](../../src/assets/01082025/22.png)

From that text, we get a hint that this `/debug` path was probably supposed to be disabled, since it might interfere with the login page somehow. After giving it some thought, I decided to test out various HTTP methods on that path, and the response to a `POST request` was intriguing:
![post_response_debug](../../src/assets/01082025/23.png)

The POST method is enabled, and is asking us to provide an IP address. Based on the previous sign in requests, the content type was `x-www-form-urlencoded` , so we can set that header and provide the IP address in the data field (using `curl`, we don’t have to set the content type though):
![adding_ip_address](../../src/assets/01082025/24.png)

Our IP address has been added to the database, but what does that mean? Let’s try to visit the login page, and try to login again, this time we enter a random username and password, but without solving the captcha:
![back_to_login_page](../../src/assets/01082025/25.png)

Apparently, the error we get is invalid credentials, and not invalid captcha. So setting our ip address in the debug endpoint helped us bypass the captcha.
One problem solved, now the monster awaits. So, what we can do from here?

#### SQL Injection
In BurpSuite, when we try to login, having scrapped off the data fields with captcha info, and without setting the IP in the debug path, we get a `500 Error` Response:
![500_without_ip_captcha](../../src/assets/01082025/26.png)

But after we set the IP address through the debug endpoint, we get a 200 OK message, and in the response we have the text `Invalid Credentials. Please try again`.
![200_with_ip_without_captcha](../../src/assets/01082025/27.png)

At this point, we can now start playing around with the form fields. We’ll test for sql injection, and I’ll use a “ or ‘ to test if the server is going to error out. A `"` returns a 200 OK response, but we get a message that `WAF detected SQL injection. Please try again.`
![WAF_err](../../src/assets/01082025/28.png)

We have a web application firewall probably protecting the web server against injection attacks. However, sending a single quote `'`  returns a 500 Error, which means that it wasn’t part of the blacklisted characters or strings of the web application firewall.
![single_quote_passes_waf](../../src/assets/01082025/29.png)

At this point , I felt injection was possible. But first, we need to get a valid injection string, that can bypass the WAF, but also not error out the server. After trial and error, I got something, and this actually confirmed that we're dealing with a sqlite database:
![confirm_sqlite_db](../../src/assets/01082025/30.png)


> TO BE CONTINUED 



## OUTRO
These were great challenges, huge shoutout to [ChasingFlags](https://linkedin.com/company/chasingflags/) and the challenge creators.
There was a ton to learn, and I can't wait to see what they set up in the upcoming challenges.