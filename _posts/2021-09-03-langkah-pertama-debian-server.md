---
title: Persiapan Installasi Debian Server
description: "Persiapan Installasi Debian untuk server HTTP"
date: 2021-09-02 12:34:56
en_url: "/first-step-debian-server/"
categories: [linux]
---


Berikut langkah untuk persiapan menggunakanan Debian untuk Server setelah instalasi selesai.

 - Mempersiapkan Repository
 - Instal Paket Awal
 - Install Paket-paket Pembantu
 - Install Paket-paket Servis


## Mempersiapkan Repository

Pertama, kita harus login dengan user `root`.

`/etc/apt/sources.list`
```
deb http://deb.debian.org/debian/ bullseye main contrib non-free
deb-src http://deb.debian.org/debian/ bullseye main contrib non-free

deb http://deb.debian.org/debian-security bullseye-security main contrib non-free
deb-src http://deb.debian.org/debian-security bullseye-security main contrib non-free

deb http://deb.debian.org/debian/ bullseye-updates main contrib non-free
deb-src http://deb.debian.org/debian/ bullseye-updates main contrib non-free
```

Lalu `update` dan `upgrade`.

```shell
apt update && apt upgrade
```

## Instal Paket Awal

 - `net-tools`: untuk perintah-perintah mengontrol jaringan, seperti: `arp`, `ifconfig`, `netstat`, `rarp`, `nameif` and `route`
 - `openssh-server`: Untuk terhubung menggunakan perangkat lain dengan SSH
 - `ufw`: Frontend untuk `iptables` dengan penggunaan yang lebih mudah.
 - `apt-transport-https`: dukungan `https` untuk repositori `apt`
 - `sudo`: menambahkan hak __super user__ untuk user biasa.

```shell
apt install net-tools openssh-server ufw apt-transport-https sudo
```

Ijinkan dan jalankan `ufw`
```shell
ufw enable
```

Ijinkan koneksi SSH
```shell
# See App list
ufw app list

# Allow SSH
ufw allow "SSH"
```

Untuk melihat pengaturan firewall `ufw` yang sudah disimpan gunakan perintah `ufw status`.

Tambahkan user ke grup `sudo`. Setelah ditambahkan perintah `sudo` mungkin baru bisa dijalankan setelah OS di nyalakan ulang.
```shell
adduser <YOUR_USERNAME> sudo
```

## Install Paket-paket Pembantu

 - `htop`: Pengembangan perintah `top`
 - `tmux`: Terminal multiplexer

```shell
apt install htop tmux
```

## Install Paket-paket Servis

Paket-paket yang perlu diinstal disini sesuai kebutuhan.
Setelah servis yang dibutuhkan ter-install, pastikan _firewall_ di atur dengan perintah `ufw` sesuai kebutuhan.

 - `nginx`: web/proxy server ringan dan terskalakan

```shell
apt install nginx
```

Pengaturan _firewall_ untuk  `nginx`.
```shell
# singkat dari `ufw allow "Nginx HTTP"` dan `ufw allow "Nginx HTTPS"`
ufw allow "Nginx Full"
```

## Pilihan: Install Paket untuk Persiapan Servis dengan Python

Paket ini berguna untuk mempersiapkan server untuk membuat aplikasi web dengan Python.

```shell
apt install python3-dev libssl-dev
```

## Bersihkan Cache APT

```shell
apt clean
```
