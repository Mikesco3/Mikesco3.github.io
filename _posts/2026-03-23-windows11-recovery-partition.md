---
title: Creating a Recovery Partition in Windows 11
author: mikesco3
layout: post
name: nycdh-jekyll
link: https://github.com/Mikesco3
date: 2026-03-23 10:55:00 -0500
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

1. **Disable WinRE** from within Windows so the recovery partition is not locked.
2. **Shrink the Windows partition** using GParted or MiniTool Partition Wizard (bootable USB), freeing 1.5 GB immediately *before* the Windows partition.
3. **Create and configure the recovery partition** using `diskpart`.
4. **Re-enable and re-register WinRE** using `reagentc`.

---

## Target Partition Layout

| # | Type | Size |
|---|------|------|
| 1 | EFI System Partition | 500 MB |
| 2 | Microsoft Reserved (MSR) | 128 MB |
| 3 | **Recovery (WinRE)** | **1.5 GB** |
| 4 | Windows (C:) | Remainder |
| 5 | Data / Other | (if applicable) |

---

## Step 1 — Backup WinRE (run as Administrator in Windows)

Before touching partitions, disable WinRE so Windows releases its lock on the recovery partition.

```cmd
reagentc /disable
```

Verify the status:

```cmd
reagentc /info
```

> WinRE status should show **Disabled**.
{: .prompt-info }

```cmd
reagentc /info
```


###  from **`diskpart`** assign the Recovery Partition a drive letter 

```cmd
diskpart
```

> Assuming our Recovery partition was **partition 4** in Windows


```diskpart
    list disk
    select disk 0
    list partition
    sel part 4
    ass letter R
```

### Browse to the partition and backup the Recovery Folder 
 You may need to unhide system and hidden folders and backup the `Recovery` folder which will have the winre.wim file within it to somewhere in the Windows Partition

---

## Step 2 — Shrink the Windows Partition (GParted or MiniTool)

Boot from a **GParted Live USB** or **MiniTool Partition Wizard bootable USB**.

- Select the **Windows (C:)** partition.
- Shrink it by **1536 MB (1.5 GB)**, placing the freed space **before** the Windows partition (i.e., to the left in the visual layout).
- Apply the operation and reboot back into Windows.

> Make sure the freed unallocated space lands *between* the MSR partition and the Windows partition — not at the end of the disk.
{: .prompt-warning }

> While you are at it, it's much easier to create that partition from **Gparted** or **Minitool Parition Wizard**
{: .prompt-tip }

 _but we will the commands below just in case_
---

## Step 3 — Create the Recovery Partition (diskpart)

Open **Command Prompt as Administrator** and launch diskpart:

```cmd
diskpart
```

```diskpart
list disk
select disk 0
list partition
```

> Note the partition number of your **Windows (C:)** partition and confirm unallocated space is visible before it.
{: .prompt-info }

Create and configure the recovery partition:

```diskpart
create partition primary size=1536 offset=643072
format quick fs=ntfs label="Recovery"
set id=de94bba4-06d1-4d40-a16a-bfd50179d6ac
gpt attributes=0x8000000000000001
list partition
ass letter=r
exit
```

> The GUID `de94bba4-06d1-4d40-a16a-bfd50179d6ac` is the standard Windows Recovery partition type. The GPT attribute flags mark it as required and hidden.
{: .prompt-tip }

### Copy back the `Recovery` folder previously backed up
---

## Step 4 — Re-enable WinRE and Register the New Partition

Back in an **elevated Command Prompt**:

```cmd
reagentc /enable
reagentc /info
```

A successful result looks like this:

![reagentc /info showing WinRE Enabled](/assets/img/images/20260323-reagentc-success.png)

> WinRE status should show **Enabled** and the path should point to the new partition.
{: .prompt-info }

---

## Quick Reference — Commands Only

### Step 1 — Disable WinRE
```cmd
  reagentc /disable
```

### Step 3 — `diskpart` Create the Recovery Partition
```cmd
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
### Step 4 — `reagentc` Re-enable WinRE
```cmd
reagentc /setreimage /path R:\Recovery\WindowsRE /target c:\Windows
reagentc /enable
reagentc /info
```

---

## Troubleshooting

- **reagentc /enable fails** — confirm the partition type GUID and GPT attributes were set correctly in diskpart. Run `list partition` and verify the recovery partition exists.
- **WinRE path points to wrong location** — use `reagentc /setreimage /path <path>` to manually point it to the correct partition if auto-detection fails.
- **Unallocated space landed at the wrong position** — GParted and MiniTool both allow you to drag which side of the partition the freed space goes to; make sure it goes to the *left* (before Windows, not after).

---

> This is a personal reference guide. Always verify partition numbers on your specific disk before running destructive commands.
{: .prompt-warning }

---

© 2026 Michael Schmitz  
This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License.  
You are free to share and adapt this work with attribution. Derivative works must be distributed under the same license.
