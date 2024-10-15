---
title: 'Swift mountshare command line tool'
date: Mon, 03 Jul 2023 09:52:00 +0000
draft: false
tags: ['macOS', 'Swift']
---

There have been many times over the years of my career where I have wanted to script mounting a file share for a user. There are several ways to make this happen from manually adding a file share to a user's Login Items, using AppleScript, and scripting the `mount` command. Eventually I found and settled on using a python script with the python objc bridge from an [old frogor/pudquick gist](https://gist.github.com/pudquick/1362a8908be01e23041d). This worked great with the built in python on macOS and uses the same bits of the OS that the Connect to Server… dialog does. I was able to [automatically mount shares based on AD group](https://sneakypockets.wordpress.com/2016/09/26/mounting-file-shares-based-on-ad-group-membership-using-enterprise-connect/) at a previous job using this script. Eventually Apple stopped including python in macOS, so I had to [install another one](https://github.com/macadmins/python). There were some modifications needed to make it work with python 3 (see the comments on the gist) and installing python everywhere for this one thing seemed like overkill. So I started working on a Swift implementation using [swift argument parser](https://swiftpackageindex.com/apple/swift-argument-parser/1.2.2/documentation/argumentparser). Since the python script calls into the system a lot of the pieces were already laid out and then I found a mosen [gist that laid out basically all the options and proper terms](https://gist.github.com/mosen/2ddf85824fbb5564aef527b60beb4669).

With those pieces I was able to create [mountshare](https://github.com/ehemmete/mountshare). This is a Swift command line tool that covers all the options I can find for mounting file shares from the command line. It ties into the system like Connect to Server… so things like storing file share authentication in the Keychain or using Kerberos authentication work as expected. There are options to make the share read only (‐‐readonly), not show up in the Finder (‐‐nobrowse), or connect as guest (‐‐guest). The help lays out all the options, but the most basic usage is `mountshare smb://server.domain.tld/share`.

```
OVERVIEW: Mount network shares the way "Connect to Server…" does

The default usage assumes the needed credentials are in the user's keychain or the server uses Kerberos.
Otherwise, a username and password can be passed in, or the system can prompt for authentication with a GUI window.
The various mounting and opening options are available as command line flags.
This command should be run as the user that is mounting the share.

USAGE: mountshare [<options>] <server-share-path>

ARGUMENTS:
  <server-share-path>     The full path to the share to mount i.e. "smb://server.domain.tld/share"

OPTIONS:
  -v, --verbose           Print success or error message to STDOUT
  -u, --username <username>
                          If necessary, specify a username to mount the share as.
  -p, --password <password>
                          Works with username to mount the share as a different user
  --local-path <local-path>
                          To change the local mountpoint.  The default is /Volumes/
  --nobrowse              No browsable data here (see <sys/mount.h>)
  --readonly              A read-only mount (see <sys/mount.h>)
  --allow-sub-mounts      Allow a mount from a dir beneath the share point
  --disable-soft-mount    Prevent mounting with "soft" failure semantics
  --mount-at-directory    Mount on the specified mountpath instead of below it
  --guest                 Login as a guest user
  --allow-loopback        Allow a loopback mount
  --allow-auth-dialog     The default GUI authentication window option, displays if needed
  --no-auth-dialog        Do not allow a GUI authentication window
  --force-auth-dialog     Force a GUI authentication window
  --version               Show the version.
  -h, --help              Show help information.
```

The code and installer are [available on my GitHub](https://github.com/ehemmete/mountshare). The installer is under the Releases area on the right. The package is signed and notarized and puts the tool in `/usr/local/bin/mountshare`.

A note about usage. The tool is meant to run as the user that is mounting the share. So if you are trying to automate mounting behind the scenes, take a look at running a script on connection via the [Kerberos-SSO extension](https://support.apple.com/guide/deployment/kerberos-single-sign-on-extension-depe6a1cda64/web), [Outset](https://github.com/macadmins/outset)'s login-every option, or [ScriptingOSX's post on running as a user](https://scriptingosx.com/2020/08/running-a-command-as-another-user/).