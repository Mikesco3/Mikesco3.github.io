---
title: Upgrade Samsung SSD Firmware from Linux
author: mikesco3
name: nycdh-jekyll
link: https://github.com/Mikesco3
date: 2025-11-28 10:34:00 -0500
categories: [Guides,Tutorial]
tags: [guide, tutorial, linux, proxmox, firmware, samsung, nvme]
# pin: false
---

# Upgrade Samsung SSD Firmware from Linux
___


## Before We Begin
### Introduction — Why This Matters

I ran into an issue where my Samsung NVMe Pro SSD would intermittently lock up under Proxmox.
After some research, I discovered that outdated firmware can cause stability problems—and that Samsung provides a bootable firmware updater packaged in an ISO.

This guide walks through how to extract the firmware from Samsung’s ISO and update the drive **directly from Linux**, without needing to boot into a separate environment.

I was having an issue where my Samsung NVME Pro SSD was randomly locking up in proxmox.
So in searching around I found the process for upgrading the NVME's firmware from linux:

___

### Overview of the Process:
1. Download the correct firmware ISO from Samsung.
2. Mount and extract the firmware payload.
3. Run the upgrade tool.
4. Shut down and boot back up.

___

### Important Safety Notes
> A word of **Caution** :
>
> Do not blindly copy and paste these commands.
> Always verify:
> - You have the correct firmware for your exact SSD model.
> - The device path you flash is correct.
> - You have backups in case something goes wrong.
>
> This guide is not a simple "copy/paste and hope" set of commands.
{: .prompt-warning }

### Step-by-Step Instructions:

#### 1. Download the firmware ISO

Go to Samsung’s official support site and download the firmware image for your specific model:

Samsung SSD Tools & Firmware:
>[(https://semiconductor.samsung.com/consumer-storage/support/tools/](https://semiconductor.samsung.com/consumer-storage/support/tools/)

(Example below uses the Samsung 990 Pro firmware ISO.)

```sh
wget https://download.semiconductor.samsung.com/resources/software-resources/Samsung_SSD_990_PRO_7B2QJXD7.iso
```

#### 2. Install required tools
```sh
apt-get -y install gzip unzip wget cpio
```

#### 3. Mount the ISO 
```sh
mkdir /mnt/iso
sudo mount -o loop ./Samsung_SSD_990_PRO_7B2QJXD7.iso /mnt/iso/
```

#### 4. Extract the firmware payload 
```sh
mkdir /tmp/fwupdate
cd /tmp/fwupdate

# Extract the initrd from the ISO
gzip -dc /mnt/iso/initrd | cpio -idv --no-absolute-filenames
```

#### 5. Upgrade the firmware 

```sh
cd /tmp/fwupdate/root/fumagician/
sudo ./fumagician
```

___

### Sources:
- [https://blog.quindorian.org/2021/05/firmware-update-samsung-ssd-in-linux.html/](https://blog.quindorian.org/2021/05/firmware-update-samsung-ssd-in-linux.html/)
- [https://community.frame.work/t/solved-samsung-ssd-980-pro-firmware-update-on-linux-how/27046/2](https://community.frame.work/t/solved-samsung-ssd-980-pro-firmware-update-on-linux-how/27046/2)

___
