---
title: Aomei Backupper Tutorial 
author: mikesco3
name: nycdh-jekyll
link: https://github.com/Mikesco3
date: 2022-11-29 11:54:00 -0500
categories: [Blogging,Tutorial]
tags: [backup, backup/aomei-backupper, guide, tutorial]
# pin: false
---
# Full System Backup

## Why, What for...
A full image backup captures the whole filesystem, so that in the event of a hard drive failure, or total loss of the computer, the system can be restored entirely as it was at the time of the backup, even if restoring onto an entirely different computer. <br>

We are going to focus on Aomei Backupper Free, as it is fairly generous in it's free version and the backups are pretty maleable. <br>
*See the bottom on this article for a [list](#software) of other backup software options.*

<hr class="solid">

## Setup
### Download Aomei Backupper Standard (free)
Download and install [Aomei Backupper free](https://www2.aomeisoftware.com/download/adb/AOMEIBackupperStd.exe) <br> 
from: https://www.aomeitech.com/ab/standard.html <br>
### Skip the offer
*Bypass and skip all of the upgrade offers.*
   ![Skip Offer](/assets/img/images/Pasted_Image_20221128121653.png)
### Install
   ![Install Now](/assets/img/images/Pasted_Image_20221128121907.png)

<hr class="solid">

## Backup
In order to make a first backup, make sure you have access to another store drive or network location that has enough available storage. <br>
### Start Aomei Backup
1. Open Aomei backupper
2. Select Backup, then
4. Select Disk Backup   
   ![Start Aomei Backup](/assets/img/images/Pasted_Image_20221128124017.png)
### Set the Backup Name and Source
4. Click the small pencil next to Task name and, 
   **name your backup**, I would recommend using a format like: YYYYMMDD_BackupName, 
   ie: `20221128_JohnDoeLaptopWin10`
5. Click **Add Disk**.
   ![Add Disk](/assets/img/images/Pasted_Image_20221128124943.png) 
   
6. **Select the Disk** that is going to be the source of the backup (*in most cases it will be **Disk0***)
7. Make a note of the drive letter for the Backup destination,
  *in our example it is drive **F:***
8. Click **Add**
![Select Backup Source](/assets/img/images/Pasted_Image_20221128130117.png) 
### Set the Destination and Start       
9. Select the Destination Path
   >[! Info] In most cases the drive letter will be correct.

   >[! Warning] **9b.** If the drive letter is incorrect, <br>
   >  click the drop down <br>
   >  and select the proper location from <br>
   >  the local path or network share.<br>
 
10. Click **Start Backup**
   ![Start Backup](/assets/img/images/Pasted_Image_20221128155830.png)

### Check Progress
11. To view more information about the Backup, including the finish time click the link below the backup progress indicator.
   ![Check Progress](/assets/img/images/Pasted_Image_20221128131629.png)
   ![Remaining Time](/assets/img/images/Pasted_Image_20221128155550.png)

### Finish and Eject
12. When the backup is done, click **Finish**.
   ![Finish](/assets/img/images/Pasted_Image_20221128155735.png)

>[! Info] Remember to Eject the drive if it is a USB Device.<br>
 > a. click the system tray option to show more icons.<br>
 > b. find the icon for USB devices.<br>
 > c. Select the Eject option above the correct device.<br>
 
 ![Eject Disk](/assets/img/images/Pasted_Image_20221128161251.png)

<hr class="solid">

## Software
### List of other Full Image Backup Software
There are many programs out there for capturing Full Image backups, some of them are:

|Name      | Description  | Free Option   | Runs from Windows | restore to smaller drive |
|----------|---------|:----------:|:----------:|:----------:|
| **[Aomei Backupper:](https://www.aomeitech.com/ab/standard.html)**         | This has a rather generous free version        | ✓ | ✓ | ✓ |
| **[Macrium Reflect free.](https://www.macrium.com/reflectfree)** | Allows you to easily back up your entire computer, make accurate and reliable images of your HDD or individual partitions and schedule backups. | ✓ | ✓| ✓ |
| **[Paragon Backup & Recovery Free](https://www.paragon-software.com/us/free/br-free/)** | It supports all major features, including file, partition/disk and entire computer backups, scheduled backups, incremental and differential backups, and more. | ✓ | ✓| ✓ |
| **[Clonezilla:](https://clonezilla.org/)** | This is free and open source, however it is not possible to restore backups onto smaller drives, it can backup, restore and clone Linux, Windows, and raw drives. | ✓ | 🗴 | 🗴 |
| **[Rescuezilla](https://rescuezilla.com/download)** | An open source backup solution that is fully compatible with clonezilla, and has a graphical interface, however is more limited in featues. It is also operating system agnostic. | ✓ | 🗴 | 🗴 |
| **Windows Backup:** | This comes with windows, however it can be complicated to restore an image onto a smaller drive than the source. | ✓ | ✓| 🗴 |
| **Acronis:** | This has been an industry standard. However they don't offer an easily accessible free product, and there has been a perceived reduce in feature attention over the years. | 🗴 | ✓| ✓|
| | | | | |