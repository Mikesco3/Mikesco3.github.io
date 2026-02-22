---
title: Selfhosting Rustdesk
author: mikesco3
name: nycdh-jekyll
link: https://github.com/Mikesco3
date: 2026-02-06 10:34:00 -0500
categories: [Blogging,Tutorial]
tags: [guide, tutorial, linux, rustdesk, selfhosting]
# pin: false
---

___

### Important Safety Notes
> A word of **Caution** :
>
> Do not blindly copy and paste these commands.
>
> This guide is not a simple "copy/paste and hope" set of commands.
{: .prompt-warning }


## Self-Hosting RustDesk Server
1. Create a VPS somewher (like Digital Ocean)
2. Update:  `apt update && apt dist-upgrade -y`
3. Create a user (optional): `add testuser`
4. harden ssh (see section below)
5. Install Docker (see below)
6. Deploy the RustDesk Server

## RustDesk Server
### Create the docker compose 
_I like to create it in: **`/opt/stacks/rustdesk/compose.yml`**_

```yaml
networks:
  rustdesk-net:
    external: false

services:
  hbbs:
    container_name: hbbs
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21118:21118
    # I like to pin the version number, instead of latest
    image: rustdesk/rustdesk-server:1.1.15
    command: hbbs -r rustdesk.yourdomain.com:21117 -k _
    volumes:
      - ./hbbs:/root
    networks:
      - rustdesk-net
    depends_on:
      - hbbr
    restart: unless-stopped

  hbbr:
    container_name: hbbr
    ports:
      - 21117:21117
      - 21119:21119
    image: rustdesk/rustdesk-server:1.1.15
    command: hbbr  -k _
    volumes:
    ## changed this after having issues with a key error
      - ./hbbs:/root
    networks:
      - rustdesk-net
    restart: unless-stopped
```

### Spin up the server and make a note of the key:
```sh
docker compose up -d --force-recreate && docker compose logs -f
```
> Make a note of the Key:
>
> _you should see it in the docker compose logs_
> It should look somethink like this:
> `TcPb8aBlahBlahBlahBlahBlah=`
{: .prompt-info }

### Deploy the clients

> Download them from:
> [https://github.com/rustdesk/rustdesk/releases](https://github.com/rustdesk/rustdesk/releases)
{: .prompt-tip }

That should be it, just instal the rustdesk client and change the networking information:
- Settings --> Network --> change the info in **ID/Relay server**
  - ID server
  - Key

or for Windows rename the executable to:
```
rustdesk-host=remote.yourdomain.com,key=TcBlahBlahBlah=.exe
```
___
___
### Hardening SSH
Edit `/etc/ssh/sshd_config` (_at the **very least!**_):

- Disable Password-Based Authentication
``` apacheconf
PasswordAuthentication no
```
- Deny Emtpy Passwords
``` apacheconf
PermitEmptyPasswords no
```
- Deny Root Login
``` apacheconf
PermitRootLogin no
```

- Limit login attempts
``` apacheconf
MaxAuthTries 5
```
- Additional Settings
``` apacheconf
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no
```
___
### Install Docker
 
1.- Install Pre-requisites

``` sh
apt update && apt install -y apt-transport-https curl
```

2.- Add Docker’s Official GPG Key and 

``` sh
`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg`
```
3.- add Repo

```sh
`echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`

```

4.- Update and Install Docker:

``` sh
apt update && apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

___

## Sources
- RustDesk github:
[https://github.com/rustdesk/rustdesk-server](https://github.com/rustdesk/rustdesk-server)

- Installing Docker:
Source: https://linuxiac.com/how-to-install-docker-on-ubuntu-24-04-lts/

___