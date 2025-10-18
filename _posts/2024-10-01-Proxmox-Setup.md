---
title: Basic Troubleshooting Guide 
author:
name: nycdh-jekyll
link: https://github.com/Mikesco3
date: 2024-10-01 13:30:00 -0500
categories: [Blogging,Tutorial]
tags: [guide, tutorial, linux, proxmox, virtualization, zfs]
# pin: false
---

# Why, What for...
## Introduction
[Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment) is a popular open-source hypervisor platform that allows users to create and manage virtual machines (VMs) with ease. When setting up a Proxmox environment, it's essential to consider data protection and high availability to ensure business continuity in case of hardware failures or other disasters.  <br>

In this article, we'll walk through my current Proxmox setup using [ZFS](https://openzfs.org/wiki/Main_Page) and [sanoid and syncoid](https://github.com/jimsalterjrs/sanoid) for automated snapshotting and synchronization. <br>

## Hardware Layout and Design Considerations
The example setup used in this article consists of: 
- **Two 16TB Western Digital Gold hard drives in a mirror configuration** <br> 
which provides a robust and resilient storage solution for the Proxmox operating system. 
- In Addition, a **2TB Samsung Pro NVMe drive**, <br>
 used to store production VMs, taking advantage of the faster storage for improved performance. 
 
 The logic behind this setup is to ensure that the Proxmox operating system is protected against disk failures, while also having enough room for backups and Shadow Virtual Machines. We run the production Virtual Machines run on the NVMe drive to take advantage of it's speed. However in the background we can periodically synchronize to the larger more resilient pair of hard drives.
 
 also providing fast storage for production VMs. By separating the operating system and VM storage, we can ensure that a failure of the NVMe drive won't affect the Proxmox operating system, and vice versa. Furthermore, by synchronizing the VMs to a shadow VM on the mirrored storage, we can ensure that in the event of a disaster, we can quickly recover our production VMs.


|  NAME   |   | MODEL                   |   |  Configuration    | 
| :-----: |:-:| :---------------------- |:-:| :--------------   | 
|   sda   |   | WDC WD161KRYZ-01AGBB0   |   |  `rpool` mirror-0 | 
|   sdb   |   | WDC WD161KRYZ-01AGBB0   |   |  `rpool` mirror-0 | 
| nvme0n1 |   | Samsung SSD 990 PRO 2TB |   |  `fast200 `       | 

___

## Install Proxmox 


1. Install Proxmox Using the following configuration  
    2x 14TB WD Gold drives in a ZFS Raid 1 (mirror) `rpool` where proxmox is installed

```
root@pve0:/dev/disk/by-id# zpool status

  pool: rpool
config:
        NAME                                          
        rpool                                         
          mirror-0                                    
            ata-WDC_WD161KRYZ-01AGBB0_XXXX-part3  
            ata-WDC_WD161KRYZ-01AGBB0_XXXX-part3  

```    

___
## Index
1. Prepare the Hardware
2. Install Proxmox with defaults
3. Run helper script to fix repos.
4. Install additional packages
5. Created zfs datasets <strong><code>_PCS, _Backup, _Shadows, and _VMs </code></strong>
6. Created the `fast200` SSD Pools along with it's dataset <strong><code>fast200/_VMs </code></strong>
7. Added Datasets to the Storage section of the Proxmox Web Interface 
8. Added a PersonalUser user and added to proxmox 
9. Created the <strong><code> masters </code></strong> User Group and granted Admin rights
10. Installed additional packages
11. Installed drivestatus
12. Configured Samba
13. Installed `sanoid - syncoid`
14. Configure Replication and Shadow VMs
____
____

## 1. Prepared the Hardware
 - Visually Inspect.
 - Clean the dust, checked heatsink compound
 - Check the voltage of the CMOS Battery (replace if under 3v)
 - Reconfigure the BIOS (enable virtualization options)
 - Test RAM (using [memtest86+](https://memtest.org/))
 - Install the Hard Drives

____

## 2. Installed proxmox with defaults
 Installed Proxmox with mostly defaults. However when selecting the drives, I chose ZFS and created a RAID 1 with the 2x 16TB WD Gold
 
```
root@pve0:/dev/disk/by-id# zpool status

  pool: rpool
config:
        NAME                                          
        rpool                                         
          mirror-0                                    
            ata-WDC_WD161KRYZ-01AGBB0_XXXX-part3  
            ata-WDC_WD161KRYZ-01AGBB0_XXXX-part3  

```

  I installed proxmox onto a ZFS mirror of the 2 16 Terabyte Western Digital Gold Drives.
 _I'm using the mirrored drives for mr_



12. switched visudo from nano to vim
``` sh
sudo update-alternatives --config editor
```


10. Installed zerotier and joined (service@pointcomputerserv.com rpdnet)
11. copied over the backup of the etc.tar 
12. added the entries for pve1 and pve 2 to hosts file
13. created fast200
	1. created first a boot partition and efi partition then finally the partition where the data would go.
	   *thinking to leave room if I even need to make this a bootable drive.*
	   followed [[Proxmox _ Clone zfs boot disk]]
	2. Created SSD zfs pools

>[!code]- Create fast200 SSD  pool
>  ___
  ```sh
  zpool create fast200 \
  -o ashift=13 \
  -o autotrim=on -d \
  ata-Samsung_SSD_870_EVO_2TB_S6PNNS0W204746W-part3
 ```
 ___
  ``` sh
  zpool create fast300 \
  -o ashift=13 \
  -o autotrim=on -d \
  nvme-INTEL_SSDPEDMW012T4_CVCQ637000EU1P2BGN_1-part3
  ```
  ___

13.  Configured email forwarding

___

Installed additional packages:
``` sh
apt install archivemount build-essential debhelper ethtool fio git htop iftop iperf3 kpartx libcapture-tiny-perl libconfig-inifiles-perl libsasl2-modules lshw lzop make mbuffer mc ncdu neofetch net-tools nfs-kernel-server ntfs-3g pv pv rpcbind samba screen smartmontools sshfs sudo sysstat tldr tmux unzip vim 
```
___
 and 
    Install Promox following the prompts on the CD, when choosing where to install step, Select the 2 Hard drives and create a ZFS Raid1 mirror 
    IP: `192.168.2.`**`40`**
  
2. After rebooting disabled the logitech module so the usb kvm would work 
   [[Linux Mint _ USB KVM not working after upgrade]]
```sh
echo "blacklist hid_logitech_dj" >> /etc/modprobe.d/pve-blacklist.conf   && \
update-initramfs -u                                                      && \
pve-efiboot-tool refresh
```
3.  Added proxmox ip to host *to resolve slow dowload speed from their repo*
    add to `/etc/hosts` the following:
``` apacheconf
212.224.123.70    download.proxmox.com    
```

4. Use Helper Scripts:
### Proxmox Helper Scritps:
[[Proxmox _ Helper Scripts#Proxmox Helper Scripts]]  
[Proxmox Helper Scripts / Post Install](https://helper-scripts.com/scripts?id=Proxmox+VE+Post+Install)  


``` sh
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"
```
 - Disabled the Enterprise repos
 - Enable non-subscription free repos
 - Enable ceph 
 - Disable susbscription nag.

5. Installed additional packages
### Install Additional Packages

``` sh
apt install libcapture-tiny-perl libconfig-inifiles-perl pv lzop mbuffer net-tools sudo mc unzip screen iftop lshw sshfs make nfs-kernel-server sysstat smartmontools ethtool pv htop ntfs-3g git samba rpcbind libsasl2-modules neofetch ncdu vim tmux iperf3 archivemount fio kpartx tldr 
```
   
#### Install `drivestatus`
``` sh
sudo wget -O /usr/bin/drivestatus https://raw.githubusercontent.com/Mikesco3/drivestatus.sh/main/drivestatus.sh &&  sudo chmod +x /usr/bin/drivestatus
```

6. Created Users
### Create Users
#### To create Linux user 
 
``` bash
sudo useradd -m -s /usr/bin/bash -G adm,sudo,shadow,staff,users,crontab,postfix,sambashare,dip,root personaluser && sudo passwd personaluser
```
*type a password*

#### Create the Samba User for the Windows Shares
``` bash
smbpasswd -a personaluser
```
*re-type a password*

#### Add user to Proxmox
[[Proxmox_ Create Users]]

Create a user in Proxmox and Grant Admin rights

1. Within the web admin console and **create a group:** ( _I called it_ **`masters`** )  
    **Datacenter → Permissions → Groups**
    
2.  From the command line **grant the administrator** role to the group.    
```bash
pveum aclmod / -group masters -role Administrator
```

3. Within the web admin console, go to datacenter and **create the user:**  
    **Datacenter → Permissions → Users**
   And assign the user to the **`masters`**  group

____
### Create ZFS Datasets
#### Create \_PCS Dataset
```bash
zfs create \
  -o atime=off \
  -o compression=zstd \
  -o casesensitivity=mixed \
  -o mountpoint=/_PCS \
  rpool/_PCS  
```
#### Create \_Backup Dataset
```bash
zfs create \
  -o atime=off \
  -o compression=zstd \
  -o casesensitivity=mixed \
  rpool/_Backup 
```
#### Create \_Shadows Dataset
```bash
zfs create \
  -o atime=off \
  -o compression=zstd \
  -o casesensitivity=mixed \
  rpool/_Shadows 
```
#### Create \_VMs Dataset
```bash 
zfs create \
  -o atime=off \
  -o compression=zstd \
  -o casesensitivity=mixed \
  rpool/_VMs
```

### Create SSD Pool `fast200` \_VMs Dataset
```bash 
zpool create fast200 \
  -o ashift=13 \
  -o autotrim=on -d \
ata-Samsung_SSD_870_EVO_1TB_S75BNL0X407525V-part3
```

Upgrade and enable zstd compression:
```
zpool upgrade fast200
zfs set compression=zstd fast200
```
#### Create the `fast200\_VMs` Dataset

```bash 
zfs create \
  -o atime=off \
  -o compression=zstd \
  -o casesensitivity=mixed \
  -o mountpoint=/_VMs \
  fast200/_VMs
```

8. Configure Samba:  Orig Article: [[Linux _ Create Samba Shares]]
### Configure Samba 
  edit the `/etc/samba/smb.conf` file, where the following lines:

`workgroup =`**`WORKGROUP`**             or whatever the net workgroup will be, just make it ALLCAPS!

```apacheconf

# Then under homes:  
	read only = no  
	create mask = 0775  
	directory mask = 0775  

include = /etc/samba/shares.conf

```

Then in the file `/etc/samba/shares.conf` place the shares:

```apacheconf
# Then define the shares  

[.pcs$]
   comment = Tech share
   browseable = yes
   path = /_PCS
   guest ok = no
   read only = no
   valid users = personaluser, @staff, +sambashare, @adm
#   users = @adm, personaluser
#   force user = personaluser
#   force group = adm
   case sensitive = no
   directory mask = 2770

# Optionally you can share the Backup location
[.Backup$]
   comment = Tech share
   browseable = yes
   path = /rpool/_Backup
   guest ok = no
   read only = no
   valid users = personaluser, @staff, +sambashare, @adm
#   users = @adm, personaluser
#   force user = personaluser
#   force group = adm
   case sensitive = no
   directory mask = 2770

```

Finally, change the ownership and permissions of the destination folders:
``` sh
chown :sambashare --recursive /_PCS && \
chmod g+rwx --recursive /_PCS

## do the same for any remaining locations
```

___
9. Configure the Storage options in Proxmox
### Add storages to Proxmox
Go to Datacenter --> Storage then add

| **ID**            | **Type**  | **Content**                     | **Path / Target** | Enabled |
| ----------------- | --------- | ------------------------------- | ----------------- | ------- |
| fast200_VMs-dir   | Directory | VZDump, ISO, container template | `/fast200/_VMs`   | Yes     |
| fast200_VMs-zfs   | ZFS       | Disk Image, Container           | `fast200/_VMs`    | Yes     |
| ~~**local**~~     | Directory | VZDump, ISO, container template | `/var/lib/vz`     | **NO**  |
| ~~**local-zfs**~~ | ZFS       | Disk Image, Container           | `rpool/data`      | **NO**  |
| rpool_Backup-dir  | Directory | VZDump, ISO, container template | `/_Backup`        | Yes     |
| rpool_Shadows-zfs | ZFS       | Disk Image, Container           | `rpool/_Shadows`  | Yes     |
| rpool_VMs-dir     | Directory | VZDump, ISO, container template | `/_VMs`           | Yes     |
| rpool_VMs-zfs     | ZFS       | Disk Image, Container           | `rpool/_VMs`      | Yes     |

![[Pasted image 20240521131635.png]]

___
### Copied over some ISO Images and Templates
Browsed from Windows to the newly created share:
`\\the-pve0-ip-address\.Backup$\template\iso`
*On my network the address was: `\\192.168.2.40\.Backup$\template\iso`*

Then copied over some ISO files and backup templates.

___
10. Schedule Scrub and replication:
## Scrub
### Schedule Scrubs 
The scrub examines all data in the specified pools to verify that it checksums correctly.

To do this, run the command `crontab -e` as root and add the following:
``` apacheconf
0 06 * * SUN /usr/sbin/zpool scrub fast200
2 06 * * SUN /usr/sbin/zpool scrub rpool
```
*This should schedule the NVME SSD (`fast200`) pool to be scanned on Sunday night at 0:06 am, and the Hard Drive pool (`rpool`) to be scanned on Sunday night at 2:06 am*
___

## Replication - Method 1
[[Proxmox _ Syncoid VMs to Backup]]

 Replicate the VM's from the fast Storage to the `_Shadows` dataset on the larger Hard Drive mirror pool
### 1. Install Sanoid

[[Proxmox _ Install Sanoid and Syncoid manually]]
>[!todo]- Install Syncoid
>   ![[Proxmox _ Install Sanoid and Syncoid manually]]
>   


#### 2. Install the replication script:
 1. Download the script:
 ```sh
wget https://raw.githubusercontent.com/Mikesco3/syncoid-replicate.sh/main/syncoid-replicate.sh
```
 
 2. Mark it executable: 
```sh 
 chmod +x syncoid-replicate.sh
```
 3. Test the script:
``` sh
./syncoid-replicate.sh fast200/vm-vmname rpool/vm-vmname
```
#### 3. Schedule the Replication:
1. Open Cron for scheduling the job
``` sh
   crontab -e
```
2. Add the following line: _In the example we are replicating vm-name at 12:06 near midnight_
``` sh
06 0 * * * (/path/to/syncoid-replicate.sh fast200/vm-vmname rpool/_Shadows/vm-vmname) > /dev/null
```
#### 4. Optionally: copy the conf file for the VM and make it a bootable shadow:
Copy the conf file for the VM (say we are replicating vm 203 on fast200 to vm 103 on rpool):
``` sh
cd /etc/pve/qemu-server
cp 203.conf 103.conf
```
Then edit the lines corresponding to the VM's virtual disk, so in our example, open `203.conf`, 
change the lines:
![[Pasted image 20231016134544.png]]

to:
![[Pasted image 20231016134603.png]]

and, turn off automatically starting the vm when proxmox reboots, by setting:
 `onboot: 0`

___
## Replication - Method 2

### Create a script

Create a Script where only the specified virtual disks are replicated to the backup.

```bash
vim /tank100/_PCS/sync-fast200-to-tank100.sh
```

and paste the following (adjust for the dataset and virtual disks)  
_in this example_ we are replicating **disk-0, 1 and 2** from **fast200/_VMs** to **tank100/_Backup**

#### simpler script:
``` sh
#!/usr/bin/bash
## RPDFS00-VM 
### nvme (fast200) to HD (rpool/_Shadows)
        /usr/sbin/syncoid --force-delete fast200/vm-105-disk-0 rpool/_Shadows/vm-1105-disk-0  
        /usr/sbin/syncoid --force-delete fast200/vm-105-disk-1 rpool/_Shadows/vm-1105-disk-1  
        /usr/sbin/syncoid --force-delete fast200/vm-105-disk-2 rpool/_Shadows/vm-1105-disk-2  

### nvme (fast200) to pve1 HDs (pve1:tank100/_Shadows)
        /usr/sbin/syncoid --force-delete fast200/vm-105-disk-0 pve1:tank100/_Shadows/vm-1105-disk-0  
        /usr/sbin/syncoid --force-delete fast200/vm-105-disk-1 pve1:tank100/_Shadows/vm-1105-disk-1  
        /usr/sbin/syncoid --force-delete fast200/vm-105-disk-2 pve1:tank100/_Shadows/vm-1105-disk-2  

```
#### more advanced script with logging

```sh
#!/usr/bin/bash

## Server0
## fast200/_VMs/vm-101-disk-0
( /usr/local/sbin/syncoid --force-delete fast200/_VMs/vm-101-disk-0 rpool/_Shadows/vm-1101-disk-0  > /tmp/vm-101-disk-0.log 2>&1 || cat /tmp/vm-101-disk-0.log ) && rm /tmp/vm-101-disk-0.log

## fast200/_VMs/vm-101-disk-1
( /usr/local/sbin/syncoid --force-delete fast200/_VMs/vm-101-disk-1 rpool/_Shadows/vm-1101-disk-1  > /tmp/vm-101-disk-1.log 2>&1 || cat /tmp/vm-101-disk-1.log ) && rm /tmp/vm-101-disk-1.log

## fast200/_VMs/vm-101-disk-2
( /usr/local/sbin/syncoid --force-delete fast200/_VMs/vm-101-disk-2 rpool/_Shadows/vm-1101-disk-2  > /tmp/vm-101-disk-2.log 2>&1 || cat /tmp/vm-101-disk-2.log ) && rm /tmp/vm-101-disk-2.log

## fast200/_VMs/vm-101-disk-3
( /usr/local/sbin/syncoid --force-delete fast200/_VMs/vm-101-disk-3 rpool/_Shadows/vm-1101-disk-3  > /tmp/vm-101-disk-3.log 2>&1 || cat /tmp/vm-101-disk-3.log ) && rm /tmp/vm-101-disk-3.log

## Unifi VM
## fast200/_VMs/vm-104-disk-0
( /usr/local/sbin/syncoid --force-delete fast200/_VMs/vm-104-disk-0 rpool/_Shadows/vm-1104-disk-0  > /tmp/vm-104-disk-0.log 2>&1 || cat /tmp/vm-104-disk-0.log ) && rm /tmp/vm-104-disk-0.log

## fast200/_VMs/vm-104-disk-1
( /usr/local/sbin/syncoid --force-delete fast200/_VMs/vm-104-disk-1 rpool/_Shadows/vm-1104-disk-1  > /tmp/vm-104-disk-1.log 2>&1 || cat /tmp/vm-104-disk-1.log ) && rm /tmp/vm-104-disk-1.log
```

### Schedule the Script

Schedule the script created in the previous step with `crontab -e`  
(_in the example, we'll set it to run every 6 hours on minute 7_)

```apacheconf
0 */6 * * *  /_PCS/_scripts/sync_fast200_to_rpool-Shadows.sh
```

#linux #linux/proxmox #zfs/sanoid #zfs/replication #zfs #zfs/syncoid #linux/cron
___
## Replication - Method 3
### Zrepl 
Another replication method is via `zrepl`, however the configuration for this can be more challenging.
>[!todo]- Install Zrepl 
> ![[Zrepl_ Install and Configure]]
___

___
## Optional Software
### zoxide:
[https://github.com/ajeetdsouza/zoxide](https://github.com/ajeetdsouza/zoxide)
Initiated zoxide:
``` sh 
apt install -y zoxide 
```
  run the command AND add to the end `.bashrc`
``` sh
eval "$(zoxide init bash)"
```

___
### zerotier
```bash
curl -s https://install.zerotier.com | sudo bash
```

>[!NOTE]- When asking about IPv6 **RFC4193** (/128 for each device)
>
> RFC4193 defines Unique Local IPv6 Unicast Addresses, which are similar in concept to IPv4 private addresses (like those in the 192.168.x.x range). These addresses are not globally routable and are intended for local communication within a specific network. 
> 
> Enabling RFC4193 in ZeroTier means that each device in your network will receive a unique IPv6 address from the unique local address range. Each device will have its own IPv6 address, allowing them to communicate within the ZeroTier network using IPv6.
> 
> Enabling this option can be useful if you want to ensure that each device on your ZeroTier network has a unique IPv6 address for communication purposes. It helps in organizing and managing the network traffic efficiently.

>[!NOTE]- When asking about **6PLANE** (/80 routable for each device)
> 6PLANE is a method for automatically assigning IPv6 addresses in a ZeroTier network. When you choose the 6PLANE option and enable it for each device, it means that each device will receive a routable IPv6 address from the /80 subnet.
>
> In IPv6 addressing, a /80 subnet provides a large number of available addresses. Each device in the ZeroTier network will get its own IPv6 address within this subnet, allowing it to communicate not only within the local network but also potentially with other networks on the internet, depending on the routing configuration.
> 
> Enabling 6PLANE with a /80 subnet for each device can be beneficial if you want your ZeroTier devices to have globally routable IPv6 addresses, potentially allowing them to communicate with other IPv6-enabled devices and networks across the internet. This can be particularly useful if you need your ZeroTier network to interact with external IPv6 networks.
> 

___

### Link the ISO VM
create an iso folder in `/_Backup`**:**  and link the `iso` folder
 ```bash
mkdir -p /_Backup/template/iso  && \
mv /var/lib/vz/template/iso /var/lib/vz/template/iso1  && \
ln -s  /_Backup/template/iso/ /var/lib/vz/template/iso
```

create an dump folder in `/_Backup`**:**  and link the `dump` folder
 ```bash
mkdir -p /_Backup/dump  && \
mv /var/lib/vz/dump /var/lib/vz/dump1  && \
ln -s  /_Backup/dump/ /var/lib/vz/dump
```

___

## Backup
### Backup the Proxmox Linux OS 
Download and set backup 

``` sh
wget https://raw.githubusercontent.com/Mikesco3/pve-node-backup.sh/main/pve-node-backup.sh  && \
chmod +x ./pve-node-backup.sh && \
cp ./pve-node-backup.sh /usr/bin/pve-node-backup.sh 
```

or by cloning the git repo:
```bash
git clone https://github.com/Mikesco3/pve-node-backup.sh.git \
chmod +x ./pve-node-backup.sh/pve-node-backup.sh \
cp ./pve-node-backup.sh/pve-node-backup.sh /usr/bin/pve-node-backup.sh \
```

Now Schedule the task by running `crontab -e`, and adding the following at the bottom:

```apacheconf
    06 01  * * SUN /usr/bin/pve-node-backup.sh  /tank100/_Backup 6 weekly
```
*Every Sunday at 01:06 am, keeping 6 versions*

```
## Every 2nd day of the month at 1:06am
6 1 2 * * /usr/bin/pve-node-backup.sh /_Backup 7 monthly
```
*Every 2nd day of the Month at 01:06 am, keeping 7 months*

```
## Every 2nd day of the month at 01:06am
06 01 2 * * /usr/bin/pve-node-backup.sh /_Backup 7 monthly
```
*Every 2nd day of the Month at 01:06 am, keeping 7 months*
___

## Email alerting 
1. Add CC emails for alerting purposes:  
    Edit the **`/root/.forward`** file

```apacheconf
# cat /root/.forward
|/usr/bin/pvemailforward
foo@company.com, bar@company.com, ...
```

source: [https://forum.proxmox.com/threads/proxmox-alert-emails-can-you-automatically-cc-people.53332/](https://forum.proxmox.com/threads/proxmox-alert-emails-can-you-automatically-cc-people.53332/)
## Added a way to start the VM's from windows
>[!NOTE]- Ran a shell script from a batch file via putty
>![[Proxmox _ Start VMs from a Batch file]]
___

## Troubleshooting:

### If lower than 64GB of ram consider setting the arc size:
>[!caution]- If lower than 64GB of RAM 
> Consider setting the arc size, 
> Instructions here:
> ![[ZFS _ set arc size]]

### If no network after update and reboot
>[!caution]- Check if the networking interface didn't get renamed
> to check the log since the previous boot run `journalctl -b`
> 1. run up a and get a list of the current network devices
> 2. go to the `/etc/networking/interfaces` and make sure that the brige is assigned to the correct device.
> source: https://forum.proxmox.com/threads/network-state-down-after-update-reboot.130708/

 #### Permanent fix found:
 
Here's a scriptlet to automatically generate these files. It outputs them to `/tmp`, then you can copy to `/etc/systemd/network` if they look correct. 
  

```bash
#!/bin/bash

OUTDIR=/tmp

for ETH in `ls -1 /sys/class/net/ | sort -V | grep -v -e "^bond"  -e "^fwbr" -e "^fwln" -e "fwpr" -e "^lo$" -e "^tap" -e "^vmbr"`
    do echo "-------------------------"
       echo "[Match]" > ${OUTDIR}/10-${ETH}.link
       echo -n "MACAddress=" >> ${OUTDIR}/10-${ETH}.link
       ip link show dev ${ETH} | grep "link/ether" | awk '{print $2}' >> ${OUTDIR}/10-${ETH}.link
       echo "Type=ether" >> ${OUTDIR}/10-${ETH}.link
       echo "" >> ${OUTDIR}/10-${ETH}.link
       echo "[Link]" >> ${OUTDIR}/10-${ETH}.link
       echo "Name=${ETH}" >> ${OUTDIR}/10-${ETH}.link
       cat ${OUTDIR}/10-${ETH}.link
done
echo "-------------------------"
```

  
After copying them into /etc/systemd/network, be sure to then run this:  


```bash
sudo update-initramfs -u -k all
```

source: https://forum.proxmox.com/threads/the-joys-of-spontaneous-network-interface-renaming.153817/post-699566
source2: https://pve.proxmox.com/pve-docs/pve-admin-guide.html#network_override_device_names
#linux/proxmox/pve8/troubleshooting #Troubleshooting #linux/networking 

___

___
#linux #linux/proxmox #guide #zfs