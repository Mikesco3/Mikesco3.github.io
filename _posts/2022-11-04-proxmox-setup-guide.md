---
title: Proxmox Setup Guide
author:
  name: Mikesco3
  link: https://github.com/Mikesco3
date: 2022-11-04 23:40:00 -0500
categories: [Blogging,Tutorial]
tags: [proxmox, linux, zfs]
---

## **PCS-Doc_ Proxmox Setup Guide v20220318**


## Install Proxmox 

1. Installed Proxmox Using the following configuration  
  2 SAS drives 300GB in a Mirror and 2  
  Install Promox following the prompts on the CD, only in the choose where to install step, 
  Select the 2 drives and create a ZFS Raid1 mirror 

2. When I rebooted I ran into an error where it was stuck on (initrd) 
```bash 
   zpool import -R / rpool
```
   then pressed CTR-D  
   
  #### To make the change permanent, once it booted  
```bash
 echo -n " rootdelay=15" >> /etc/kernel/cmdline  
 pve-efiboot-tool refresh
```
   
### Add repos and packages
3. Run Updates, upgrade and install basics
4. Add the no subcription repository for proxmox   
Edit **`/etc/apt/sources.list.d/pve-enterprise.list`** and insert these lines: 
``` 
# Proxmox no subscription  
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription  
```

5. Run Updates , upgrade and install basics
```bash
apt update && \
apt dis-upgrade && \
apt install --yes net-tools sudo mc unzip screen iftop lshw sshfs make nfs-kernel-server sysstat smartmontools ethtool pv htop ntfs-3g git samba rpcbind libsasl2-modules neofetch ncdu vim tmux iperf3 archivemount fio kpartx  

wget https://github.com/muesli/duf/releases/download/v0.8.0/duf_0.8.0_linux_amd64.deb && \

dpkg -i duf_0.8.0_linux_amd64.deb
``` 

### Disable Proxmox Suscription Notice
6. Edit the file
**# vi /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js  
** Then search for **“No valid subscription”** and replace on **line 507 or 473**
```javascript
Ext.Msg.show({
    title: gettext('No valid subscription'),
```

With 
```javascript
void({ //Ext.Msg.show({
  title: gettext('No valid subscription'),
```
 Restart the service 
`# systemctl restart pveproxy.service`  

___
