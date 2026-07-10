---

title: Proxmox VE Setup Guide - Speedrun
layout: post
author: mikesco3
name: nycdh-jekyll
link: https://github.com/Mikesco3
date: 2026-07-10 04:00:00 -0500
categories: [Technology, Tutorial, Virtualization, Blogging]
tags: [proxmox, proxmox-ve, linux, virtualization, zfs, homelab, server, self-hosting, guide, tutorial]
excerpt: A brief version of the guide for setting up Proxmox VE.
keywords: proxmox, proxmox ve, virtualization, zfs, linux server, homelab, self-hosting, virtual machines, containers
description: A Proxmox Installation Speedrun that assumes the reader already understands the concepts from the Detailed Proxmox VE setup guide.
pin: false

---
## Proxmox Installation, Speedrun - v20260710

> ### Need the explanations and design decisions? [Read the full guide](https://mikesco3.github.io/posts/proxmox-setup-guide/).
> https://mikesco3.github.io/posts/proxmox-setup-guide/
{: .prompt-tip }

#### In this setup
We will set up Proxmox VE using the following layout:

![Hardware](/assets/img/images/Pasted_Image_20251018151700.png)

> These commands are meant to be read, understood, and adapted (not just copy-pasted). Think of this as a recipe that keeps improving over time, and feel free to refine it for your own setup or share any insights back.
{: .prompt-warning }

___
___

## 1. Hardware Checklist

- [ ] Clean out dust; check the heatsink compound.  
- [ ] Check CMOS battery (replace if ≤ 3.0 V)
- [ ] Update BIOS
- [ ] Configure BIOS (Enable VT-x/AMD-V and IOMMU)
- [ ] Install drives
- [ ] Check the RAM with `memtest`

___
___

## 2. Install Proxmox

- [ ] Boot into the installation media and choose:
- Filesystem: **ZFS**
- Layout: **RAID1 (mirror)**
- Select only the two HDDs
- Set **compression=zstd** (optional)
- Set **`ashift=12`**

- [ ] Proceed with the install and reboot

___
___

## 3. Helper Script
- [ ] Run the [PVE Post-Install Script](https://community-scripts.org/scripts/post-pve-install?id=post-pve-install)
```sh
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/misc/post-pve-install.sh)"
```
- disable enterprise repo, 
- enables no-sub repo, 
- kill subscription nag, 
- disable corosync and clustering
- **DON'T Update yet** 

___
___

## 4. Install Basic Tools

```sh
apt update && apt upgrade -y && \
apt install -y libcapture-tiny-perl libconfig-inifiles-perl pv lzop mbuffer net-tools sudo mc unzip screen iftop lshw sshfs make nfs-kernel-server sysstat smartmontools ethtool pv htop ntfs-3g git samba rpcbind libsasl2-modules neofetch ncdu vim tmux iperf3 archivemount fio kpartx tldr speedtest-cli
```
> **Note:** Adjust for your package preference or fix any missing packages
{: .prompt-info }

- [ ] Reboot

___
___


## 5. Create ZFS Datasets

Create separate datasets for `_Backup`, `_Shadows`, and `_VMs`.

```sh
zfs create -o atime=off -o compression=zstd -o casesensitivity=mixed rpool/_Backup
ln -s /rpool/_Backup/ /_Backup

zfs create -o atime=off -o compression=zstd -o casesensitivity=mixed rpool/_Shadows
ln -s /rpool/_Shadows/ /_Shadows

zfs create -o atime=off -o compression=zstd -o casesensitivity=mixed rpool/_VMs
```
___

#### Verify Everything
```sh
zpool status
zfs list
lsblk
pvesm status

```
___
___

## 6. Create NVMe Pool
- [ ] Install the NVMe Drive.
- [ ] Locate the NVMe drive
```sh
lsblk -o name,model,serial
```

___

### Partition the NVMe Drive

> **Warning:** Do **NOT** blindly copy the device names shown in this guide. Confirm the correct NVMe device using the previous commands and substitute those values before creating the pool.
{: .prompt-danger }


```sh
cfdisk /dev/nvme0n1 
```

- [ ] create a `gpt` partition table
- [ ] create a primary partition of type: <br>
 `Solaris /usr & Apple ZFS (6A898CC3-1DD2-11B2-99A6-080020736631)`

> - [ ] Only allocate 85% of total capacity, leaving room for over-provisioning.
{: .prompt-tip }

- [ ] Write and quit `cfdisk`.

Finally the following command to label the partition:
```sh
sudo sgdisk --change-name=1:"fast200" /dev/nvme0n1 
```  
You can list disks and their IDs with:
```sh
ls -l /dev/disk/by-id/ |grep nvme0n1 |grep part1
```

___

### Create the pool
> **Danger:** Creating a ZFS pool destroys all data on the selected device. Double-check the device identifier before pressing Enter.
{: .prompt-danger }

```sh
zpool create fast200 \
  -o ashift=12       \
  -o autotrim=on     \
  nvme-2TB-Samsung_SSD_EVO_2TB_SN0001234-part1
```
#### Create the `fast200/_VMs` Dataset

```sh
zpool upgrade fast200
zfs set compression=zstd fast200

zfs create                 \
  -o atime=off             \
  -o compression=zstd      \
  -o casesensitivity=mixed \
  fast200/_VMs
```
___

### Schedule ZFS Scrubs

Run `crontab -e` as `root` and add:
```cron
0 06 * * SUN /usr/sbin/zpool scrub fast200
2 06 * * SUN /usr/sbin/zpool scrub rpool
```
___
___

## 7. Add Storages in Proxmox

**Datacenter → Storage**

- [ ] **Disable:** `local` and `local-zfs`
- [ ] **Add:** `fast200/_VMs`, `rpool/_Backup`, `rpool/_Shadows` and `rpool/_VMs`

![StorageLayout](/assets/img/images/StorageLayout_202510210105.png)

___
___
## 8. Create an Admin User

- [ ] Create the Linux User:

> _In our example, called: **`johndoe`**_

```sh
sudo useradd -m \
  -s /usr/bin/bash \
  -G adm,sudo,shadow,staff,users,crontab,postfix,sambashare,dip,root \
  johndoe

sudo passwd johndoe
```

- [ ] Create the same user for Samba access:

```sh
smbpasswd -a johndoe
```

- [ ] Add User to Proxmox

1. In the **Proxmox Web Interface**, create a group (for example, `masters`):  
   **Datacenter → Permissions → Groups → Create**

2. Grant that group administrative rights from the command line:
   ```sh
   pveum aclmod / -group masters -role Administrator
   ```

3. Then, back in the web interface, create the user:

**Datacenter → Permissions → Users → Add**
and assign the user (e.g. `johndoe@pam`) to the `masters` group.
___
___

## 9. Configure Samba
- [ ] Backup Samba config first
```sh
cp /etc/samba/smb.conf /etc/samba/smb.conf_orig
```
___

### Edit the Main Configuration

- [ ] Open `/etc/samba/smb.conf`, and edit the following lines:

```ini
workgroup = WORKGROUP

# Under [homes]
read only = no
create mask = 0775
directory mask = 0775

# At the end of the file, include the custom shares file:
include = /etc/samba/shares.conf
```

___

### Add the Shares
- [ ] Create `/etc/samba/shares.conf` and add the Shares:

Example:
```ini
[.Backup$]
   comment = Proxmox Backup Share
   path = /rpool/_Backup
   browseable = no         
   guest ok = no           
   read only = no          
   valid users = johndoe, @staff, @adm
   admin users = johndoe, @adm
   force user = johndoe
   force group = adm
   create mask = 0660
   directory mask = 2770
   case sensitive = no     
```

___

### Set the Permissions

```sh
chmod 644 /etc/samba/shares.conf 
chown :sambashare --recursive /rpool/_Backup
chmod g+rwx --recursive /rpool/_Backup

```

___

### Final Check and Start the Service

Before starting Samba, verify that the configuration is valid:

```sh
testparm   # Checks for syntax errors and lists available shares
```
If no errors appear, enable and start the service:

```sh
systemctl enable --now smbd nmbd
systemctl status smbd nmbd
```
From a Windows or macOS system, test access to the share.

___
___
## 10. Install Additional Software & Tools

### [Sanoid & Syncoid](https://github.com/jimsalterjrs/sanoid)
For Creating Snapshots and Replicating ZFS Datasets

```sh
apt update
apt install -y sanoid
```

- [ ] Create the configuration `/etc/sanoid/sanoid.conf`

> Samples are available from:
> - [The Sanoid project](https://raw.githubusercontent.com/jimsalterjrs/sanoid/master/sanoid.conf ), or 
> - [The GitHub repository for this article](https://raw.githubusercontent.com/Mikesco3/Mikesco3.github.io/refs/heads/main/_posts/_samples/sanoid.conf)
{: .prompt-info }

- [ ] Enable the timer:

```sh
systemctl enable --now sanoid sanoid-prune 
systemctl status sanoid sanoid-prune sanoid.timer 
```
___

### [DriveStatus](https://github.com/Mikesco3/drivestatus.sh)
`drivestatus` is a small utility for identifying disks.

Example output:

![drivestatus](/assets/img/images/20260104-0055_DriveStatusExample.png)

- [ ] Download and Install `drivestatus`
```sh
sudo wget -O /usr/bin/drivestatus \
  https://raw.githubusercontent.com/Mikesco3/drivestatus.sh/main/drivestatus.sh && \
sudo chmod +x /usr/bin/drivestatus
```

___

### [ZeroTier](https://www.zerotier.com/)

```sh
curl -s https://install.zerotier.com | sudo bash
```
___
___

## 11. Replicate Production VMs

Once we have Production VMs we can setup replication to their Shadow Equivalents:

> _In this example we will replicate **`VM 201`** to a **`Shadow VM 1201`**_

___

### Create the Shadow VM:

- [ ] Copy the VM Config file in `/etc/pve/qemu-server/` from `201.conf` to `1201.conf` 
- [ ] Change the Shadow VM drive entries:
- In `/etc/pve/qemu-server/1201.conf`

_**Example:**_
> **from:** **`fast200`** `vm 201`

```TOML
efidisk0: fast200_VMs-zfs:vm-201-disk-0,efitype=4m,pre-enrolled-keys=1,size=1M
tpmstate0: fast200_VMs-zfs:vm-201-disk-1,size=4M,version=v2.0
virtio0: fast200_VMs-zfs:vm-201-disk-2,discard=on,iothread=1,size=150G
```

> **To:** **`_Shadow`** `vm 1201`.

```TOML
efidisk0: rpool_Shadows-zfs:vm-1201-disk-0,efitype=4m,pre-enrolled-keys=1,size=1M
tpmstate0: rpool_Shadows-zfs:vm-1201-disk-1,size=4M,version=v2.0
virtio0: rpool_Shadows-zfs:vm-1201-disk-2,discard=on,iothread=1,size=150G
```
- [ ] **Disable the network interfaces** of the Shadow VM 1201:

___

### Create the Replication Script
In our example we're creating the script in `/root/replicateVMs-to-Shadows.sh`

```sh
#!/usr/bin/bash

## VM 201 Win11 
  /usr/sbin/syncoid --force-delete fast200/_VMs/vm-201-disk-0 rpool/_Shadows/vm-1201-disk-0 && \
  /usr/sbin/syncoid --force-delete fast200/_VMs/vm-201-disk-1 rpool/_Shadows/vm-1201-disk-1 && \
  /usr/sbin/syncoid --force-delete fast200/_VMs/vm-201-disk-2 rpool/_Shadows/vm-1201-disk-2

```

___

### Schedule the Script
Schedule this script from `crontab -e`:

```cron
5 8-18/2 * * * /root/replicateVMs-to-Shadows.sh >/dev/null 2>&1
```

## 12. Maintenance & Lifecycle Planning

- [ ] Enable Sanoid
- [ ] Configure replication
- [ ] Schedule scrubs
- [ ] Configure monitoring & Alerts
- [ ] Verify Replicatoin and Restores
- [ ] Keep Proxmox Updated
___

- [ ] Backup Strategy
- [ ] Schedule Lifecycle & Redundancy Planning
- [ ] Disaster Recovery Drills

___

### Schedule Replacements:
- [ ] NVMe Wear Level
- [ ] Hard Drives
- [ ] CMOS Battery 

___
___

If you run into installer issues, network problems, USB/KVM quirks, or hardware compatibility issues, refer to the Troubleshooting section of the [full guide](https://mikesco3.github.io/posts/proxmox-setup-guide/#troubleshooting)
[Link:](https://mikesco3.github.io/posts/proxmox-setup-guide/#troubleshooting)

___
___
