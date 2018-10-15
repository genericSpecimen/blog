---
title: "The Android Debug Bridge"
date: 2018-10-14T16:35:23+05:30
draft: true
---
Android Debug Bridge (_**adb**_) is a command-line tool that lets you communicate with an actual device or an emulator. If you have experience with developing Android applications, you might be quite familiar with this nifty tool. This tool allows you to install and debug applications and also provides you access to the Unix shell on your Android device. If you don't have experience with developing Android applications, don't fret! This post won't be requiring knowledge of those aspects anyway. Instead, we will be focussing on how users of an Android device might make use of this tool in their daily lives.

Here are some of the ways you can make use of **_adb_**:

* [pull files from your device to your computer](./#push-and-pull-files-to-and-from-your-device)
* [push files to your device from your computer](./#push-and-pull-files-to-and-from-your-device)
* [take a screenshot](./#taking-a-screenshot)
* [record the screen of your device](./#record-your-device-s-screen)
* [send touch events to your device](./#sending-touch-events-to-your-device)

Before proceeding to execute these commands, make sure you have [USB Debugging enabled](https://developer.android.com/studio/command-line/adb#Enabling) in your device, and **_adb_** installed in your computer, which is included with Android Studio (if you've installed it) else you can install **_adb_** and other tools bundled in the [**__platform-tools__**](https://developer.android.com/studio/releases/platform-tools) package. To verify **_adb_** is installed, type the command:
```
$ adb devices
```
This should print an output similar to this:
```
$ adb devices
List of devices attached
2918f25e0404    device

```



## Push and pull files to and from your device
The most basic functionality would be to store data in and access data from your devices. I use these commands every time I need to transfer large files, since I find the MTP protocol to be unreliable.

To push (copy) a file from your computer to the device, use the command:
```
adb push local remote
```
To pull (copy) a file from your device to your computer, use the command:
```
adb pull remote local
```

Here, ```local``` refers to the path to the target file/directory on your computer and ```remote``` refers to the target path on your device.
Here's a sample push-pull session:
```
$ adb push home.png /sdcard/
home.png: 1 file pushed. 1.6 MB/s (16154 bytes in 0.010s)
$ adb pull /sdcard/home.png homeagain.png
/sdcard/home.png: 1 file pulled. 4.4 MB/s (16154 bytes in 0.003s)
$ ls -l | grep 'homeagain.png'
-rw-r--r--  1 hooman users  16154 Oct 14 23:12 homeagain.png
```



## Taking a screenshot
It might be impractical to use **_adb_** to do something as trivial as taking a screenshot, but in case your hardware buttons refuse to work and you need to take a screenshot of your device, **_adb_** will be there to your rescue.
If you're already inside the shell of your device, you can use this command to take a screenshot.
```
screencap filename
```
To take a screenshot of your device from your computer's terminal, use the command:
```
adb shell screencap /sdcard/screen.png
```
Now we can pull the file:
```
adb pull /sdcard/screen.png
```
Here's a sample session that demonstrates taking a screenshot
```
 $ adb shell screencap /sdcard/test.png
 $ adb pull /sdcard/test.png
/sdcard/test.png: 1 file pulled. 15.5 MB/s (526703 bytes in 0.032s)
 $ ls -l | grep 'test.png'
-rw-r--r--  1 hooman users 526703 Oct 14 22:42 test.png
 $ file test.png
test.png: PNG image data, 1080 x 1920, 8-bit/color RGBA, non-interlaced

```



## Record your device's screen
Starting from Android 4.4 (API level 19), we have the ```screenrecord``` command that can be used to record our device's screen. By default, it records in the MPEG-4 video format, and records for 3 minutes, unless we specify a time limit using the ```--time-limit``` flag. Here's a sample session that demonstrates recording our screen.
```
 $ adb shell screenrecord /sdcard/demo.mp4
^C
 $ adb pull /sdcard/demo.mp4
/sdcard/demo.mp4: 1 file pulled. 21.6 MB/s (24519057 bytes in 1.081s)
 $ file demo.mp4
demo.mp4: ISO Media, MP4 v2 [ISO 14496-14]
```
Is that all we can do with ```screenrecord```? There's more. Say you want to mirror your Android device's screen to your computer. That can be done using ```screenrecord``` and [```ffmpeg```](../ffmpeg) using this command:
```
adb exec-out screenrecord --bit-rate=16m --output-format=h264 --size 1920x1080 - | ffplay -framerate 60 -framedrop -bufsize 16M -
```
I've read this [question](https://stackoverflow.com/questions/31472962/use-adb-screenrecord-command-to-mirror-android-screen-to-pc-via-usb) on stackoverflow and tried out the different commands people have listed there. This one seems to work fine for me. If you face any issues, try going through the link.
But hold on! Why did we use ```adb exec-out``` instead of ```adb shell```? The answer lies in this [commit](https://android.googlesource.com/platform/system/core/+/5d9d434efadf1c535c7fea634d5306e18c68ef1f) which states that unlike ```adb shell``` the ```adb exec-out``` command doesn't use ```pty``` (abbreviation of pseudoterminal, which is a pair of virtual character devices that provide a bidirectional communication channel). Although a ```pty``` helps to take care of interactive user input such as Ctrl-C, it also has a side affect of mangling the binary output of the process. The only way to fix the garbled output was to post-process the file to remove the extra mangled characters in the file. Thus, another version of the ```shell``` command was provided, called ```exec-out``` which doesn't have the interactive pty.

Options:

The ```--bit-rate=16m``` option sets the video bit rate to 16Mbps.
The ```--output-format=h264``` option sets the output format to h264 (which provides good video quality at substantially lower bit rates than previous standards).
The ```--size 1920x1080``` option sets the video size to 1920x1080 (use a resolution supported by your device).

Now we pipe the output of this command to the ```ffplay``` media player, which uses the FFmpeg libraries.

Options:

The ```-framerate 60``` option specifies the output frames per second to be 60.
The ```-framedrop``` option specifies to drop video frames if video is out of sync.
The ```-bufsize 16M``` option sets the ratecontrol buffer size to 16M bits.


## Sending touch events to your device
We can send touch input events to our device in case our touch screen is not responding and we need to use our phone.

Enter the shell on your device using
```
$ adb shell
```
Once in the shell, enter the command...
```
$ getevent -l
```
...and press anywhere on the device screen. You should get an output like this:
```
<...>
/dev/input/event1: EV_ABS       ABS_MT_TRACKING_ID   0000126f            
/dev/input/event1: EV_ABS       ABS_MT_POSITION_X    000003f3            
/dev/input/event1: EV_ABS       ABS_MT_POSITION_Y    0000072f            
/dev/input/event1: EV_KEY       BTN_TOUCH            DOWN                
/dev/input/event1: EV_SYN       SYN_REPORT           00000000            
/dev/input/event1: EV_ABS       ABS_MT_TRACKING_ID   ffffffff            
/dev/input/event1: EV_KEY       BTN_TOUCH            UP                  
/dev/input/event1: EV_SYN       SYN_REPORT           00000000            
/dev/input/event1: EV_KEY       KEY_BACK             DOWN                
/dev/input/event1: EV_SYN       SYN_REPORT           00000000            
/dev/input/event1: EV_KEY       KEY_BACK             UP                  
/dev/input/event1: EV_SYN       SYN_REPORT           00000000            
^C
```
The ```getevent``` tool runs on the device and provides information about input devices and a live dump of kernel input events. We use the ```-l``` option to display textual labels for all event codes.

Let's take a moment to observe the output.

**EV_ABS** specifies the absolute coordinates for an event. Values are usually ABS_[XYZ], or ABS\_MT for multi-touch displays.

**EV_KEY** specifies a Key press (KEY_*) or touch (BTN_TOUCH).

**ABS_MT_POSITION_X**: Reports the X coordinate of the touch event.

**ABS_MT_POSITION_Y**: Reports the Y coordinate of the touch event.

The output indicates we have touched on the screen at the coordinates 3f3 and 72f (in hexadecimal). Converting these to decimal, we get the X coordinate as 1011 and the Y coordinate as 1839.

Now we can type the command...
```
input tap 1011 1839
```
... and a touch event will be generated at that same location. 

_But wait a minute, you said this is useful if our touch screen is not responding, in that case, how will we find the coordinates in the first place?_

For that, you need to know a bit about your device. The top left corner of your device's screen has _(0,0)_ as its X and Y coordinates. If you know your resolution, say its _(1080x1920)_, you can guess the location of where you want to tap by trial and error. The bottom right corner will have _(1080,1920)_ as its X and Y coordinates. The X coordinate (horizontal) will usually be smaller than the Y coordinate (vertical).
#### Reference links:

* [**__adb__**](https://developer.android.com/studio/command-line/adb)
* [**__ffplay__** - link 1](https://ffmpeg.org/ffplay-all.html)
* [**__ffplay__** - link 2](https://ffmpeg.org/ffplay.html)
* [**__Android Input Architecture__**](http://newandroidbook.com/Book/Input.html?r#InputStack)
* [**__Touch Devices__**](https://source.android.com/devices/input/touch-devices.html)
* [**__getevent__**](https://source.android.com/devices/input/getevent.html)