---
layout: post
title:  "Brutefocing Phones Using HID Devices"
---

# Preface
So my aunt recently died and left behind a perfectly good Samsung Galaxy S7 FE tablet. Unfortunately (or fortunately for you the interested reader) we did not know the password for the device which left us with one option: Factory Reset.

Sadly all modern Android devices employ a feature Google calls **FRP** (Factory Reset Protection) which requires us to either:
- Enter the correct PIN / the correct Pattern
- or sign in with the connected Google Account.

Neither of those where available to me so I had to find another solution:

### The Easy Way
Before going into the main topic of this blog post I want to point out that there are much easier ways to perform what I did. There are still many different FRP bypasses available and we might find a working one if searching for a while.
For this specific device and many others Samsung was nice enough to add a small backdoor in their baseband with can be used to enable adb and use it to uninstall the app responsible for FRP. The flow goes something like this:
- Open the Dialer (e.g. though some talkback trickery)
- enter `*#0*#` in the Dialer to open the Samsung debug menu which connects the phones modem in serial mode to a USB connected computer
- use specific AT commands to communicate with the modem and enable USB debugging
- we can then use the ADB shell to finish the setup manually (any bypass FRP)

For more about this I recommend this riskeco.com blog post: [https://blog-cyber.riskeco.com/en/analysis-of-samsung-frp-bypass/](https://blog-cyber.riskeco.com/en/analysis-of-samsung-frp-bypass/). Note that for the Tablet in question another AT code then the one described in this blog might be needed.

# Human Interface Devices / Requirements on Linux
Human Interface Devices (HID) is a standard that devices can follow to communicate with your operating system to exchange information.
Without going too deep into [theory](https://usb.org/sites/default/files/hut1_4.pdf) the HID protocol gives a common protocol for input devices.
Among others, the USB protocol is implemented by mice, keyboards or similar input devices.

Luckily for us the Linux kernel also supplies the open to emulate such a device using the **HID Gadget Driver** [https://www.kernel.org/doc/html/latest/usb/gadget_hid.html](https://www.kernel.org/doc/html/latest/usb/gadget_hid.html).
Those gadget drivers can be enabled using **configFS** kernel feature Linux provides. ConfigFS support must be enabled in the kernel, which has to be done at kernel build time.
For an example on linux on how to do setup a HID  device check out this repo [https://github.com/qlyoung/keyboard-gadget/](https://github.com/qlyoung/keyboard-gadget/blob/master/gadget-setup.sh).

[*] further reading [https://www.kernel.org/](https://www.kernel.org/doc/html/v5.0/driver-api/usb/gadget.html)

## HID Gadget on Android and and interacting with the gadet
On Android configFS is enabled on most custom ROMS (e.g. LinageOS). For the configuration of the HID devices there is an app called [USB Gadget Tool](https://github.com/tejado/android-usb-gadget) which makes to configuration of HID devices easier.

Once a HID gadget device is created and enabled, the `/dev/hidg0` is device available which we can use to write to connected HID devices.

The format to to communicate with the HID Gadget is as follows:
We send three bytes
```
byte0 = 0x00 | 0x01 | 0x02 | 0x04  # to press no button, button 1, button 2 or 3
byte1 = deltaX                     # movement as a char with possible values between -256 and +255
byte2 = deltaY                     # y movement  -"-
```
Typical values for `deltaX` and `deltaY` are 1 or 2 for slow movement and 20 for very fast movement.


So after connecting the device to a host device, we can test the HID gadget and write to the newly added device like so:
```
echo -ne \\x00\\x64\\x00 > /dev/hidg0
```
We should should now see the mouse move by 10 units in the x direction on the host device.

# Introducing PatternBash
## The Attack, Introducing the existing brute force solutions
So what can we do with this? Inspired by the [Android-Pin-Bruteforce-Project](https://github.com/urbanadventurer/Android-PIN-Bruteforce) I have created a bash script that will bruteforce the pattern lock or an android device.

The problem with bruteforcing is the timeout between failed attempts. In our case we can actually factory reset the device one more which will reset the timeout every time we do.

## Features
My Tool I created and publishing has the following features:
- a resume function which allows you to pickup between attempts
- a list of android patterns based on the distance between the pattern points
Which is available here: (https://github.com/0xE0-rng/AndroidPatternLock/tree/50eac641818bfb03b1a102c39b4e3c71dafac1bd)[https://github.com/0xE0-rng/AndroidPatternLock/tree/50eac641818bfb03b1a102c39b4e3c71dafac1bd]
- an tool to create optimized bruteforce patterns based on the paper (On User Choice for Android Unlock Patterns)[https://web.archive.org/web/20230527233502/http://www.usablesecurity.net/USEC/NDSS/wp-content/uploads/2018/03/01-on-user-choice-for-android-unlock-patterns.pdf]
- an installer script to just push and run the apk
- a usage menu to setup the location of the dots on the screen
- a backup and resume feature to pause the bruteforcing
- auto backup of the current progress

## Challenges
When writing PatternBash I came across a couple challenges which I will describe in the following.

### Skipping Inputs
I have realized that sometimes the device skips over certain HID inputs which throws off the whole coordinates. I invastigated this issue alot and acually rewrote the tool to not use direct bash to device communication like I showed above but use the HID gadget binary provided by the linux team. Unfortunately unsucessful, so my only fix was to just go slower and limit the inputs.

### Mouse positioning
There is actually no way to know where the mouse is currently located on the device. My solution to work around this is to just move the mouse to the very top left and call this location (0,0) from there on out we can access the different locations on the screen using coordinates.

### Random Phone Resets
some phones reboot at a certain point
SHUTDOWN_AT=44
The tablet that I was evaluating this on had an

### Are we done bruteforcing?
### Exponential growth

## Download
So with this out of the way I am releasing PatternBash: [https://github.com/0xE0-rng/PatternBash](https://github.com/0xE0-rng/PatternBash)
I will most likely not continue development on this and will not give any support on it, but I felt like creating it had some nice information to share with the world.



## Future Work
Raspian acually also has ConfigFS support, so my script can be run on a raspberry too. This same rasperry can be used to drive stepper motors that automatically perform a factory reset on the device. I have experimented on this but did not create a running prototype. Maybe in another blogpost...



