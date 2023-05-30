---
layout: post
title:  "Brutefocing Android Patterns Using HID Devices"
---

# Preface
So my aunt recently died and left behind a perfectly good "Samsung Galaxy S7 FE" tablet. Unfortunately (or fortunately for you the interested reader) we did not know the password for the device which left us with one option: Factory Reset.

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

For more about this I recommend this riskeco.com blog post: [^frp-bypass]. Note that for the Tablet in question another AT code then the one described in this blog might be needed.

# Human Interface Devices / Requirements on Linux
Human Interface Devices (HID) is a standard that devices can follow to communicate with your operating system to exchange information.
Without going too deep into [^hid-specs] the HID protocol gives a common protocol for input devices.
Among others, the USB protocol is implemented by mice, keyboards or similar input devices.

Luckily for us the Linux kernel also supplies the open to emulate such a device using the **HID Gadget Driver** [^hid-gadget-driver].
Those gadget drivers can be enabled using **configFS** kernel feature Linux provides. ConfigFS support must be enabled in the kernel, which has to be done at kernel build time.
For an example on linux on how to do setup a HID  device check out this repo [^hid-sample].

For some further on the USB Gadget API: [^kernel-gadget-api]

## HID Gadget on Android and and interacting with the gadet
On Android configFS is enabled on most custom ROMS (e.g. LinageOS). For the configuration of the HID devices there is an app called **USB Gadget Tool**[^usb-gadget-tool] which makes to configuration of HID devices easier.

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
So what can we do with this? Inspired by the **Android Pin Brutefocing Project**[^pin-bruteforce-repo] I have created a bash script that will bruteforce the pattern lock or an android device.

The problem with bruteforcing is the timeout between failed attempts. In our case we can actually factory reset the device one more which will reset the timeout every time we do.

## Features
My Tool I created and publishing has the following features:
- a resume function which allows you to pickup between attempts
- a list of android patterns based on the distance between the pattern points
which is available here on its dedicated repo on github [^all-pattern-repo]
- an tool to create optimized bruteforce patterns based on the paper [^pattern-frequency]
- an installer script to just push and run the apk
- a usage menu to setup the location of the dots on the screen
- a backup and resume feature to pause the bruteforcing
- auto backup of the current progress

## Challenges and Known Issues
When writing PatternBash I came across a couple challenges which I will describe in the following.

### Skipping Inputs
I have realized that sometimes the device skips over certain HID inputs which throws off the whole coordinates. I invastigated this issue alot and acually rewrote the tool to not use direct bash to device communication like I showed above but use the HID gadget binary provided by the linux team.
Unfortunately this was also unsucessful, so my only fix was to just go slower and limit the inputs.

### Mouse positioning
There is actually no way to know where the mouse is currently located on the device. Thhe solution to work around this is to just move the mouse to the very top left and call this location (0,0) from there on out we can access the different locations on the screen using coordinates.

### Pathfinding
To move the mouse between the different points we want bruteforced we need some kind of pathfinding to move the mouse between the points in question. Because this is not trivially implemented in bash (and also depended on the distance of each step), I ended up implmeting a very lazy hack which looks like this:
```bash
    while [ "$CURRENT_X" -ne "$JUMP_X" ] || [ "$CURRENT_Y" -ne "$JUMP_Y" ]; do
        if [[ $CURRENT_X -lt $JUMP_X ]]; then
            move_right $MOVE_DISTANCE
        fi
        if [[ $CURRENT_X -gt $JUMP_X ]]; then
            move_left $MOVE_DISTANCE
        fi
        if [[ $CURRENT_Y -lt $JUMP_Y ]]; then
            move_down $MOVE_DISTANCE
        fi
        if [[ $CURRENT_Y -gt $JUMP_Y ]]; then
            move_up $MOVE_DISTANCE
        fi
    done
 ```

This works, but breaks for some complicated patterns like connecting 1 and 8 which would require a long diagonal line between those points. Above code takes a diagonal line with an angle too sharp and ends up touching oter points inbetween.

### Random Phone Resets
White brutefocing I found that some devices just decide to reboot at a certain point during brutefocing. This issue appeard very consistantly on the device I was testing after 44 attempts for the longest time. There where however times when this did not happen.

I am not sure if this is a protection implemented by Samsung or some other non related issue but my tool suppots this behavior by pausing the brutefocing at a certain point using the `SHUTDOWN_AT` variable.

### Are we done bruteforcing?
Another problem is that we dont aucally know once we are done brutefoceing the pattern of the device. This is not solvable with just a HID device to my knowlage, because we are not able to get back any feedback from the device.

### The time of each level of cooldown / Exponential growth
The times between each attempts and how many attempts are allows in a certain cool down level is different from device to device (see the [^pin-bruteforce-repo] for some examples.
This is a little cumbersome because the initial bruteforce attempts need close supervision. In my experience the first 4 levels tend to be a little different but then the logic is normally consistant.

I recommend monitoring the first levels and then trying to approximate the attempts using some function. Wolfram Alpha is quite nice for this ([sample](https://www.wolframalpha.com/input?i=fit%281%2C+30%09%2C60%09%2C120%2C%09300%2C+540%2C%091020%2C%091980%29)).
When fine-tuning your levels, set an alarm for the rough time when you think a switch will happen and configure PatternBash to switch to a higher timeout sooner rather then later.


## Some Stats
The following image shows some numbers on the time it will take bruteforcing using the cooldown numbers from the Samsung Galaxy Tab S7

![cooldown-stats](cooldown_stats.png)
As the growth is exponential, it does not take too many increases in the cooldown level to drop below 1 attempt per minute and somewhere around 1020 seconds of delay it normally makes sense to reset the device again.
This gives us around 80 attempts before a reset.
For a four digit code where all dots are only a distance of one apart we have around 500 possible combinations:

```
# cat exactly-4-connected-dots-distance-1.txt | wc -l
496
```
Which would only require 7 resets to exaust the full list. Raw bruteforcing time would be around 560 minutes and with manual resets 600 minutes seems like a fair approximation.
Overall for a random pattern that gives us an expected runtime of 300 minutes.
The paper [^pattern-frequency] I used to build the bruteforce list mentioned that `44%` of the pattern starts on the first pattern marker.
Which would mean that in reality the time could be even lower.


## Download
So with this out of the way I am releasing **PatternBash**, available on my [Github](https://github.com/0xE0-rng/PatternBash).
I will most likely not continue development on this and will not give any support on it, but I felt like creating it had some nice information to share with the world.



## Future Work
Raspian acually also has ConfigFS support, so my script can be run on a raspberry too. This same rasperry can be used to drive stepper motors that automatically perform a factory reset on the device. I have experimented on this but did not create a running prototype. Maybe in another blogpost...
One sulution to receive feedback from the device (e.g. after the bruteforcing was complete) would be to just film the device / create a pictue after each attept. A simple NN algorithm would be enoughh to decide on the current state of the device.


# Sources
[^frp-bypass]: [Riskeco: Analysis of a Samsung FRP Bypass](https://blog-cyber.riskeco.com/en/analysis-of-samsung-frp-bypass/)
[^pattern-frequency]: [Loge, Marte et al: On User Choice for Android Unlock Patterns](https://web.archive.org/web/20230527233502/http://www.usablesecurity.net/USEC/NDSS/wp-content/uploads/2018/03/01-on-user-choice-for-android-unlock-patterns.pdf)
[^pin-bruteforce-repo]: [The Android PIN Bruteforce Github Repo](https://github.com/urbanadventurer/Android-PIN-Bruteforce)
[^usb-gadget-tool]: [Android USB Gadget Github Repo](https://github.com/tejado/android-usb-gadget)
[^all-pattern-repo]: [My AndroidPatternLock Fork](https://github.com/0xE0-rng/AndroidPatternLock/tree/50eac641818bfb03b1a102c39b4e3c71dafac1bd)
[^kernel-gadget-api]: [kernel.org USB Gadget Driver Documentation](https://www.kernel.org/doc/html/v5.0/driver-api/usb/gadget.html)
[^hid-gadget-driver]: [kernel.org HID Gadget Documentation](https://www.kernel.org/doc/html/latest/usb/gadget_hid.html)
[^hid-specs]: [usb.org HID Protocol Specification](https://usb.org/sites/default/files/hut1_4.pdf)
[^hid-sample]: [HID Keyboard Gadget Sample by dlyoung](https://github.com/qlyoung/keyboard-gadget/blob/master/gadget-setup.sh)
