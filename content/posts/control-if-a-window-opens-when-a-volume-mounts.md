---
title: 'Control if a window opens when a volume mounts'
date: Wed, 06 Nov 2013 14:04:24 +0000
draft: false
tags: ['OS X General']
---

This may not be as applicable anymore with Mavericks and the new createinstallmedia command, but it was relatively common to make a copy of the OS X Installer media onto a firewire or USB drive to allow for faster installs than from a CD or DVD. One issue with this was always that when the drive is mounted, the Finder would open a window to show the installer.  This was fine if there was only one volume on the drive, you probably wanted it open, but often a drive would have multiple installer partitions and a few bootable partitions and maybe a data partition as well. We don't want any of that to open automatically. The bless command is the way to control if a drive's window is opened automatically.  Replace _externaldrive_ with your volume's name. On a drive that doesn't auto open,

Code:

```
sudo bless -folder /Volumes/_externaldrive_ -openfolder /Volumes/_externaldrive_/
```

will make that folder open on mount. To remove that auto open, just

Code:

```
sudo bless -folder /Volumes/_externaldrive_
```

As long as you don't use any of the boot options, this won't change what your startup disk is. A thanks to the contributors to this forum post that prompted the search and solution: [http://hintsforums.macworld.com/showthread.php?t=99236](http://hintsforums.macworld.com/showthread.php?t=99236 "OS X Hints forums")