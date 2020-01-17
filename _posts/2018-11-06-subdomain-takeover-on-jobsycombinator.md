---
layout: post
title: Subdomain Takeover on [jobs.ycombinator.com]
date: 2018-11-06 13:37:00 PM
permalink: /project/2018-11-06-subdomain-takeover-on-jobsycombinator/
author: Blankon33
categories: [ pentesting, missconfig ]
tags: vulnerability subdomain takeover missconfig
comments: true
image: /images/ycom9.png
---

![Image of takeover](/images/ycom9.png)

beberapa bulan yang lalu, saya menemukan _missconfiguration_ pada DNS yang menyebabkan kita bisa men-_takeover_ salah satu subdomain `ycombinator.com`.

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

lalu bagaimana untuk mengetahui apakah subdomain bisa di takeover? kita bisa memeriksa satu-persatu subdomain atau menggunakan ```Aquatone-takeover --domain domain.com``` atau tools lain seperti [SubOver](https://github.com/Ice3man543/SubOver).

apabila terdeteksi _possible takeover on xxx.com_ kita cek domain tersebut, sebagai contoh gambar berikut adalah contoh dimana subdomain yang mengarahkan `cname` ke [cloudfront](https://aws.amazon.com/id/cloudfront/)

![possible takeover](/images/ycom2.png)

untuk mengetahui apakah subdomain tersebut lakukan verifikasi, bisa menggunakan `digg`

![digg image](/images/ycom10.png)

Setelah itu coba clain subdomain menggunakan AWS Cloudfront. Langkah-langkahnya yaitu:

- login dan masuk ke cloudfront dashboard
- _Create Distribution_

![create dis](/images/ycom3.png)

- pilih _get started_

![get started](/images/ycom4.png)

- set _origin_, yaitu tambahkan _bucket_ Anda, jika belom punya bisa _create_ dan upload html sebagai halaman utama

![set origin](/images/ycom5.png)

- tambahkan domain target sebagai cname

![set cname](/images/ycom6.png)

- _create_ dan simpan, tunggu sampai proses selesai

![save](/images/ycom11.png)

- apabila proses selesai, kita bisa buka subdomain target dan __BOOOM__ kita berhasil men-_takeover_ subdomain yang menggunakan _service_ cloudfront

![success](/images/ycom8.png)

__STATUS: FIXED__

Silahkan di share :D

Referensi : 

```
https://www.hacker101.com/vulnerabilities/subdomain_takeover.html
https://www.hackerone.com/blog/Guide-Subdomain-Takeovers
https://0xpatrik.com/subdomain-takeover-basics/
```

![done](https://media.tenor.co/images/c9adfe0f26b3bfe372817b1a6764849a/tenor.gif)
