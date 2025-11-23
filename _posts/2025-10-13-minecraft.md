---
title: Configuring a baremetal Linux Server to Play with Friends
image: INSERTIMAGE.
date: 2025-10-13 12:00:00 
categories: [linux, sysadmin]
tags: [linux,sysadmin]
---

# Objective

After diving into different subjects in the realm of computers, networking, system administration and security you get exposed to a lot all at once. To be able to retain the information effectively without statically studying all day it seems the best way to learn is to split your time studying & building. Furthermore, it is even better if you can build something you yourself can use but if other people in your life can it's even better. 

Hence my new addition to the portfolio of projects is configuring a Modded Minecraft server for me and my friends to play on. This covers everything from adjusting the BIOS of a Dell tower, installing Ubuntu, hardening Linux OS and the network to only allow clients what they need. To sum it up its IT, 'sys administration', and security all in one!

## Hardware
Thankfully I had a spare Dell Optiplex 7010 tower lying around, this was perfect as it was not too beefy but also not a peashooter. For reference it was new when Windows 7 came out.

#### Specifications
* 320 GB STA
* Intel Core i7-3770S
* 8 Gb RAM (DDR3)

## Step 1: Changing the OS
I wanted the max amount of resources available for the Minecraft server since I was on limited hardware so I chose **Ubuntu**.  I chose my favorite USB stick then used Rufus to format it and put the latest Ubuntu ISO on it. I then put it into the Optiplex while it was off, turned it on, spammed **F12** to get it into the BIOS and finally set the boot order to be the USB first. 

I then installed Ubuntu and followed the prompts without changing anything. 

## Step 2: Linux Hygiene
The next step was to have Linux properly setup prior to adding everything for minecraft, this included using the following commands while directly connected to the Linux box. *The commands assume you have sudo access, security setup will be in the next section*. 
1. `apt update && apt upgrade`: Update & Upgrades
2. `apt install curl/wget`: Install necessary packages 
3. `apt install screen`: Used to have the server run properly when not logged onto the server.
4. `/opt, mkdir minecraft, cd minecraft`: Navigate to the `/opt` directory then create a called minecraft then navigate to it. Once in the minecraft directory, download the forge installer for 1.12.2 and then copy it to this dir.
5. `java -jar forge-1.12.2-14.23.5.2860-installer.jar --installServer`: Run the forge installer, creates a second file called **forge-1.12.2-14.23.5.2860.jar**.
6. `java -Xms1024M -Xmx2000M -jar /opt/minecraft/forge-1.12.2-14.23.5.2860.jar nogui`: Start the minecraft server with a minimum 1gb of memory with a maximum of 2gb allocated, also run the server in nogui for faster performace. Knowing how xms, xms, jvm works in conjunction with [managing JVM's heaps](https://sematext.com/glossary/jvm-heap/) is useful here.
7. Accept the eula by navigating to the `eula.txt` file with `nano` then set `eula=true`.
8. *Optional:* `nano` to the `server.properties` file then set the `level-seed` to a preset seed if that is desired.
9. Create a script to start the minecraft server if that is desired, I did it. It should be a `.sh` file with the following contents. After creating it make sure to use `chmod` to make it executable.
```bash
    #!/bin/bash
    cd /opt/minecraft/ && java -Xms1024M -Xmx2000M -jar /opt/minecraft/forge-1.12.2-14.23.5.2860.jar nogui
```

10. Before starting the server use the `screen` command to start a screen session. Then start the server with the script, after that is complete use the shortcut `ctrl + a, d` to detach from the session allowing you to do other configuration while the server is running.