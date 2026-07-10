# Proxmox Speed-Run
Pve 8.3.3 v20250304

## Hardware
- Check CMOS battery voltage
- Check BIOS
- Install drives
- `memtest`

## Install
- ZFS, default settings, reboot
- Boot fails → strip `quiet splash=silent`, add:
```
nomodeset vesafb
acpi=off noapic nolapic simplefb nofb nomodeset
modprobe.blacklist=MODULE_NAME
```

## Helper Script
```sh
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/misc/post-pve-install.sh)"
```
disables enterprise repo, enables no-sub repo, kills nag, no auto-update

## Packages
```sh
apt install -y libcapture-tiny-perl libconfig-inifiles-perl pv lzop mbuffer net-tools sudo mc unzip screen iftop lshw sshfs make nfs-kernel-server sysstat smartmontools ethtool pv htop ntfs-3g git samba rpcbind libsasl2-modules neofetch ncdu vim tmux iperf3 archivemount fio kpartx tldr speedtest-cli
```

## Mirror an Unmatched-Size Drive (post-install)
```sh
sfdisk -d /dev/sda | sfdisk --force /dev/sdb
dd if=/dev/sda1 of=/dev/sdb1 status=progress
dd if=/dev/sda2 of=/dev/sdb2 status=progress
zpool attach -f rpool ata-HGST_HUH721212ALE600_D5G3UKBL-part3 ata-WDC_WD161KRYZ-01AGBB0_2PJSVV9Z-part3
```

## ZFS Datasets
```sh
zfs create -o atime=off -o compression=zstd -o casesensitivity=mixed -o mountpoint=/_PCS rpool/_PCS

zfs create -o atime=off -o compression=zstd -o casesensitivity=mixed rpool/_Backup
ln -s /rpool/_Backup/ /_Backup

zfs create -o atime=off -o compression=zstd -o casesensitivity=mixed rpool/_Shadows
ln -s /rpool/_Shadows/ /_Shadows

zfs create -o atime=off -o compression=zstd -o casesensitivity=mixed rpool/_VMs
```

## SSD Pool `fast200`
```sh
zpool create fast200 -o ashift=13 -o autotrim=on -d \
  nvme-KINGSTON_OM8PGP41024Q-A0_50026B7382BA1E6C-part1 \
  ata-Samsung_SSD_870_EVO_1TB_S75BNL0X407525V-part3

zpool upgrade fast200
zfs set compression=zstd fast200

zfs create -o atime=off -o compression=zstd -o casesensitivity=mixed -o mountpoint=/fast200/_VMs fast200/_VMs
```

## Users
```sh
sudo useradd -m -s /usr/bin/bash -G adm,sudo,shadow,staff,users,crontab,postfix,sambashare,dip,root pcstech && sudo passwd pcstech
smbpasswd -a pcstech
```
**Proxmox:** Datacenter → Permissions → Groups → create `masters`
```sh
pveum aclmod / -group masters -role Administrator
```
Datacenter → Permissions → Users → create user → assign to `masters`

## Samba
`/etc/samba/smb.conf`
```apacheconf
workgroup = WORKGROUP    # ALLCAPS

# under [homes]:
read only = no
create mask = 0775
directory mask = 0775

include = /etc/samba/shares.conf
```

`/etc/samba/shares.conf`
```apacheconf
[.pcs$]
   comment = Tech share
   browseable = yes
   path = /_PCS
   guest ok = no
   read only = no
   valid users = pcstech, @staff, +sambashare, @adm
   admin users = @adm, pcstech
   case sensitive = no
   directory mask = 2770

[.Backup$]
   comment = Tech share
   browseable = yes
   path = /rpool/_Backup
   guest ok = no
   read only = no
   valid users = pcstech, @staff, +sambashare, @adm
   admin users = @adm, pcstech
   case sensitive = no
   directory mask = 2770
```
```sh
chown :sambashare --recursive /_PCS && chmod g+rwx --recursive /_PCS
# repeat per share
```

## Maintenance / Extra Tools
- [ ] Install Drive Status
```sh
sudo wget -O /usr/bin/drivestatus https://raw.githubusercontent.com/Mikesco3/drivestatus.sh/main/drivestatus.sh && sudo chmod +x /usr/bin/drivestatus
```
