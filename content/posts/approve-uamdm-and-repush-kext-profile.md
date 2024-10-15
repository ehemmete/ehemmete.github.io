---
title: 'Approve UAMDM and repush KEXT Profile'
date: Wed, 23 May 2018 15:52:57 +0000
draft: false
tags: ['Uncategorized']
---

I recently came across a way to have our JAMF JSS resend a failed KEXT whitelist policy triggered from the client end ([Retry a failed Profile from a client](https://sneakypockets.wordpress.com/2018/05/21/retry-a-failed-profile-from-a-client/)).  At that point I wasn't sure how I wanted to deploy it during our provisioning process.  I now have a plan to prompt for UAMDM approval and then automatically resend the KEXT profile.

> ```
> #!/bin/bash#!/bin/bash
> while ! $(profiles status -type enrollment | grep -q "User Approved"); do 
>  open /System/Library/PreferencePanes/Profiles.prefPane
>  sleep 10
> done
> curl -sku "$apiuser":"$apipass" -H "Content-Type: application/xml" -d "<os\_x\_configuration\_profile><general><redeploy\_on\_update>Newly Assigned</redeploy\_on\_update></general></os\_x\_configuration\_profile>" "$jssurl"/JSSResource/osxconfigurationprofiles/id/$id -X PUT
> ```

You will need to supply the username, password, url, and the id number of your Kext profile in your script. I install this script as a login-once action for [Outset](https://github.com/chilcote/outset).  When the user or tech provisioning the Mac signs in after JAMF Imaging is complete, they will see warnings for unapproved kernel extensions.![Screen Shot 2018-03-27 at 9.42.45 AM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2018/05/screen-shot-2018-03-27-at-9-42-45-am.png) But once the script runs, System Preferences will open to the Profiles pane and keep opening if the user closes it.  Once they approve the MDM Profile, the script triggers the JSS to resend the KEXT profile, which some applications notice immediately.![Screen Shot 2018-03-27 at 9.44.22 AM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2018/05/screen-shot-2018-03-27-at-9-44-22-am.png) I may add a JAMF helper dialog explaining what to do and will probably add an OS version check as the profiles status line only works in 10.13.4 and above. Thanks to Rich Trouton for [a method to check for UAMDM](https://derflounder.wordpress.com/2018/03/30/detecting-user-approved-mdm-using-the-profiles-command-line-tool-on-macos-10-13-4/).