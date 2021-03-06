# Android dev usefuls
<br>

A list of things I learnt the hard way.
Many of these use [adb](https://developer.android.com/studio/command-line/adb.html). ([adbshell.com](http://adbshell.com/) has a good list of commands you can use with adb).  See [Connecting](#Connecting) for a bit more about connecting adb to emulators or real hardware.

These tips and tricks are based around Xamarin, C# and Visual Studio 2017.
I usually run the Android Emulators, rather than the MS ones as I prefer VirtualBox to Hyper-V, but it shouldn't matter here.



### Table of contents
1. [Emulator tips](#EmulatorTips)
    1. [Pasting text in a text box](#PastingText)
    2. [Debug Log Messages](#DebugLogMessages)
    3. [Controlling the emulator via telnet](#Telnet)
    4. [Setting Location](#Location)
    5. [Simple commands](#SimpleCommands)
    6. [Finding files](#FindingFiles)
    7. [Emulator Errors](#EmulatorErrors)
          1. [App stops at startup](#AppsStopsAtStartup)
          2. [Could not open avd_name.avd/cache.img](#CouldNotOpenCacheImg)
    8. [Emulators and proxies](#EmulatorsProxies)
          1. [Android in VirtualBox and ADB](#AndroidVirtualBoxADB)
    9. [Emulators and email](#EmulatorEmail)
    10. [Cameras](#Cameras)
          1. [Snapping existing digital images](#SnappingExistingDigitalImages)
    11. [Wifi problems](#Wifi)

2. [Accessing Sqlite databases](#Sqlite)
   1. [Cursor navigation](#SqliteCursor)
   2. [TimeStamps](#TimeStamps)
3. [Target hardware tips](#TargetHw)
    1. [Connecting](#Connecting)
         1. [Connecting to multiple devices/emulators](#MultipleDevices)
   2. [Viewing logs](#ViewingLogs)
<br>
<br>



## Emulator tips <a name="EmulatorTips"></a>
<br>
<br>

### Pasting text in a text box <a name="PastingText"></a>

(There should only be one device found with "adb devices")
* In the emulator, focus the target
* `adb shell input text 'Text to send'`

	From http://stackoverflow.com/questions/37039829/pasting-text-on-new-android-emulator/37043990#37043990

* Note: You can also send an SMS and copy/paste, but this is much easier.
<br>
<br>

### Debug Log Messages <a name="DebugLogMessages"></a>

It's useful to see log messages while running without too much effort.
Logging code is based on the [Android.Util.Log Class](https://developer.xamarin.com/api/type/Android.Util.Log/) and looks like this:

    Log.Info("IConnectionCallbacks", "OnConnected()");

In VS2017, display the Android Log with View => Other Windows => Device Log

Drop down the "Choose device" selector to choose your emulator.

There will be a shed-load of message, so you will need to filter with the search option.

I might use "IConnectionCallbacks" for the above message.

Another option is to prepend all your messages (or tags) with a unique string e.g. "###".

You can then search for that string and only your messages will appear.

From https://developer.xamarin.com/guides/android/deployment,_testing,_and_metrics/android_debug_log/

See also  [Viewing logs](#ViewingLogs)
<br>
<br>

### Controlling the emulator via telnet <a name="Telnet"></a>

Injecting GPS (location) coordinates, inbound calls and SMSs

* Find the emulator telnet port with adb

	**adb devices**

		List of devices attached
		emulator-5554   device

	The telnet port is the number, 5554 here.

* Use telnet or [MobaXterm](http://mobaxterm.mobatek.net/) or similar to telnet to 127.0.0.1:5554

* On first start, you'll see something like this
	```
	Android Console: Authentication required
	Android Console: type 'auth <auth_token>' to authenticate
	Android Console: you can find your <auth_token> in
	'C:\Users\alan\.emulator_console_auth_token'
	OK
	```

* Find the token as instructed and send it (in telnet)

	**auth xxxxxxxxxxxxxxxxx**
	
	You should get
	```
	Android Console: type 'help' for a list of commands
	OK
	```

* From here you can control various components

	| Command        | Function           | Notes  |
	| ------------- |:-------------:| -----:|
	| power capacity 80      | Set battery level |  |
	| geo fix longitude latitude [altitude]      | Set location      |   Optional altitude in meters |
	| network delay gprs | Simulate gprs network delays      |    etc |
	| gsm call <number> | Simulate an inbound call |  |
	| sms send 987654321 Hello fred | Simulate an inbound text |  |

	and lots of other stuff: see the link

	From https://developer.android.com/studio/run/emulator-console.html
<br>
<br>

### Setting Location <a name="Location"></a>

I couldn't get the telnet approach to setting location to work (although other things did e.g. battery level)

Another approach for GPS is to use the dynamic controls provided with the emulator.

For this to work, you need to set "Skin" to "Skin with dynamic hardware controls" when you create / edit your AVD (Android Virtual Device).

Then, when the emulator is running, click the three dots (more) at the bottom of the dynamic control pane and select Location.

Enter the values you want and click SEND.

From https://developer.android.com/studio/run/emulator.html#extended

<br>
<br>

### Simple commands <a name="SimpleCommands"></a>

You can append simple commands straight on to adb:

* List files in a folder

	**adb ls -la /data/user/0**

* List out a file

	**adb shell cat /data/misc/net/rt_tables**

* etc etc

<br>
<br>

### Finding files <a name="FindingFiles"></a>

* adb to get a shell on the emulator

	**adb shell**

* Normal Linux /Android command to search from root (/)

	__find / -name *.txt__

	If you start at root (/) you will get a lot of "permission denied" etc, you should be able to use `grep -v` to filter those out.

<br>
<br>

###  Emulator Errors <a name="EmulatorErrors"></a>


#### App stops at startup <a name="AppsStopsAtStartup"></a>

Before your app starts up properly, you get a popup on the emulator:
```
"Unfortunately, DarkWeather.Droid has stopped"
```
or similar.

The xamarin Diagnostics output window in Visual Studio shows something like this:
```
[E:]:  Emulator name lookup failed for emulator emulator-5537'
System.AggregateException: One or more errors occurred. ---> System.AggregateException: One or more errors occurred. ---> System.NullReferenceException: Object reference not set to an instance of an object.
   at Mono.AndroidTools.AndroidEmulatorConsole.<>c__DisplayClass7_0.<RunCommand>b__0() in C:\d\lanes\4985\0841c2aa\source\xamarinvs\External\md-addins\MonoDevelop.MonoDroid\external\androidtools\Mono.AndroidTools\AndroidEmulatorConsole.cs:line 85
```
The error message repeats in various guises.

In my case, the problem  was  that  I'd left VS  in Release mode following a deployment.

As  soon as I set it back to Debug, [voil�](https://en.wiktionary.org/wiki/voil%C3%A0).
<br>
<br>

#### qemu-system-i386.exe: Could not open avd_name.avd/cache.img <a name="CouldNotOpenCacheImg"></a>

Typically because another process is accessing the emulator.

Do you have another Visual  Studio running?

Kill the emulator and if that doesn't work, hunt down and kill qemu-system-i386.exe in task manager.

From https://stackoverflow.com/questions/35701174/could-not-open-avd-name-avd-cache-img/40789371#40789371
<br>
<br>
<br>


###  Emulators and proxies <a name="EmulatorsProxies"></a>

It seems hard to proxy everything out of Android and Android in an emulator.

For pen-testing, as distinct from Android dev, I've taken to running [Android_x86](http://www.android-x86.org/) in [Virtual Box](https://www.virtualbox.org/). 
Specifically, I run android-x86_64-6.0-r3 as this is pre-Nougat and allows (easy) certificate additions.

To push all traffic to a proxy e.g. [Burp](https://portswigger.net/burp) listening on 192.168.1.4 port 8080
```
adb shell settings put global http_proxy 192.168.1.4:8080
adb shell settings put global global_http_proxy_host 192.168.1.4
adb shell settings put global global_http_proxy_port 8080
adb shell settings put global https_proxy 192.168.1.4:8080
adb shell settings put global global_https_proxy_host 192.168.1.4
adb shell settings put global global_https_proxy_port 8080
```

192.168.1.4 is the address of the host machine running Burp and 8080 is the port Burp is listening on.

List global settings with **adb shell settings list global**

Remove with
```
adb shell settings delete global http_proxy
adb shell settings delete global global_http_proxy_host
adb shell settings delete global global_http_proxy_port
adb shell settings delete global https_proxy
adb shell settings delete global global_https_proxy_host
adb shell settings delete global global_https_proxy_port
```


You will probably need to confgure your setup so that ADB talks to Android inside the emulator

####  Android in VirtualBox and ADB <a name="AndroidVirtualBoxADB"></a>

This is the way I found to get ADB to talk to the emulated Android:

* Determine Android IP address as follows:
* < Alt >F1 to get a console
* **ifconfig** and read the eth0 address (ipaddr)
* < Alt >F7 to get back to the UI
* On the host: **adb connect ipaddr**
* **adb devices** will show you if it is connected

###  Emulators and email <a name="EmulatorEmail"></a>

It can be useful to have email on the emulator.

Use the supplied (simple) client with a gmail address.

[This link](https://www.e-mailsettings.com/android/gmail-mail-setup) explains how to configure the account.

###  Cameras <a name="Cameras"></a>

####  Snapping existing digital images <a name="SnappingExistingDigitalImages"></a>

You may need to feed the camera with an existing digital image, e.g. of a QR code an app is looking for.
<br>The back camera (facing away) must be set to "Emulated" (and you cannot have both front and back caneras set to Emulated)
<br>In the extended controls of the emulator, select Camera.
<br>If you can't see the Camera option, most likely the back camera is not set to Emulated.
<br>The Camera page is titled "Virtual Scene Images".
<br>From https://developers.google.com/ar/develop/java/emulator#add_augmented_images_to_the_scene
<br>Once the camera is working, you will be able to pan around a virtual scene with a table and a wall.
<br>I suggest loading a picture for the wall as the table-top is hard to see at a sensible angle.
<br>If none of the above makes sense, just press on!
<br>I suggest giving the wall image a size of 1 metre.
<br>When you start the camera app, or another app tries to use the camera, you will find yourself in a room looking at a window.
<br>Hold down the <Alt> key and move the mouse to look left, right, up and down.
<br>Hold down the <Alt> key and tap the keys to move:
<br>Move left or right  -  Hold Alt + press A or D
<br>Move down or up	- Hold Alt + press Q or E
<br>Move forward or back  -  Hold Alt + press W or S

<br>To get to the wall and table images, you will have to turn around 180 degrees, enter the room on your right and move to the wall.  The table gets in the way.
<br> From https://developers.google.com/ar/develop/java/emulator#control_the_virtual_scene
<br><br>Alternatively, try an image-to-camera app such as this:
https://github.com/GhostFlying/image-to-camera


###  Wifi problems<a name="Wifi"></a>
If the emulator wifi (AndroidWifi) is saying "No internet" and everything else is working, go back to the hardware adapter that is providing the network connection to the emulator.
Look at the adapter properties, specifically "Internet Protocol Version 4 (TCP/IPv4)".
<br> If manual DNS settings are enabled, the DNS servers need to be the Google ones: 8.8.8.8 and 8.8.4.4.
<br> From https://stackoverflow.com/questions/50670547/android-emulator-wifi-connected-with-no-internet/52765004#52765004
<br> I'm guessing this is Google trying to make sure your DNS doesn't get poisoned

## Accessing Sqlite databases <a name="Sqlite"></a>

Of course, you can find the db, copy it to the host and use a tool to inspect and change it.

This way is a bit quicker.

* Start adb (see [Connecting](#Connecting)) and make sure Android Emulator is running

	**adb devices**

* Set root

	**adb root**

* Run the shell

	**adb shell**

* Locate sqlite db file (you might have to look around: I know where mine is)
	
	**ls -la /data/data/DarkWeather.Droid/files**

* Run Sqlite against the db file

	**sqlite3 data/data/DarkWeather.Droid/files/DarkWeather.db**

* Show tables

	**.tables**

* Show a table structure (actually it's the create script)

	**.schema HourDataPointDto**

* View table contents

	**select * from HourDataPointDto;**  // Don't forget the semicolon

* Show top 5 rows

	**select * from HourDataPointDto limit 5;**

* Show latest five rows

	**select * from HourDataPointDto order by Time desc limit 5;**

* Exit Sqlite: any of

	**[CTL]D**

	**.exit**

	**.quit**

* Exit adb shell

	**exit**


	From http://alvinalexander.com/android/android-command-line-shell-show-sqlite-tables-adb-sqlite3

    More help with Sqlite commands [here](https://www.sqlite.org/cli.html).
<br>
<br>

### Cursor navigation <a name="SqliteCursor"></a>

You'll probably notice that as soon as you run Sqlite against the db file

	sqlite3 data/data/DarkWeather.Droid/files/DarkWeather.db

You get to the

    sqlite>

cursor and you can no longer use the cursor keys here, you just get this sort of stuff: \^[[A^[[B^[[C.
These are the control codes for up-arrow, down-arrow etc

If you need to use the cursor keys a lot, don't go into the sqlite interpeter at all, but just run sqlite from the adb cursor:

    sqlite3 data/data/DarkWeather.Droid/files/DarkWeather.db "select count(*) from KeyValuePairDto"

The sql commands need to be completely enclosed in double quotes

For some reason, the terminal width is limited to about 78 chars wide (let me know if you find a fix)

So, you can reduce the command size by cd-ing to the db subdirectory in adb shell:

    cd data/data/DarkWeather.Droid/files

Then, just reference the db without the path:

    sqlite3 DarkWeather.db "select count(*) from KeyValuePairDto"

From http://stackoverflow.com/questions/15747564/cursor-keys-not-working-when-using-sqlite3-from-adb-shell/30282915#30282915
<br>
<br>

### TimeStamps <a name="TimeStamps"></a>

You can store your timestamps in various ways including strings (ugh!). Some of the more efficient ways are

#### Int
If you want second precision -  Unix Time, the number of seconds since 1970-01-01 00:00:00 UTC

To show a comprehensible value at the command line, use this

`select datetime(IntTimeColumn, 'unixepoch') from TableWithTimes;`

#### BigInt
If you want subsecond precision - hundreds of nanoseconds from the begining of Common Age (I think).

To show a comprehensible value at the command line, use this

`select datetime(BigIntTimeColumn/10000000 - 62135596800, 'unixepoch') from TableWithTimes;`

From https://stackoverflow.com/questions/1342448/convert-sqlite-bigint-to-date/41607123#41607123
<br>
<br>


## Target hardware tips <a name="TargetHw"></a>
<br>

### Connecting <a name="Connecting"></a>
Connect the hardware (phone / tablet) to your PC (sorry, I'm not an apple fan) with a normal USB cable.
Locate your adb executable.

Mine is in `C:\Program Files (x86)\Android\android-sdk\platform-tools`

Look for devices:

**adb devices**

You should get something like this

	List of devices attached
	TA38172UIS      unauthorized

If you see "unauthorised", there are several possibilities, the simplest is probably that a dialog has popped up on your hardware and you need to allow USB debugging. Running the "adb devices" command again should now give

	List of devices attached
	TA38172UIS      device

You're now ready to communicate using adb
<br>
<br>

#### Connecting to multiple devices/emulators <a name="MultipleDevices"></a>
Sometimes you may have more than one emulator active.

Get devices ids:

**adb devices**

	List of devices attached
	emulator-5554   device
	emulator-5556   device

Then use
<br>
**adb -s emulator-5554 commands...**

OR, if you have just one emulated device and one real device connected over USB

	adb -d  ... // directs the adb command to the USB device
	adb -e  ... // directs the adb command to the emulator
This returns an error if there is more than one USB device/emulator connected

From https://stackoverflow.com/questions/14654718/how-to-use-adb-shell-when-multiple-devices-are-connected-fails-with-error-mor



### Viewing logs <a name="ViewingLogs"></a>

Look at all the logs

**adb logcat**

**adb logcat -v time**  // includes a timestamp (host machine if on an emulator), but it's a strange layout and I don't know how to format it.

This produces a lot of stuff. There are several ways to filter it down.  In all cases, I can leave `adb logcat` running and the display will update as new messages come in.

* Use the DOS `find` command

    **adb logcat | find /I DarkWeather**

    Here I am filtering just log lines that contain the name of my project. (The /I is in case I forgot to capitalise the name correctly)

* Use specific logcat commands to limit the data

    **adb logcat --help**

    shows the possibilities, also [this](https://developer.android.com/studio/command-line/logcat.html).

* See also [Debug Log Messages](#DebugLogMessages)
