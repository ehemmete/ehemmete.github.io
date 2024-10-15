---
title: 'Automating making bootable macOS installer dmgs'
date: Fri, 21 Sep 2018 20:04:56 +0000
draft: false
tags: ['Uncategorized']
---

macOS Mojave should be released on Mon (9/24). At my employer, we will not be ready to support it yet as we are waiting for a couple of vendors to update their products. So we will need to be able to build High Sierra Macs without relying on Internet Recovery for erasing and reinstalling macOS. To make things more confusing, macOS High Sierra doesn't have a unified installer for all models. Most use 17G65, but the 2018 MacBook Pros need 17G2208. Both can be downloaded today (before Mojave comes out) with [installinstallmacos.py](https://github.com/munki/macadmin-scripts/blob/master/installinstallmacos.py).  Instead of manually making USB drives with createinstallmedia for both and then making dmgs to distribute to our deskside team across the country, I wanted to automate this process. I wrote a script that takes the output of installinstallmacos.py as input, creates an empty writable disk image, uses the createinstallmedia command to make the new disk image a bootable macOS installer, converts the installer dmg to compressed, and scans it for restore.  I have tested it with both High Sierra and Mojave installers. See [makeBootableInstaller](https://github.com/ehemmete/makeBootableInstaller) for the script and details. The way to use it is to first use installinstallmacos.py to download the version and build you need.  Then run:

> sudo makeBootableInstaller.sh ~/Install\_macOS\_10.13.6-17G2208.dmg

You will see output like:

> Creating installer dmg for macOS 10.13.6 17G2208. Mounting target sparseimage. Mounting source dmg. Creating macOS install media. Erasing Disk: 0%... 10%... 20%... 30%...100%... Converting sparseimage to dmg. Scanning dmg for restore Copied installer dmg to ~ and cleaned up.

And you will have Install\_macOS\_High\_Sierra\_10.13.6\_17G2208\_Installer.dmg in your home folder.  This dmg is ready to be restored to an external drive.