---
title: 'New Folder in List View'
date: Mon, 26 Mar 2018 13:07:22 +0000
draft: false
tags: ['Automator', 'OS X General']
---

I often use List View in the Finder for the directory where I organize all my packages.  There doesn't seem to be any way to create a folder within the highlighted folder below.![Screen Shot 2018-03-23 at 4.48.33 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2018/03/screen-shot-2018-03-23-at-4-48-33-pm.png) In this post I will show how we can create that option.  Right-click/Control-click doesn't have a New Folder option in this view.  The File menu/Action menu/keyboard shortcut for New Folder makes a the new folder in the folder that is at the root of the window.  That doesn't work for me since there are so many folders in this directory that I lose my place. We can use AppleScript to make a new folder and Automator to make it a Service that will be available in the right-click menu.  Open Automator and choose to make a New Document and Choose to make it a Service.![Screen Shot 2018-03-23 at 4.56.03 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2018/03/screen-shot-2018-03-23-at-4-56-03-pm.png) Then search on the left for Run.  Drag a Run AppleScript action into the workflow on the right.  Then change the dropdowns to "Service receives selected folders in Finder"![Screen Shot 2018-03-23 at 4.58.57 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2018/03/screen-shot-2018-03-23-at-4-58-57-pm.png) The AppleScript block will be passed the folder we have selected in the Finder as `input`.  We can use that to know where to make our new folder.  We could just use:```
tell application "Finder"
    make new folder at input
end tell
```to make a new folder, but then we have to rename it.  Since we are already using AppleScript, I decided to prompt for the name up front.  So my final AppleScript looks like```
on run {input, parameters}
    set folderNameDialog to display dialog "Enter the folder name:" default answer "" buttons {"OK"} default button "OK"
    set folderName to text returned of folderNameDialog
    tell application "Finder"
        make new folder at input with properties {name:folderName}
    end tell
    return input
end run
```Now when this is run, I can enter a folder name first: ![Screen Shot 2018-03-23 at 5.08.29 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2018/03/screen-shot-2018-03-23-at-5-08-29-pm.png) Save the Automator workflow as New Folder.  Automator puts the service in ~/Library/Services/ automatically, so it will only work for you unless you move it. Now the final workflow is:

1.  Select the soon to be parent folder
2.  Right click -> Services -> New Folder
3.  Enter a name and click OK (or press Enter)
4.  Folder is created with the given name![NewFolder.gif](https://sneakypockets.wordpress.com/wp-content/uploads/2018/03/newfolder.gif)