---
title: Debian Server Setup for HTTP
description: "First Step for Debian Server set up for HTTP Server"
date: 2021-09-02 12:34:56
id_url: "/langkah-pertama-debian-server"
en: 1
categories: [linux]
---


After install Debian from ISO file, first step we gonna do are:

 - Set APT repository list
 - Install Initial Packages
 - Install Helper Package
 - Install Server Services Package


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

Then fetch repository updates and upgrade.

```shell
apt update && apt upgrade
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

Allow SSH Connection
```shell
# See App list
ufw app list

# Allow SSH
ufw allow "SSH"
```

Enable UFW
```shell
# Warning: make sure you add "SSH" to allowed list first
ufw enable
```

See `ufw` applied rules
```shell
ufw status
```

Add `sudo` user. The `sudo` command will take effect after restart.
```shell
adduser <YOUR_USERNAME> sudo
```

## Install Helper Package

 - `htop`: Enhanced process viewer similar to top
 - `tmux`: Terminal multiplexer

```shell
apt install htop tmux
```

## Install Server Services Package

This packages is based on your needs on the server.

 - `nginx`: small, powerful, scalable web/proxy server

```shell
apt install nginx
```

Firewall setting for `nginx`.
```shell
# shortcut for `ufw allow "Nginx HTTP"` (port 80) and `ufw allow "Nginx HTTPS"` (port 443)
ufw allow "Nginx Full"
```

## Optional: Python Dev Packages to run Python Services

This packages will userful to set this server up for Python Web Application.

```shell
apt install python3-dev libssl-dev
```

Other than Python services, you might install the additional packages based on you service requirements.


## Clean APT Caches

```shell
apt clean
```
