---
title: Aomei Backupper Tutorial 
author:
name: mikesco3
link: https://github.com/Mikesco3
date: 2023-01-17 11:28:00 -0500
categories: [Blogging,Tutorial]
tags: [linux, xdotool, guide, tutorial]
# pin: false
---

# Automatically refresh the browser by sending an F5 keystroke at regular intervals

## Why, What for...
I had a project where we ran a linux machine that displays a website as a dashboard on a large monitor,  <br>
however I needed to issue a refresh to the browser at regular intervals in case,  <br>
for example in case the site changed or the internet went out... <br>

In our example we are running the Brave Browser on Linux Mint 21.1 cinnamon


# Setup
## Install xdotool
1. I installed a tool called xdotool that is supposed to allow to send keystrokes to a window: <br>
  ` apt install --yes xdotool`

## Write a script 
2. I Wrote this script so that it would cycle whether it runs Firefox, chrome or brave and send F5 to refresh every 2 minutes. <br>
  We called the script   `refreshBrowser.sh` and placed in a folder we created on the users home called `/bin/` <br>
  here is the scrpt: <br>
```bash
#!/usr/bin/bash

browsers=("chrome" "brave" "firefox")

/usr/bin/sleep 30s &&

echo This script auto refreshes the browser every 2 minutes
echo You can stop the script with ctrl+c
echo or by just closing this window.

while true; do
    for browser in "${browsers[@]}"
    do
        browser_window_id=$(xdotool search --onlyvisible --name $browser)
        if [ -n "$browser_window_id" ]; then
            xdotool windowactivate $browser_window_id
            xdotool key F5
            sleep 2m
        fi
    done
done

```

## Schedule the script to run on startup
3. Schedule `gnome-terminal` to run the script on Startup: <br>
*The nice thing is that if you want the command to stop, you can just close the terminal screen or press CTRL+C* <br>
   1. Open the "Startup Applications" utility by searching for it in the menu or by running the command "cinnamon-settings startup" in the terminal.
	2. Click on the "+" button to add a new startup application.
	3. Fill in the Name and Comment fields with a name and a brief description of the command you want to run.
	4. In the Command field, enter the following command
	   `/usr/bin/gnome-terminal  --command="~/bin/refreshBrowser.sh"`	   
	5.  Increase the delay before startup by around 3 seconds
	6. Click on the Add button to add the command to the startup applications list.
	7. The next time you log in, the gnome-terminal will open and run the command automatically.
	   *And the nice thing is that if you want the command to stop, you can just close the terminal screen or press CTRL+C*

# Sources
- https://unix.stackexchange.com/questions/87831/how-to-send-keystrokes-f5-from-terminal-to-a-gui-program
- The invaluable help of [ChatGPT](https://chat.openai.com/)