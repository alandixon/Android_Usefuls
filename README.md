# Android dev usefuls
<br>

A list of things I learnt the hard way.
Many of these use [adb](https://developer.android.com/studio/command-line/adb.html).
<br>
<br>

### Table of contents
1. [Emulator tips](#EmulatorTips)
    1. [Pasting text in a text box](#PastingText)
    2. [Controlling the emulator via telnet](#Telnet)
    3. [Finding files](#FindingFiles)
2. [Accessing Sqlite databases](#Sqlite)
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

* Start adb and make sure Android Emulator is running

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


