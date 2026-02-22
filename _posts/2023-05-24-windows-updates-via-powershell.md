---
title: Windows Updates via PowerShell 
author: mikesco3
name: nycdh-jekyll
link: https://github.com/Mikesco3
date: 2023-05-24 18:11:00 -0500
categories: [Blogging,Tutorial]
tags: [windows, powershell, windows updates, guide, tutorial]
# pin: false
---

# Why, What for...
There are times where it is more convenient to run windows updates via PowerShell. In fact, there are times where it is the only way to do updates, specially when the pesky service is malfunctioning <br>

# Setup
1. Open PowerShell as admin <br>
 (right-click on the start menu and select "Windows PowerShell (Admin)") <br>
 or search for "PowerShell", then right-click on it and select "Run as Administrator" <br>

2. Set the execution policy to Remotely Signed <br>
 (paste the following command and press __Y__ or __A__ to agree and press __ENTER__) <br>
 ``` powershell
 Set-ExecutionPolicy RemoteSigned
 ```

# Getting and Intalling Updates
3. Install the PSWindowsUpdate module 
``` powershell
Install-Module PSWindowsUpdate
```
4. To get the list of pending Windows Updates
``` powershell
Get-WindowsUpdate
```
5. Finally to Install the pending Updates
``` powershell
Install-WindowsUpdate
```

Here you can press __A__ to agree to all or __Y__ or __ENTER__ to agree to the individual ones <br>
   ![PowerShell](/assets/img/images/Pasted_Image_20240524182930.png)

___

## Troubleshooting
This powershell command has helped me fix windows updates:
``` powershell
reset-wucomponents
```
