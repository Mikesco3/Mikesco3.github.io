---
title: Creating a Recovery Partition in Windows 11
author: mikesco3
layout: post
name: nycdh-jekyll
link: https://github.com/Mikesco3
date: 2026-03-23 12:12:00 -0500
categories: [Tech, Windows, How-To]
tags: [windows, windows11, recovery, diskpart, reagentc, partition, gparted, minitool, sysadmin]
excerpt: How to create a proper Windows 11 recovery partition in the correct position on disk — EFI → MSR → Recovery → Windows — using GParted/MiniTool, diskpart, and reagentc.
keywords: windows 11, recovery partition, diskpart, reagentc, WinRE, partition layout
description: Step-by-step guide to shrinking the Windows partition, creating a 1.5 GB recovery partition in the correct disk position, and re-registering WinRE with reagentc.
pin: false
---
___

# Creating a Recovery Partition in Windows 11

___

> **TLDR**: Shrink the Windows partition using GParted or MiniTool, create a 1.5 GB recovery partition in the freed space using `diskpart`, then re-register WinRE using `reagentc` from within Windows.
{: .prompt-tip }

___

## Why bother?

By default, Windows sometimes places the recovery partition at the end of the disk, or omits a proper one entirely after certain upgrade or clone operations. The correct layout — **EFI → MSR → Recovery → Windows** — ensures that WinRE (Windows Recovery Environment) is accessible even if the Windows partition is damaged, and makes future partition resizing cleaner since the recovery partition won't be stranded at the end of the drive.

This guide assumes you have a **medium level of comfort** with disk tools and the command line. You will be working with partition operations — back up your data before proceeding.

---

## Overview of the Process

1. **Backup WinRE** - Backup the `Recovery` folder to a safe location.
2. **Disable WinRE** from within Windows so the recovery partition is not locked.
3. **Shrink the Windows partition** using GParted or MiniTool Partition Wizard (bootable USB), freeing 1.5 GB immediately *before* the Windows partition.
4. **Create and configure the recovery partition** using `diskpart`.
5. **Re-enable and re-register WinRE** using `reagentc`.
6. **Cleanup** - Remove the drive letter R.

---

### Target Partition Layout

| # | Type | Size |
|---|------|------|
| 1 | EFI System Partition | 500 MB |
| 2 | Microsoft Reserved (MSR) | 128 MB |
| 3 | **Recovery (WinRE)** | **1.5 GB** |
| 4 | Windows (C:) | Remainder |
| 5 | Data / Other | (if applicable) |

---
## The Process

### Step 1 — Backup the WinRE `Recovery` folder to a safe location.

####  from **`diskpart`** assign the Recovery Partition a drive letter 

> Assuming our Recovery partition was **partition 4** in Windows

``` batch
diskpart
    list disk
    select disk 0
    list partition
    sel part 4
    ass letter R
```

#### Browse to the partition and backup the Recovery Folder 
You may need to unhide system and hidden folders and backup the `Recovery` folder which will have the `Winre.wim` file within it to somewhere in the Windows Partition

### Step 2 — Disable WinRE (run as Administrator in Windows)

Before touching partitions, disable WinRE so Windows releases its lock on the recovery partition.

``` batch
reagentc /disable
```

Verify the status:

``` batch
reagentc /info
```

> WinRE status should show **Disabled**.
{: .prompt-info }


---

### Step 3 — Shrink the Windows Partition (GParted or MiniTool)

Boot from a **GParted Live USB** or **MiniTool Partition Wizard bootable USB**.

- Select the **Windows (C:)** partition.
- Shrink it by **1536 MB (1.5 GB)**, placing the freed space **before** the Windows partition (i.e., to the left in the visual layout).
- Create a new partition in the freed space with the following settings:
  - **Type**: Primary
  - **Size**: 1536 MB
  - **File System**: NTFS
  - **Boot Flag**: Off
- Apply the operation and reboot back into Windows.

> Make sure the freed unallocated space lands *between* the MSR partition and the Windows partition — not at the end of the disk.
{: .prompt-warning }

> While you are at it, it's much easier to create that partition from **Gparted** or **Minitool Parition Wizard**
{: .prompt-tip }

> But we will the commands below just in case

---

### Step 4 — Create the Recovery Partition (diskpart)

Open **Command Prompt as Administrator** and launch diskpart:

``` batch
diskpart
```

``` batch
list disk
select disk 0
list partition
```

> Note the partition number of your **Windows (C:)** partition and confirm unallocated space is visible before it.
{: .prompt-info }

Create and configure the recovery partition:

``` batch
diskpart
  create partition primary size=1536 offset=643072
  format quick fs=ntfs label="Recovery"
  set id=de94bba4-06d1-4d40-a16a-bfd50179d6ac
  gpt attributes=0x8000000000000001
  list partition
  ass letter=r
  exit
```

> The GUID `de94bba4-06d1-4d40-a16a-bfd50179d6ac` is the standard Windows Recovery partition type. 
{: .prompt-tip }

> The GPT attribute flags mark it as required and hidden.
>> The meaning of the GPT Attributes:
>> - `0x8000000000000000`: Hidden and No Drive Letter.
>> - `0x0000000000000001`: Required partition.
>> - `0x8000000000000001`: Combined: Required, hidden, and no drive letter. 
{: .prompt-tip }


#### Copy back the `Recovery` folder previously backed up
---

### Step 5 — Re-enable WinRE and Register the New Partition

Back in an **elevated Command Prompt**:

``` batch
reagentc /setreimage /path R:\Recovery\WindowsRE /target c:\Windows
reagentc /enable
reagentc /info
```

A successful result looks like this:

![reagentc /info showing WinRE Enabled](/assets/img/images/20260323-reagentc-success.png)

> WinRE status should show **Enabled** and the path should point to the new partition.
{: .prompt-info }

### Step 6 — Cleanup (Remove Drive Letter)

At this point we can remove the drive letter `R:` from showing up in the system.

``` batch
diskpart
  sel vol r
  remove letter=r
  exit
```

Prevent the drive mount from showing up in the system. 
> From an **elevated Command Prompt**:

``` batch
  mountvol /R
```

---

## Quick Reference — Commands Only

### Step 1 — Backup andDisable WinRE
``` batch
diskpart
    list disk
    select disk 0
    list partition
    sel part 4
    ass letter R
```
> Copy the Recovery folder from R: to a safe location.

``` batch
  reagentc /disable
```
> - Remove the Recovery partition from the system.
> - Resize the Windows partition to make room for the new Recovery partition.
> - Recreate the Recovery partition with the correct size and attributes.

### Step 4 — `diskpart` Create the Recovery Partition
``` batch
diskpart
  list disk
  select disk 0
  list partition
  create partition primary size=1536 offset=643072
  format quick fs=ntfs label="Recovery"
  set id=de94bba4-06d1-4d40-a16a-bfd50179d6ac
  gpt attributes=0x8000000000000001
  exit
```
> - Copy the Recovery folder from the backup location to the new Recovery partition.

### Step 5 — `reagentc` Re-enable WinRE
``` batch
reagentc /setreimage /path R:\Recovery\WindowsRE /target c:\Windows
reagentc /enable
reagentc /info
```
> - Reboot to test

### Step 6 — Cleanup
``` batch
diskpart
  sel vol r
  remove letter=r
  exit
mountvol /R
```

---

## Troubleshooting

- **reagentc /enable fails** — confirm the partition type GUID and GPT attributes were set correctly in diskpart. Run `list partition` and verify the recovery partition exists.
- **WinRE path points to wrong location** — use `reagentc /setreimage /path <path>` to manually point it to the correct partition if auto-detection fails.

---

> This is a personal reference guide. Always verify partition numbers on your specific disk before running destructive commands.
{: .prompt-warning }

---

© 2026 Michael Schmitz  
This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License.  
You are free to share and adapt this work with attribution. Derivative works must be distributed under the same license.
