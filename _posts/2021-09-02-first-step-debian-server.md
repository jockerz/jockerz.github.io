---
title: Debian Server Setup for HTTP
description: "First Step for Debian Server set up for HTTP Server"
date: 2021-09-02 12:34:56
id_url: "#"
en: 1
categories: [linux]
---


After install Debian from ISO file, first step we gonna do are:

 - Set APT repository list

## Set APT Repository List

For this step we need to logged in as `root`.

`/etc/apt/sources.list`
```
deb http://deb.debian.org/debian/ bullseye main contrib non-free
deb-src http://deb.debian.org/debian/ bullseye main contrib non-free

deb http://deb.debian.org/debian-security bullseye-security main contrib non-free
deb-src http://deb.debian.org/debian-security bullseye-security main contrib non-free

deb http://deb.debian.org/debian/ bullseye-updates main contrib non-free
deb-src http://deb.debian.org/debian/ bullseye-updates main contrib non-free
```

Then fetch repository updates.

```shell
apt update
```

## Install Initial Packages

 - `net-tools`: network controlling commands, such as: `arp`, `ifconfig`, `netstat`, `rarp`, `nameif` and `route`
 - `openssh-server`: Connecting via other devices using SSH
 - `ufw`: The Uncomplicated FireWall is a front-end for iptables.
 - `apt-transport-https`: https support for `apt` repository
 - `sudo`: 

```shell
apt install net-tools openssh-server ufw apt-transport-https sudo
```

Enable UFW
```shell
ufw enable
```

Allow SSH Connection
```shell
# See App list
ufw app list

# Allow SSH
ufw allow "SSH"
```

See `ufw` applied rules
```shell
ufw status
```

Add `sudo` user. The `sudo` command might take effect after restart.
```shell
adduser <YOUR_USERNAME> sudo
```

## Helper Package

 - `htop`: Enhanced process viewer similar to top
 - `tmux`: Terminal multiplexer

```shell
apt install htop tmux
```

## Server Services Package

This packages is based on your needs on the server.
For introduction this packages might be useful on your server.

 - `nginx`: small, powerful, scalable web/proxy server

```shell
apt install nginx
```

Firewall setting for `nginx`.
```shell
# shortcut for `ufw allow "Nginx HTTP"` and `ufw allow "Nginx HTTPS"`
ufw allow "Nginx Full"
```

## Optional: Python Dev Packages to run Python Services

```shell
apt install python3-dev libssl-dev
```

## Clean APT Caches

```shell
apt clean
```