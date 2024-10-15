---
title: 'Using a Swift app to get rid of lots of AppleScript dialogs'
date: Mon, 15 Apr 2019 13:17:32 +0000
draft: false
tags: ['OS X General', 'Shell Script', 'Swift']
---

There was recently a discussion in #bash in the [MacAdmin Slack](https://macadmins.org) about better ways to handle multiple dialogs for gathering info from users or techs.  In my company's provisioning process we want to collect a number of things to be picked up by our inventory system (BigFix).  This started with a set of AppleScript Display Dialogs.

![AppleScriptPopUps.2019-04-13 12_40_20.gif](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/applescriptpopups.2019-04-13-12_40_20.gif)

This is not a great experience and gets worse as we try to make it less error prone (do you see the mistake?). To make this better for our techs, I wrote a relatively simple Swift application.  I didn't/don't have much Swift experience (just playing around in Playgrounds, a few simple tutorials), so I can't say this is the "right" way to do this, but it works for me/our process.

I will try to outline the process I used to make a app that has several input boxes and some pop up menus for entering data.  Some of the data is validated and then it is written to files on the Macs filesystem to be read later by our configuration system (Jamf Pro) and our inventory system.  At the end I'll show a way to incorporate this into a shell script as well.

![Screen Shot 2019-04-12 at 3.44.25 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-12-at-3.44.25-pm.png)

I'll be using Xcode 10.2 to recreate this app for this post, but I don't think there is much that changed for this level of coding in recent versions. The project and all the code is available on [GitHub](https://github.com/ehemmete/buildinfo).  There is also the example [AppleScript](https://github.com/ehemmete/buildinfo/blob/master/BuildInfo_AppleScript.sh) from above.

Open Xcode and choose a Create a new Xcode project.![Screen Shot 2019-04-12 at 12.30.37 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-12-at-12.30.37-pm.png)

Select macOS at the top and Cocoa App in the main area.![Screen Shot 2019-04-12 at 12.31.47 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-12-at-12.31.47-pm.png)

Give it a name and select the appropriate developer account.  Confirm Swift is the language. Uncheck Use Storyboards.  This will NOT be a document based application and I haven't created tests for it.  Save the project somewhere.

![Screen Shot 2019-04-12 at 12.44.28 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-12-at-12.44.28-pm.png)

Before we get to far, there is one thing we need to change at the project level.  If it isn't, select the project at the top of the structure on the left.  In the center area, select Capabilites and turn off App Sandbox.  We will be writing to other parts of the file system than sandboxing allows. ![Screen Shot 2019-04-12 at 8.36.43 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-12-at-8.36.43-pm.png)

In the main window select MainMenu.xib on the left.

If you don't see a window in the center, click the window icon in the vertical bar between the file list and the main area.

Add the labels, text fields, secure text field (shows bullets), and pop up buttons from the Library toolbar item.  Try to use the auto layout guides as much as possible.  If you have lots of input to collect, you can group them with a box.  If you need more choices in a pop up menu, use the menu item choice in the Library.![Screen Shot 2019-04-12 at 7.34.26 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-12-at-7.34.26-pm-4.png)

Add a Reset and Submit button.

Delete any Menus you don't want.

This can be kind of frustrating at first.  I won't go into a lot of detail, but the Editor menu and the constraints pop up in the lower right can be helpful in getting things to look good/Mac like.  I like to use Reset to Suggested Constraints.![Screen Shot 2019-04-12 at 2.14.47 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-12-at-2.14.47-pm.png)

One last thing, with the Submit button selected, go to the attributes inspector on the right and find the Key Equivalent field.  Click in there and press the Return key.  Now users can press return to start the Submit process.![Screen Shot 2019-04-12 at 8.41.27 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-12-at-8.41.27-pm-1.png)

Now to get to some code.  We need to add outlets and actions for our menus, fields, and buttons.  Click the Assistant Editor button and open the AppDelegate.swift file next to our MainMenu.xib.![Screen Shot 2019-04-12 at 8.44.02 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-12-at-8.44.02-pm-1.png)

Holding the control key, grab each text field and pop up menu and drag to the assistant editor under the `@IBOutlet week var window: NSWindow!` line.  In the pop, up add a good name for the fields.

![CreatingOutletsAndActions.2019-04-13 10_03_22.gif](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/creatingoutletsandactions.2019-04-13-10_03_22.gif)

Do this for all the elements in the window, including the Reset button (we will get to that later).

![Screen Shot 2019-04-13 at 10.04.19 AM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-13-at-10.04.19-am.png)

Then do the same for the buttons, but change the Connection type to Action and the Type to NSButton.

![Screen Shot 2019-04-12 at 8.50.51 PM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-12-at-8.50.51-pm.png)

Now we are going to focus on the code side of things.

In Swift our functions can go pretty much anwhere in the file. I added the following at the end of my AppDelegate.swift to write a string to a file:

> ```
> //Write a string to a file
> func writeToFile(file: String, data: String) {
>     let fileURL = URL(fileURLWithPath: file)
>     do {
>         try data.write(to: fileURL, atomically: true, encoding: .utf8)
>     } catch {
>         print("Writing \\(data) to \\(fileURL) failed")
>     }
> }
> 
> ```

Now when the data is filled in and the the Submit button is pushed, we can write this data to the file system.  In the submitButton action, we can add the steps to read the data from the window.  Text Field have a .stringValue property and Pop Up Button menus have a .titleOfSelectedItem property to get what we want.  I don't fully understand why, but I think, because it would be possible for a menu to have nothing selected, we need to treat the selected menu items a bit different.  I'm sure there is a better way, but by ending .titleOfSelectedItem with a !, the warnings in Xcode go away :)

> ```
> let techID = techIDField.stringValue
> let assignedUser = assignedUserField.stringValue
> let topGunValue = topGunPopUp.titleOfSelectedItem!
> let computerName = computerNameField.stringValue
> let businessUnit = businessUnitPopUp.titleOfSelectedItem!
> let countryCode = countryCodePopUp.titleOfSelectedItem!
> 
> ```

Then we use the writeToFile function to write this all out.  Pass the path to the file you want to create and the string you want to write there.  I created a variable for the directory path so as to not have it repeated over and over. `let tagRoot = "/Users/Shared/OrgName"` Then we can quit the application with exit(0).  You can use other numbers if you want to be able to know from where the application was quit.

> ```
> writeToFile(file: "\\(tagRoot)/BuildTech.txt", data: techID)
> writeToFile(file: "\\(tagRoot)/AssignedUser.txt", data: assignedUser)
> writeToFile(file: "\\(tagRoot)/TopGun.txt", data: topGunValue)
> writeToFile(file: "\\(tagRoot)/ComputerName.txt", data: computerName)
> writeToFile(file: "\\(tagRoot)/BusinessUnit.txt", data: businessUnit)
> writeToFile(file: "\\(tagRoot)/CountryCode.txt", data: countryCode)
> ```

The reset action will set all our fields and menus to some default. Basically we do the reverse of the submit action.  Instead of reading the values, we will set them.  We can use the same .stringValue property for the text fields, but need to use .selectItem(withTitle: String) property for the menu buttons.

> ```
> @IBAction func resetButton(\_ sender: NSButton) {
>         techIDField.stringValue = ""
>         assignedUserField.stringValue = ""
>         topGunPopUp.selectItem(withTitle: "None")
>         computerNameField.stringValue = ""
>         businessUnitPopUp.selectItem(withTitle: "Corporate")
>         countryCodePopUp.selectItem(withTitle: "US")
>     }
> ```

At this point, we can build our application and most of it works.  Reset should work and the menus/text fields are set-able.  But depending where you placed your tagRoot, you might not be able to submit.  If the folder doesn't exist, it won't work.  So let's discuss some setup for the app.  There is a built in function for `applicationDidFinishLaunching`, where we can do some setup.  In here we can make the tagRoot folder if necessary and initialize our pop ups to the most common values.  I'm going to take advantage of the reset function and run that as part of the initialization.  To call our resetButton action, we need to pass it the "sender".  This is why we created the Reset button outlet earlier. Finally, lets capture when this build process was started. Now if we run our application, the default choices should be selected.

> ```
> func applicationDidFinishLaunching(\_ aNotification: Notification) {
>        //Create build started
>         let currentDate = Date()
>         let dateFormatter = DateFormatter()
>         dateFormatter.dateFormat = "MM/dd/YY HH:mm"
>         let buildStarted = dateFormatter.string(from: currentDate)
>         writeToFile(file: "\\(tagRoot)/BuildStarted.txt", data: buildStarted) 
> 
>         //Create folder for tags
>         let fileManager = FileManager.default
>         if  !fileManager.fileExists(atPath: tagRoot) {
>             do {
>                 try fileManager.createDirectory(atPath: tagRoot, withIntermediateDirectories: true, attributes: nil)
>             } catch {
>                 print("Cannot create folder at \\(tagRoot)")
>             }
>         }
>         //Set our defaults
>         resetButton(resetButton)
>     }
> ```

We should now have a working application that can collect data and write it to files.

![Screen Shot 2019-04-15 at 8.16.57 AM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-15-at-8.16.57-am.png)

Another things we can do is create some error checking and data validation.  Maybe we want to only allow submission if all the fields are filled out.  We can add a check in the submitButton action to only proceed if they aren't empty.  The || means OR, so if the techIDField is empty OR the assigned UserField is empty OR ...

> ```
> if techIDField.stringValue.isEmpty || assignedUserField.stringValue.isEmpty || computerNameField.stringValue.isEmpty {
>             return
>         }
> ```

The return will drop the program out of this function and not continue until all the fields are full.  We can let the user know a few ways.  Maybe a label with instructions.  Or in the full version of this, there are other windows with all the needed messages and their .visible property controls when they are shown.  To add this, back in the MainMenu.xib, add a second window (from the Library) and put the message as a label. The window title is set in the attributes inspector on the right.  The Empty Fields window uses a Multi-line label.  Don't forget an OK button to close it.  In the assistant editor, drag an action for the OK button and an outlet for the whole window.

![Screen Shot 2019-04-13 at 10.51.17 AM.png](https://sneakypockets.wordpress.com/wp-content/uploads/2019/04/screen-shot-2019-04-13-at-10.51.17-am.png)

Now we can add `emptyFieldsWindow.setIsVisible(true)` right before we return if some fields are empty. When the OK button is pressed, we reverse that with `emptyFieldsWindow.setIsVisible(false)`.  If you run your application now, the Empty Fields window will be visible at startup.  We can fix this in the attributes area for it, or in the initialization function where we set the menu items.

> ```
> if techIDField.stringValue.isEmpty || assignedUserField.stringValue.isEmpty || computerNameField.stringValue.isEmpty {
>             emptyFieldsWindow.setIsVisible(true)
>             return
>         }
> 
> ```

> ```
> @IBAction func confirmEmptyFieldsButton(\_ sender: NSButton) {
>         emptyFieldsWindow.setIsVisible(false)
>     }
> ```

Similarly we can do some error checking on the given computer name or the user names. In the live version of this, there is some swift code to run ldapsearch.  This may be a separate post at some point, but the basics of it are:

> ```
> let userLookup = Process()
> let userPipe = Pipe()
>         
> userLookup.launchPath = "/usr/bin/ldapsearch"
> userLookup.arguments = \["-LLL", "-Q", "-H", "ldap://my.dc.domain.tld", "-b", "dc=my,dc=domain,dc=tld", "(&(objectCategory=Person)(objectClass=User)(sAMAccountName=\\(username)))", "sAMAccountName", "2>/dev/null"\]
> userLookup.standardOutput = userPipe
> userLookup.launch()
> userLookup.waitUntilExit()
>         
> let handle = userPipe.fileHandleForReading
> let data = handle.readDataToEndOfFile()
> let ldapOutput = String(data: data, encoding: String.Encoding.utf8)
> 
> ```

For our sample project we can make sure the computer name matches some criteria. Let's say it needs to be 8-15 characters and start with OrgMac. In our submit action, we can test computerName and return if it doesn't match our needs with the computer name field ready to recieve new input.  The && means AND, so if the computername length greater than or equal to 8 AND less than or equal to 15 that would be true, but then the ! means NOT.  So if the length is not in the right range, OR it doesn't start with the right prefix, execution will stop.  We could/should also pop up another message window explaining the problem.

> ```
> if !(computerName.count >= 8 && computerName.count <= 15) || !computerName.starts(with: "OrgMac") {
>             computerNameField.becomeFirstResponder()
>             return
>         }
> 
> ```

As an exercise to the reader, how could you pre-fill the ComputerNameField with the serial number of the computer? This link has some info that will help: https://gist.github.com/leogdion/77f6143ecf793e1ba381917d4b3b286c

Hopefully this has had enough to get started on a simple Swift application of your own. There are lots of parts here that can be reused/recombined.  In my production version, there are 6 error checking pop ups and an about window.  All names are validated against AD and group membership is checked to make sure the tech has rights to build a Mac.  There are 11 fields/pop ups that the tech fills in.

There are many good guides available for getting started with Swift.  A few are linked below:

https://docs.swift.org/swift-book/GuidedTour/GuidedTour.html#//apple\_ref/doc/uid/TP40014097-CH2-ID1

https://developer.apple.com/swift/resources/

To finish up, here is a way to incorporate this into a larger script.  This example will be a [shell script](https://github.com/ehemmete/buildinfo/blob/master/BuildInfo_InUse.sh), but the same could be done in Python or other language.  The `open` command can be told to wait for a process to exit with the `-W` option.  We can trigger our application from a shell script and then wait for it to close.  Then read in the values saved and use them later in the script.

> ```
> #!/bin/bash
> 
> # Run the application and wait for it to finish
> /usr/bin/open -W -a /Applications/BuildInfo.app
> 
> # Collect the output from the .app for use in this script
> tagRoot="/Users/Shared/OrgName"
> 
> BuildStarted=$(cat ${tagRoot}/BuildStarted.txt)
> BuildTech=$(cat ${tagRoot}/BuildTech.txt)
> AssignedUser=$(cat ${tagRoot}/AssignedUser.txt)
> TopGunStatus=$(cat ${tagRoot}/TopGun.txt)
> BusinessUnit=$(cat ${tagRoot}/BusinessUnit.txt)
> CountryCode=$(cat ${tagRoot}/CountryCode.txt)
> 
> # Do the rest of the setup using these values
> .
> .
> .
> 
> ```