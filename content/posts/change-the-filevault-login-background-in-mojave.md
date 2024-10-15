---
title: 'Change the FileVault login background in Mojave'
date: Thu, 30 May 2019 20:36:00 +0000
draft: false
tags: ['Uncategorized']
---

There are sites on the internet about changing the Mojave login window background, but nothing I found for when FileVault is enabled.  I think I found a solution. For the login window, the "trick" is to swap out /Library/Desktop Pictures/Mojave.heic with your image.  It must be the same name and format.  Preview on Mojave can help with converting your image to .heic format. Then to get FileVault to use it, we update the preboot environment/partition with \[code\]diskutil apfs updatePreboot /\[/code\] Restart and you should see your custom background. Apple may very well "fix" this with a/every software update and set it back, but if you really want to brand that screen it is possible for now.