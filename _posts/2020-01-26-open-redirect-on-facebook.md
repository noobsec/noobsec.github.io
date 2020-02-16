---
layout: post
title: Open-redirect on Facebook (Bypass Linkshim)
date: 2019-12-22 10:00:00 PM
permalink: /project/2020-01-26-open-redirect-on-facebook/
author: dw1
categories: [ open-redirect, bugbounty, missconfig ]
tags: facebook-bugbounty open-redirect bypass linkshim
comments: true
image: https://user-images.githubusercontent.com/25837540/74606985-84089f80-5107-11ea-8a4e-6273311a5a8f.png
featured: false
---

#### TL;DR

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;My Facebook personal account is blocked for up to a month because violating Facebook community standards for over-shitposting, LMAO. I can't do anything to my account, such as update/share status, comment on status, like pages, add/accept friend requests, etc., the only thing I can do is see all the statuses of friends on my homepage, I feel like a fucking CIA agent. At first, I tried opening Facebook with a mobile browser `(m.facebook.com)` due to downloading my friend's story. I see this story through the Home page, if we look at the story automatically we will see the next story list of my friends in a certain time.

#### What is Linkshim?

![facebook_linkshim](https://user-images.githubusercontent.com/25837540/74607003-997dc980-5107-11ea-9c46-1b7f601d132a.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Every time a link is clicked on the site, the link will check that the URL against Facebook has its own internal list of malicious links, along with the lists of numerous external partners including McAfee, Google, Web of Trust, and Websense. If Facebook detects that a URL is malicious, Facebook will display an interstitial page before the browser actually requests the suspicious page.

Read the full explanation in this note: [https://www.facebook.com/10150492832835766](https://www.facebook.com/10150492832835766).

#### Proof of Concept

<img src="https://user-images.githubusercontent.com/25837540/74607029-cc27c200-5107-11ea-9d68-13efb4c8a6b0.png" width="400">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Then I decided to see the story through the profile page directly. When I click the profile picture, the endpoint story will be generated as follows:

>	https://m.facebook.com/story/view/?bucket_id=:bucket_id&viewer_session_id=:session_id&exit_uri=/profile.php?id=:profile_id

**Vulnerable parameter**: `exit_uri`

I don't see this parameter if I see stories through the Home page. This parameter works like a URL callback, if we have seen all the available stories, then we'll be directed to value of **exit_uri**. I changed value of the parameter to "`http://evilzone.org`". And it works! Page redirects on the site, there is **no linkshim filter**.

![it_works_linkshim_bypass](https://media3.giphy.com/media/2ehjfMoWiBRTy/giphy.gif)

Should be noted, Facebook doesn't consider vulnerability of open-redirect if the payload URL is shortened. In other words, Facebook doesn't check 2 or more levels deep, because users can shorten the URL to 100 levels, makes sense.

![facebook_bugbounty_reward](https://user-images.githubusercontent.com/25837540/74606985-84089f80-5107-11ea-8a4e-6273311a5a8f.png)

#### Timeline:

```
- 19-01-2020 - Vulnerability reported.
- 20-01-2020 - The Facebook team reproduces & investigates the vulnerability.
- 21-01-2020 - Triaged.
- 11-02-2020 - Resolved. Vulnerability has been patched.
- 12-02-2020 - Bounty rewarded.
```