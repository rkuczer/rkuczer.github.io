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
The next step was to have Linux properly setup prior to adding everything for minecraft, this included.
1. `sudo apt update && sudo apt upgrade`: Update & Upgrades
2. `sudo apt install curl/wget`: Install necessary packages 
