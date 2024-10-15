---
title: 'Fun with AppleScript and Notes'
date: Fri, 20 Dec 2019 16:05:53 +0000
draft: false
tags: ['AppleScript', 'Shell Script']
---

I have pretty large collection of movies ripped from my DVDs and a few Blu-Rays stored on a file server. Occasionally when I want to watch something old that we have, I'd rather borrow a Blu-Ray from our library to see it in better quality, but it is hard to remember what movies are DVD quality and which are Blu-Ray. The post will show how I collect that information and made it sync to my iPhone. To collect the resolution information I am using `mdls -name kMDItemPixelHeight $movie` and `mdls -name kMDItemPixelWidth $movie`. I initially thought about writing this to a static webpage, but didn't want to go to the trouble of setting all that up.  I had visions of launchdaemons and shell scripts to automate it all, but that felt like too much work for a trivial want.  So I started thinking about other ways to get data available on the go and eventually remembered Notes.  That lead to a bit of research, including the always useful [MacOSXAutomation](https://www.macosxautomation.com/applescript/notes/02.html) and of course a [StackExchange](https://apple.stackexchange.com/questions/295569/can-an-applescript-write-to-a-notes-app-document) post. The fun part is using AppleScript to write to a note in Notes.app which then is synced through iCloud automatically. Notes.app notes use html for formatting which can be pretty powerful. In my case, I just need each entry on its own line. In AppleScript Editor I saved the following as a script and now double click it when I want to update the note.```
**set** ss **to** "cd /Volumes/External/Media/Movies
for movie in \*.m\*; do
 height=$(mdls -name kMDItemPixelHeight \\"$movie\\" | awk '{print $NF}')
 width=$(mdls -name kMDItemPixelWidth \\"$movie\\" | awk '{print $NF}')
 echo \\"<div>${movie}, ${height}, ${width}</div>\\"
done"
**set** notebody **to** **do shell script** ss
**tell** _application_ "Notes"
** activate**
** tell** default account **to** **tell** _folder_ "Notes"
**  set** body **of** _note_ "Movie Resolutions" **to** notebody
** end** **tell**
**end** **tell**
```To make it simpler for myself, I created the note and named it first, but you can do that through AppleScript.  Since I am going to be running this periodically as the collection changes, I didn't want to deal with naming conflicts. This certainly isn't ground breaking, but I liked the idea of using AppleScript to bridge between shell and my iPhone.  Now that data is available where ever I am. Notes:

*   *   My Notes setup is very simple.  One account synced through iCloud.  There are account options available in the Notes AppleScript library if you need them.
    *   I can still automate running the AppleScript periodically with a LaunchAgent and the `open` command.  I'd probably save the script as an Application for this use.