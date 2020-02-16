---
layout: post
title: Cara Reverse Engineering APK
date: 2018-11-04 09:43:00 AM
permalink: /project/2018-11-04-cara-reverse-engineering-apk/
author: dw1
categories: [ reverse-engineer, apk ]
tags: reverse-engineering baksmali disassembler apk android-package
comments: true
image: https://www.javatpoint.com/images/androidimages/flow.jpg
featured: false
---

![Flow](https://www.javatpoint.com/images/androidimages/flow.jpg)

*(Flow Dalvik Virtual Machine. Sumber: www.javatpoint.com)*

Semua aplikasi Android yang terinstall adalah APK _(Android Package)_. Format file paket ini digunakan oleh sistem operasi Android untuk distribusi dan pemasangan aplikasi _smartphones_, dan ini berisi semua sumber daya aplikasi, aset, sertifikat, dan sebagainya, termasuk _source code_ aplikasi.

###### Catatan: jika pratinjau gambar kurang jelas, klik gambar untuk melihat dengan resolusi utuh.

Dalam postingan ini, saya gunakan aplikasi __BCA mobile__ sebagai percobaan _reverse engineering_ yang sudah terinstall di _smartphone_ saya.
Saya menggunakan _tool_ [Android Debug Bridge (adb)](https://developer.android.com/studio/command-line/adb?hl=en), dengan perintah `adb shell pm list packages` kita dapat melihat semua aplikasi yang terinstall pada _smartphone_ kita, lalu saya memindahkan file APK BCA mobile dari Android ke laptop dengan perintah `adb pull [file]`.

[![ADB Pull](/images/cara-reverse-engineering-apk_1.png)](/images/cara-reverse-engineering-apk_1.png)

Setelah itu, mari kita ekstrak file APK tersebut dengan `unzip [file]`.

[![unzip](/images/cara-reverse-engineering-apk_2.png)](/images/cara-reverse-engineering-apk_2.png)

### File Dex?

Dex adalah __Dalvik Executable__, dan file tersebut adalah bentuk kompilasi dari __Java Virtual Machine-compatible__ yang berisi seluruh kelas-kelas Java yang terdiri dari Dalvik _bytecode_. Dalvik _bytecode_ dijalankan oleh __Dalvik Virtual Machine__, seperti __Java Virtual Machine__ _(JVM)_, tetapi dioptimalkan untuk perangkat keras dan kebutuhan _smartphones_. Dalvik Virtual Machine-lah yang bertanggung jawab untuk menjalankan program pada sistem operasi Android.

Kita dapat dengan mudah mengekstrak file Dex dari APK _(bahkan dengan WinRAR, karena APK adalah format file paket yang terextend dari ZIP)_, tetapi instruksi _bytecode_ tidak __human readable__. Dengan __Baksmali__ _(disassembler)_ kita dapat mengubah file Dex kita menjadi file __Smali__ yang ditulis dalam sintaks seperti awal _(masih tingkat rendah, tetapi dapat dibaca dengan pasti)_. Disini saya menggunakan [dex2jar](https://github.com/pxb1988/dex2jar).

Karena secara _default_ file Dex dari APK bernama `classes.dex`. _Disassembler_ file Dex ke Smali dengan perintah `d2j-dex2smali [file]`.

[![baksmali](/images/cara-reverse-engineering-apk_3.png)](/images/cara-reverse-engineering-apk_3.png)

Saya mulai mencari __"sesuatu yang penting"__ secara rekursif dengan `grep`. Mengawali dengan `grep` adalah kebiasaan saya, ini berguna untuk mencari _host_ dan/ titik _endpoint_, baris konfigurasi dan/ _environment_, _secret key_ yang ditulis secara statis, dan lain-lain.

[![grep](/images/cara-reverse-engineering-apk_4.png?_)](/images/cara-reverse-engineering-apk_4.png)

Setelah memiliki file Smali, kita dapat mengkonversikannya kembali ke file Java, tetapi dengan hanya sebagian keberhasilan. Beberapa alasannya mungkin termasuk penggunaan __obfuscator__ oleh _developer_ _(alat yang dimaksudkan untuk membuat proses reverse engineering menjadi lebih sulit)_. Namun, saya akan mencoba membongkar apa yang saya bisa karena itu sangat memudahkan proses memahami apa yang dilakukan aplikasi.

_Disassembler_ file Dex ke Jar (Java) dengan perintah `d2j-dex2jar [file]`.

[![dex2jar](/images/cara-reverse-engineering-apk_5.png)](/images/cara-reverse-engineering-apk_5.png)

Untuk melihat kelas-kelas dari file Jar tersebut saya menggunakan aplikasi _standalone_ [Java Decompiler GUI](https://github.com/java-decompiler/jd-gui).

[![jd-gui](/images/cara-reverse-engineering-apk_6.png)](/images/cara-reverse-engineering-apk_6.png)

Berikut pratinjau dari Java Decompiler GUI.

[![jd-gui Preview](/images/cara-reverse-engineering-apk_7.png)](/images/cara-reverse-engineering-apk_7.png)

Kita juga dapat menyimpan _source code_ kelas-kelas Java dari file Jar dengan __jd-gui__, __File > Save All Sources (CTRL+Alt+S)__.

[![jd-gui save all sources](/images/cara-reverse-engineering-apk_8.png?_)](/images/cara-reverse-engineering-apk_8.png)

Saya coba mencari __"sesuatu yang penting"__ lagi, tetapi di dalam sekumpulan kelas-kelas Java yang baru saja saya simpan dari __jd-gui__ dan mengekstraknya.

[![grep java](/images/cara-reverse-engineering-apk_9.png)](/images/cara-reverse-engineering-apk_9.png)

Berikut hasil pemyimpanan kelas-kelas Java dari __jd-gui__ yang baru saja saya ekstrak dan juga diimport ke Sublime Text.

[![jd-gui preview java classes](/images/cara-reverse-engineering-apk_10.png)](/images/cara-reverse-engineering-apk_10.png)

### etc

Oke, sebelum terlalu jauh, saya hentikan sampai sini dulu pembahasan kali ini _(karena takut "berlebihan")_. Mungkin nanti saya akan menuliskan bagian ke-2 atau seterusnya, tentang bagaimana memodifikasi APK, atau bagaimana cara melacak aktivitas APK dari awal berjalannya aplikasi, dan lain sebagainya.

###### Perlu diingat, segala tindakan yang Anda implementasi dari tutorial ini bukan tanggung jawab dari penulis.

Tinggalkan komentar di bawah jika masih bimbang tentang apa yang sudah saya jabarkan.

## Referensi

- https://developer.android.com/studio/command-line/adb?hl=en
- https://github.com/pxb1988/dex2jar
- https://github.com/java-decompiler/jd-gui
- https://docs.oracle.com/javase/specs/jvms/se9/html/jvms-4.html
- https://en.wikipedia.org/wiki/Dalvik_(software)
- https://source.android.com/devices/tech/dalvik/dalvik-bytecode
- http://pallergabor.uw.hu/androidblog/dalvik_opcodes.html
- https://github.com/JesusFreke/smali/wiki/Registers
- https://github.com/JesusFreke/smali/wiki/TypesMethodsAndFields