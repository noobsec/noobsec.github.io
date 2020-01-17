---
layout: post
title: How We Get 4000$ in 5 Minutes
date: 2019-12-16 12:00:00 PM
permalink: /project/2019-12-16-How-We-Get-4000$-in-5-Minutes/
author: Blankon33
categories: [ pentesting, missconfig, bugbounty ]
tags: directory traversal, path traversal
comments: true
image: /images/passwd.jpg
featured: true
---

![Image of passwd](/images/passwd.jpg)

Ah, sudah lama ga nulis, btw ini adalah temuan kami kurang lebih setahun yang lalu. Kami menemukan celah Path Traversal pada salah satu subdomain di salah satu perusahaan di Bugcrowd.com. Tentunya hal ini tidak disengaja karena awalnya hanya mencoba methode dari https://snyk.io/vuln/npm:wangshuai:20170910

Langkah-langkah untuk mencari subdomain (`enumeration`) bisa menggunakan:
- [Aquatone](https://github.com/michenriksen/aquatone)

```
aquatone-discover --domain domain.com
```

- [Sublister](https://github.com/aboul3la/Sublist3r)

```
./Sublist3r.py -d domain.com
```

- dsb.

kami mendapatkan satu subdomain yaitu myaccount.redacted.com. Lalu kami mencoba dengan memasukkan payload directory traversal https://myaccount.redacted.com//etc/passwd. Diluar ekspektasi kami, muncul file /etc/passwd.

![passwd](/images/etc_passwd.png)

Tapi, kami tidak percaya kami coba untuk memasukkan payload lain seperti /etc/group 

![passwd](/images/etc_group.png)

dan /etc/hosts 

![passwd](/images/etc_hosts.png)

muncul seperti yang kami harapkan, tanpa berpikir panjang, kami report dan besoknya bug tersebut resolved



__STATUS:__
```
03 Sep 2018 21:21:25 PDT - Created the Submission
04 Sep 2018 02:40:01 PDT - Response and triaged
04 Sep 2018 07:32:30 PDT - Change severity to Critical (P1)
04 Sep 2018 07:33:24 PDT - Rewarded $4000
```

Silahkan di share :D

Referensi : 

```
https://snyk.io/vuln/SNYK-JAVA-IOVERTX-72442
```

![done](https://media.tenor.co/images/c9adfe0f26b3bfe372817b1a6764849a/tenor.gif)
