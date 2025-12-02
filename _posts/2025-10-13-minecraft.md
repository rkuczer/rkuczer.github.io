---
title: Configuring a Bare-Metal Linux Server to Play with Friends
image: dellopti.jpg
date: 2025-10-13 12:00:00 
categories: [linux, sysadmin]
tags: [linux,sysadmin]
---

# Objective 

After diving into different subjects in the realm of computers, networking, system administration, and security, you get exposed to a lot all at once. To retain the information effectively without staying to study all day, it seems the best way to learn is to split your time between studying and building. Furthermore, it is even better if you can build something you yourself can use but if other people in your life can use it too, it's even better. 

Hence, my latest addition to my portfolio of projects is configuring a modded Minecraft server for me and my friends to play on. This covers everything from adjusting the BIOS of a Dell tower, installing Ubuntu, hardening the Linux OS and the network to only allow users what they need. To sum it up, it's IT, sys administration, and security all in one!

*This article assumes you have sudo access on the server and are knowledgeable with Minecraft and Linux.*

## Hardware
Thankfully, I had a spare Dell OptiPlex 7010 tower lying around; this was perfect as it was not too beefy but also not a peashooter. For reference, it was new when Windows 7 came out. I wanted to self host this server as it provided:
* Maximum control
* The ability to work with the CLI
* It's cheaper than a VPS
* Learning the full process of hosting from start to finish

### Server Specifications
* 320 GB SATA
* Intel Core i7-3770S
* 8 GB RAM (DDR3)

## Step 1: Changing the OS
I wanted the maximum amount of resources available for the Minecraft server since I was on limited hardware, so I chose **Ubuntu**. I chose Ubuntu because Windows inherently has more 'bloat' that can eat away at the total amount of compute power for other processes that matter more. 

So, I chose my favorite USB stick, then used [Rufus](https://rufus.ie/en/) to format it and put the latest Ubuntu ISO file on it. I then put it into the OptiPlex while it was off, turned it on, spammed **F12** to get it into the BIOS, and set the boot order to be the USB first. 

I then installed Ubuntu and followed the prompts without changing anything out of the ordinary.

## Step 2: Linux Hygiene
The next step was to properly set up Linux before adding everything for Minecraft; this included using the following commands while directly connected to the Linux box. *The commands assume you have sudo access; security setup will be in the next section*. 
1. `apt update && apt upgrade`: Update & upgrades
2. `apt install curl wget`: Install necessary packages 
3. `apt install screen`: Used to have the server run properly when not logged onto the server.
4. `cd /opt && mkdir minecraft && cd minecraft`: Navigate to the `/opt` directory, then create a folder called minecraft, then navigate to it. Once in the minecraft directory, download the Forge installer for 1.12.2 and then copy it to this dir. This version of Forge is included as it is not extremely old but has the most compatibility with everyone's favorite mods. *Note: If using SSH for initial configuration, use something like WinSCP or SCP to transfer the files to your Linux server*.
5. `java -jar forge-1.12.2-14.23.5.2860-installer.jar --installServer`: Run the Forge installer; it creates a second file called **forge-1.12.2-14.23.5.2860.jar**.
6. `java -Xms1024M -Xmx2000M -jar /opt/minecraft/forge-1.12.2-14.23.5.2860.jar nogui`: Start the Minecraft server with a minimum of 1 GB of memory and a maximum of 2 GB allocated; also run the server in nogui for faster performance. Knowing how Xms, Xmx, and JVM work in conjunction with [managing JVM's heaps](https://sematext.com/glossary/jvm-heap/) is useful here.
7. Accept the EULA by navigating to the `eula.txt` file with `nano`, then set `eula=true`.
8. *Optional:* `nano` to the `server.properties` file, then set the `level-seed` to a preset seed if that is desired.
9. Create a script to start the Minecraft server if desired to make launching easier I did. It should be a `.sh` file with the following contents. After creating it, make sure to use `chmod` to make it executable; this was the initial script:
```bash
#!/bin/bash
cd /opt/minecraft/ && java -Xms1024M -Xmx2000M -jar /opt/minecraft/forge-1.12.2-14.23.5.2860.jar nogui
```

10. Before starting the server, use the `screen` command followed by `Ctrl + A, then C` to start a screen session. Then start the server with the script; after that is complete, use the shortcut `Ctrl + A, then D` to detach from the session, allowing you to do other configuration while the server is running. To reattach to the screen session whenever you need, use the `screen -r` command; learn more on the screen command [here](https://www.geeksforgeeks.org/linux-unix/screen-command-in-linux-with-examples/).

At this point, users on your local network can connect to the server and play on it as long as they are also running Minecraft + Forge 1.12.2. However, there are no mods currently, and we need to make the server available over the internet (securely) still.

## Step 3. Mods
Download your mods from one of the reputable mod library sites that are supported with Forge; I used **CurseForge**. Users on the client side will put their mods in this folder: `C:\Users\YOURUSERHERE\AppData\Roaming\.minecraft\mods`

## Step 4. Security
Now that the server is running with everything installed on the Minecraft side, it is time to harden the server. Each section below will go over the specific security addition.

### SSH Security

The following section applies changes to SSH configuration, choosing a **non-standard port** and only allowing **key-based authentication**.

* I chose a non-standard SSH port other than the default of `22` to avoid low-hanging-fruit-style attacks, where the port will be scanned either manually or automatically to identify potential system vulnerabilities.

* Key-based authentication was used to provide stronger authentication and to remove worries about using a password that could be leaked.

To do this, navigate to the SSH server config file at `/etc/ssh/sshd_config`, then make the following changes to the file.

```conf
Port 2222
PasswordAuthentication no
ChallengeResponseAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```
After this, restart the SSH service with `sudo systemctl restart ssh` and generate an SSH key pair with the command: `ssh-keygen -t rsa -b 4096 -C "your-email@example.com"`. I then transferred the key to my other local machine. Test connection status with the command `ssh -p 2222 youruser@your-server-ip`.

### Network Security
* I first set up port forwarding on my router to allow users to access my internal server from my public IP on port `25565`. Reference your router's documentation to complete this.

* I then blocked all other non-necessary ports on the server itself with the following commands:
  ```bash
  sudo ufw default deny incoming
  sudo ufw allow 25565/tcp
  sudo ufw enable
  ```

* To allow SSH only from the local network, ensure UFW is enabled, then use the command: `sudo ufw allow from <local_subnet> to any port 2222 proto tcp`. In my case, the local subnet was `192.168....`

### Whitelisting
Since my server was visible on port `25565`, I then used whitelisting to only allow certain players to be able to join. To do this, first set `white-list=true` in the `server.properties` file. Then restart the server, then use the command `whitelist add USERNAME` for each player you want to be able to join.

### Backups
I regularly backed up the server files to a safe location (another computer) to avoid backlash from my friends if I lost all their progress. This backup can be used to restore the entire world itself.
```bash
# Log in to your server
cd /opt
tar -zcvf minecraft_backup.tar.gz minecraft
```
#### Automatic
To make the backups automatic on a weekly basis, create the following script:
```bash
# What to backup. Name of minecraft folder in /opt
backup_files="minecraft"

# Specify which directory to backup to.
# Warning: Minecraft worlds can get fairly large, so choose your backup destination accordingly.
dest="/home/username/minecraftbackups"

# Create backup archive filename.
day=$(date +%A)
archive_file="$day-$backup_files-.tar.gz"

# Backup the files using tar.
cd /opt && tar zcvf $dest/$archive_file $backup_files
```
Make sure to make the file executable with `chmod` and test the script before creating a scheduled task. Then create a task with the cron scheduler; use this line at the end of a root crontab: `02 2 * * 0 /opt/scripts/mcbackup.sh &> /dev/null` (runs weekly on Sundays).

## Server Improvements
* I recreated my script to have a catch for when it inevitably crashed at some point, whether it was from a player overusing resources or generic crashes.

```bash
while true; do 
	echo "Starting Minecraft server..." 
	java -Xms2G -Xmx4G -jar /opt/minecraft/forge-1.12.2-14.23.5.2860.jar nogui  # Scaled for 8GB RAM; adjust as needed
	echo "Minecraft server crashed or stopped. Restarting in 10 seconds..." 
	sleep 10 
done
```

## Summary of Experience
Constructing this Minecraft server turned a simple evening of gaming with friends into a rewarding dive into system administration. Though built on humble hardware, the foundational skills I gained from Linux fundamentals and robust security measures to efficient server management readily scale to enterprise-level challenges. It's a reminder that the most enduring knowledge emerges from hands-on projects that marry enjoyment with expertise.