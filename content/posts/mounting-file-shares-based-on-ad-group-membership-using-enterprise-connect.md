---
title: 'Mounting File Shares Based on AD Group Membership using Enterprise Connect'
date: Mon, 26 Sep 2016 14:27:25 +0000
draft: false
tags: ['OS X General', 'Shell Script']
---

\[edit\] Apple changed something in macOS 10.13 High Sierra. The mount\_share() function in the below python needs to be changed to set the open\_options and mount\_options to 'None'. This can be done like```
def mount\_share(share\_path):
    # Mounts a share at /Volumes, returns the mount point or raises an error
    sh\_url = CoreFoundation.CFURLCreateWithString(None, share\_path, None)
    # Set UI to reduced interaction
    #open\_options  = {NetFS.kNAUIOptionKey: NetFS.kNAUIOptionNoUI}
    # Allow mounting sub-directories of root shares
    #mount\_options = {NetFS.kNetFSAllowSubMountsKey: True}
    # Mount!
    result, output = NetFS.NetFSMountURLSync(sh\_url, None, None, None, None, None, None)
    # Check if it worked
    if result != 0:
         raise Exception('Error mounting url "%s": %s' % (share\_path, output))
    # Return the mountpath
    return str(output\[0\])
```\[/edit\] In a [previous post](https://sneakypockets.wordpress.com/2016/09/22/using-ldapsearch-to-get-ad-data/), I discussed using `ldapsearch` to look up user data from AD.  In this post we will use the user's memberOf attribute to mount the appropriate file share. Some background on my use case for this.  The company I work for has ~15,000 Windows computers in use bound to AD.  When a user logs in, a GPO runs a batch file hosted on the domain controller's file share.  The batch file is basically a large case statement```
if in group A; then
    mount shares X and Y
if in group B; then
    mount share Z
```I wanted to provide our Mac users with a similar experience.  Read how below the break. Our Macs are not bound to Active Directory and we have historically had password confusion issues with network vs local passwords and keychain sync issues.  To help this we have setup Apple's excellent [Enterprise Connect](https://jamfnation.jamfsoftware.com/discussion.html?id=17757) to help with that. When a Mac user is logged in and is on a network that can connect to our domain controllers (internal, VPN, etc), Enterprise Connect automatically retrieves a Kerberos Ticket Granting Ticket for the user and, optionally, confirms that the local password is in sync with the user's AD password and prompts the user to fix things if necessary.  Enterprise Connect can also mount file shares including the user's home folder.  This is handled on a per user basis or by profile.  This works great if the user wants to mount a share they use, but it doesn't scale well across many locations and OUs. One of the other benefits of Enterprise Connect is that it can run scripts based on a couple of triggers including a successful connection (i.e. the Mac is on the right network and a valid TGT has been granted).  Note that Enterprise Connect can only run shell scripts this way. The script to run can be configured in the preference file or via profile with the key/value pair:```
connectionCompletedScriptPath
 /Library/Scripts/LogonFileShareMounter.wrapper.sh
```This script is stored locally on the client system and is what calls the script that will do the actual mounting.  Before that though, we first make sure that we have the current version of the mounting script by checking an embedded serial number (similar to how DNS primary/secondaries work).  The master copy of the mounting script is hosted on the same share as the Windows batch file.  This way it can be updated whenever the Windows version is.  As long as the serial number is updated, the Mac clients will get the updated version next time they successfully connect with Enterprise Connect. https://gist.github.com/ehemmete/618c96dc364d8a40723e30692617d2a8 The script starts by creating a directory if necessary and then mounting the file share where the master script is kept.  The `mount_smbfs` line authenticates to the share with the Kerberos TGT  from Enterprise Connect.  Then as long as the master script can be found, the local serial is compared to the remote version.  If the remote is newer, it is copied in place of the local copy.  Then the share is unmounted and the local copy of the script is run. https://gist.github.com/ehemmete/2f893815bd03d96d89855feb4e9b7237 This script starts with the serial number that is compared against the master copy.  Then the part between the ### lines comes straight from MacAdmin Slack member and frequent gist'er Froger/pudquick/MikeyMikey.  His [gist](https://gist.github.com/pudquick/1362a8908be01e23041d) is referenced in the script and provides a python function for mounting network shares easily and in such as way that the system treats them the same as the Connect to Server dialog box. Then we get the user's local account name, which in my environment is the same as their Active Directory name, and use `ldapsearch` to find their group memberships.  Next we need to clean up the output.  At the end we have an array of names of the groups this user is a member of.  Now we start a case-like statement similar to the Window counter part.```
if 'Group1' in groups:
    mount\_share('smb://server.my.domain.com/share')
    mount\_share('smb://server2.my.domain.com/share2')
if 'Group2' in groups:
    mount\_share('smb://server3.my.domain.com/share')
etc...
```On the user end, they just have connected to our network and see their file shares mount automatically.  The servers show up in the Finder's sidebar, but that isn't necessarily intuitive to them, so I also use Casper to fill their home folder with a Finder preference file to show Connected Servers on the Desktop.