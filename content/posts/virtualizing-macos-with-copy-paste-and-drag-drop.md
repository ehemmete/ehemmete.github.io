---
title: 'Virtualizing macOS with copy/paste and drag &amp; drop'
date: Wed, 29 Nov 2023 01:51:20 +0000
draft: false
tags: ['AppleScript', 'macOS', 'Sonoma', 'virtualization']
---

With Apple adding their [Virtualization](https://developer.apple.com/documentation/virtualization) [Framework](https://developer.apple.com/documentation/virtualization), there are several options for virtualizing macOS (and others) on our M# based Macs. I use VMs for connecting to some of our customer networks that require full-tunnel VPN. One common complaint is that these don't offer copy/paste or drag and drop from the host to or from the guest. A workaround for this is to use Apple's Screen Sharing or Remote Desktop application to connect to the VM. Because of this, I like to use [Tart](https://github.com/cirruslabs/tart) as my VM software because it offers a no gui option. I have an AppleScript start it up in a particular way.

To install Tart I used `brew install cirruslabs/cli/tart`. You can also download releases from their Github. Then to create the VM, `tart create <name> --from-ipsw <path>` will give you a VM with a ~44GB drive based on the ipsw file you pass in. You can get various ipsw files from Apple via the nice links on [Mr. Macintosh's](https://mrmacintosh.com/apple-silicon-m1-full-macos-restore-ipsw-firmware-files-database/) site. For example I created mine with `tart create FullTunnel --from-ipsw ~/Downloads/UniversalMac_13.6_22G120_Restore.ipsw`.

Then for the first launch, you can run `tart run FullTunnel` and go through the initial setup. Create an account and then in System Settings -> General -> Sharing, enable Screen Sharing or Remove Management. If you aren't already using Remote Desktop, just stick with Screen Sharing. Then in General -> About, set the Name. Also go to System Settings -> Network -> Ethernet and set the IP to be static. I use whatever IP the VM got via DHCP. In my experience they are in the 192.168.65.0/24 subnet.

Next we will configure our app to connect to this VM. If you are using Screen Sharing, click the plus sign in the toolbar, if you are using ARD, select Add by Address in the File menu. Authenticate to the new VM and say to Remember Password if using Screen Sharing. Once connected shut down the VM.

To set things up for next time, open /System/Applications/Utilities/Script Editor.app and create a new script. My ARD script is below. Change all 4 references to 'FullTunnel' to whatever you called the VM:

```
set ready to do shell script "/opt/homebrew/bin/tart list | awk '/FullTunnel/ {print $4}'"
if ready is not "running" then
	do shell script "/opt/homebrew/bin/tart run --no-graphics FullTunnel  > /dev/null 2>&1 &"
	repeat until ready = "running"
		set ready to do shell script "/opt/homebrew/bin/tart list | awk '/FullTunnel/ {print $4}'"
	end repeat
	delay 10

end if

tell application "Remote Desktop"
	activate
	control computer "FullTunnel"
 --This is the computer name
end tell
```

This first checks if the VM is already running with `tart list` and seeing if it is "running". If not, then it starts it with `tart run --no-graphics FullTunnel` so the VM doesn't open a GUI window. Then the script waits while the VM boots up and then tells Remote Desktop to connect to that remote system.

For Screen Sharing the script is slightly different. Again set the name instead of 'FullTunnel' in 3 places and set the ip in the GetURL line for Screen Sharing :

```
set ready to do shell script "/opt/homebrew/bin/tart list | awk '/FullTunnel/ {print $4}'"
if ready is not "running" then

	do shell script "/opt/homebrew/bin/tart run --no-graphics FullTunnel  > /dev/null 2>&1 &"
	repeat until ready = "running"

		set ready to do shell script "/opt/homebrew/bin/tart list | awk '/FullTunnel/ {print $4}'"
	end repeat
	delay 10
end if

tell application "Screen Sharing"

	activate

	GetURL "vnc://192.168.65.4"

end tell
```

In either case save the script as an Application somewhere useful and now you have a quick way to launch the VM and connect via the remote application. I generally trigger it with spotlight.