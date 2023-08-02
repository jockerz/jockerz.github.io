---
title: Membuat Rangkaian Iptables
description: "Rangkaian Iptables sederhana"
date: 2022-05-13 12:34:56
# en_url: "/basic-iptables-rule-chains/"
disable_i18n: true
categories: [linux, keamanan]
---


Info untuk membantu mengikuti tutorial ini bilsa dilihat di:

 - manual `iptables`
 - [Iptables Tutorial](https://phoenixnap.com/kb/iptables-tutorial-linux-firewall) by **phoenixnap.com**

 Gunakan perintah `iptables -L` untuk melihat daftar rangkaian aturan **iptables**.


## Rangkaian Aturan Baru

#### Membuat Rangkaian Aturan Baru

Pertama kita buat nama rangkaian aturan **iptables**-nya.
String **`NAMA_RANGKAIAN`** bisa diganti sesuai keinginan anda.

```
iptables -N NAMA_RANGKAIAN
```


#### Aturan 1: Beri Ijin Koneksi dari IP Tertentu

Parameter

 - `-s`: **Alamat IP**.
 - `-j`: `target` aturan (lihat `man iptables` untuk info lebih lengkap).

```
iptables -A NAMA_RANGKAIAN -s 10.11.0.0/24 -j ACCEPT
```


#### Aturan 2: Beri Ijin Koneksi Dari antarmuka **`lo`** (*localhost*)

Argumen

 - `-i lo`: antarmuka koneksi internal **localhost**

```
iptables -A NAMA_RANGKAIAN -i lo -j ACCEPT
```


#### Aturan 3: Jenis Aturan yang Diijinkan

Argumen

 - `-m conntrack`: Tambahan keadaan koneksi
 - `--cstate ESTABLISHED,RELATED`: Jenis keadaan koneksi
   - *ESTABLISHED*: Koneksi yang sudah ada/berjalan
   - *RELATED*: Paket yang berhubungan dengan koneksi yang sudah ada/berjalan, tapi merupakan paket koneksi yang berbeda

```
iptables -D NAMA_RANGKAIAN -m conntrack --ctstate ESTABLISHED,RELATED \
    -j ACCEPT
```


#### Aturan 4: Putuskan Koneksi Selain dari yang Sudah Dibuat Tadi

```
iptables -A NAMA_RANGKAIAN -j DROP
```

#### Aturan 5: Aplikasikan Aturan di Atas Ke Port yang Ditentukan

Argumen

 - `--dport 8000:8004`: port *8000* sampai dengan 8004

```
iptables -I INPUT -m tcp -p tcp --dport 8000:8004 -j NAMA_RANGKAIAN
```


## Menghapus Rangkaian Aturan

Kita berencana membuat perintah-perintah tadi dalam satu *script*.
Tapi kita *script* tersebut kita jalankan berulang kali, aturan-aturan dari akan terus ditambahkan ke daftar aturan **iptables**.

Jadi kita akan buat perintah untuk mend-*drop* rangkaian aturan kita tadi.


#### *Drop* Aturan 1 sampai 5

Parameter **iptables** yang kita gunakan adalah **`id`**, seperti berikut

```
iptables -D NAMA_RANGKAIAN -s 10.11.0.0/24 -j ACCEPT
iptables -D NAMA_RANGKAIAN -i lo -j ACCEPT
iptables -D NAMA_RANGKAIAN -m conntrack --ctstate ESTABLISHED,RELATED \
    -j ACCEPT
iptables -D NAMA_RANGKAIAN -j DROP
iptables -D INPUT -m tcp -p tcp --dport 8000:8004 -j NAMA_RANGKAIAN
```


#### Hapus Rangkaian Aturan Iptables

```
iptables -X NAMA_RANGKAIAN
```


## Simpan Semua Dalam Satu Script

Untuk membuat rangkaian aturan tadi dalam satu *script*, pertama kita jalankan dulu pertintah untuk menghapus rangkaian aturannya.


<div class="note warning">
  <h5>Peringatan</h5>
  <p>Saat <em>script</em> ini pertama kali dijalankan, akan ada error muncul karena kita mencoba meng-<em>drop</em> rangkaian aturan yang belum tersimpan.
  Tapi ini tidak jadi masalah karena <em>script</em> tetap dapat dijalankan dan rangkaian aturan baru <strong>NAMA_RANGKAIAN</strong> akan disimpan.
  </p>
</div>

Simpan dalam satu file dan jalankan sebagai **`root`**

```shell
# Bersihkan Aturan yang ada
iptables -D NAMA_RANGKAIAN -s 10.11.0.0/24 -j ACCEPT
iptables -D NAMA_RANGKAIAN -i lo -j ACCEPT
iptables -D NAMA_RANGKAIAN -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -D NAMA_RANGKAIAN -j DROP
iptables -D INPUT -m tcp -p tcp --dport 8000:8004 -j NAMA_RANGKAIAN
iptables -X NAMA_RANGKAIAN

# Buat rangkaian aturan
iptables -N NAMA_RANGKAIAN
iptables -A NAMA_RANGKAIAN -s 10.11.0.0/24 -j ACCEPT
iptables -A NAMA_RANGKAIAN -i lo -j ACCEPT
iptables -A NAMA_RANGKAIAN -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A NAMA_RANGKAIAN -j DROP
# Apply ke port 8003
iptables -I INPUT -m tcp -p tcp --dport 8000:8004 -j NAMA_RANGKAIAN
```