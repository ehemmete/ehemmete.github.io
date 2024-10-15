---
title: 'Setting the Screen Saver to a static image'
date: Fri, 02 Feb 2018 20:28:49 +0000
draft: false
tags: ['Casper', 'High Sierra', 'Packaging', 'Shell Script', 'Sierra']
---

On our Windows systems we have a static image as our screen saver and I wanted to match that on our Macs.  There are several options in the Screen Saver settings of System Preferences.  I needed to figure out how to set a default across all our Macs with a script.![Screen Shot 2018-02-02 at 1.55.51 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2018/02/screen-shot-2018-02-02-at-1-55-51-pm.png)  We wanted a single image statically on the screen.  By making a folder with one image in it, we can use the slideshow options to show our single image. After setting things up the way I wanted, I looked around to see where the settings were stored.  You can use something like [FSMonitor](https://fsmonitor.com) to watch what files are edited when you change things in System Preferences.  These settings are per user and per device, so they end up in ~/Library/ByHost/com.apple.screensaver.plist, ~/Library/ByHost/com.apple.ScreenSaver.iLifeSlideShows.plist, and ~/Library/ByHost/com.apple.ScreenSaverPhotoChooser.plist.  If you select other options, you might not need all of these files. To work with this files, use `defaults -currentHost read [preference-domain]`. First look at com.apple.screensaver.plist:```
$ defaults -currentHost read com.apple.screensaver
{
    PrefsVersion = 100;
    idleTime = 1800;
    moduleDict =     {
        moduleName = iLifeSlideshows;
        path = "/System/Library/Frameworks/ScreenSaver.framework/Resources/iLifeSlideshows.saver";
        type = 0;
    };
}
```Then ~/Library/ByHost/com.apple.ScreenSaver.iLifeSlideShows.plist:```
defaults -currentHost read com.apple.ScreenSaver.iLifeSlideShows
{
    styleKey = Classic;
}
```Then finally, ~/Library/ByHost/com.apple.ScreenSaverPhotoChooser.plist:```
defaults -currentHost read com.apple.ScreenSaverPhotoChooser
{
    CustomFolderDict =     {
        identifier = "/Library/Screen Savers/CompanyName";
        name = CompanyName;
    };
    LastViewedPhotoPath = "";
    SelectedFolderPath = "/Library/Screen Savers/CompanyName";
    SelectedSource = 4;
}
```To set these, we use a very similar command: `defaults -currentHost write [preference-domain] [key] [value]`. After some testing I've found we just need:```
#!/bin/sh
defaults -currentHost write com.apple.screensaver moduleDict -dict moduleName iLifeSlideshows path "/System/Library/Frameworks/ScreenSaver.framework/Resources/iLifeSlideshows.saver" type 0
defaults -currentHost write com.apple.ScreenSaver.iLifeSlideShows styleKey Classic
defaults -currentHost write com.apple.ScreenSaverPhotoChooser SelectedFolderPath "/Library/Screen Savers/CompanyName"
```Now to deploy this, I have 2 pieces.

1.  I need a package that puts my image in a known folder ("/Library/Screen Savers/CompanyName")
2.  A way to run this script for the user. There are several ways to do this; LaunchAgent, Self Service, or what I use is [Outset](https://github.com/chilcote/outset). Outset could set this every login, but I don't want to enforce it, so I use the Login-Once option.  The important thing to remember is that this is a per user setting, so if you try to set these as root, you will need to do some extra work with permissions.

These could be in one package or separate.  Since I implemented Outset after I had already created the image package, I have separate packages for the image and the script.  There are several options for creating your packages, but I really like [Packages](http://s.sudre.free.fr/Software/Packages/about.html).