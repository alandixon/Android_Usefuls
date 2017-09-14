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
    5. [Finding files](#FindingFiles)
2. [Accessing Sqlite databases](#Sqlite)
   1. [Cursor navigation](#SqliteCursor)
3. [Target hardware tips](#TargetHw)
   1. [Connecting](#Connecting)
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

### Finding files <a name="FindingFiles"></a>

* adb to get a shell on the emulator

	**adb shell**

* Normal Linux /Android command to search from root (/)

	__find / -name *.txt__

	If you start at root (/) you will get a lot of "permission denied" etc, you should be able to use `grep -v` to filter those out.
<br>
<br>
<br>

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

### Viewing logs <a name="ViewingLogs"></a>

Look at all the logs

**adb logcat**

This produces a lot of stuff. There are several ways to filter it down.  In all cases, I can leave `adb logcat` running and the display will update as new messages come in.

* Use the DOS `find` command

    **adb logcat | find /I DarkWeather**

    Here I am filtering just log lines that contain the name of my project. (The /I is in case I forgot to capitalise the name correctly)

* Use specific logcat commands to limit the data

    **adb logcat --help**

    shows the possibilities, also [this](https://developer.android.com/studio/command-line/logcat.html).
