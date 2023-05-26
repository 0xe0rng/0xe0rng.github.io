---
layout: post
title:  "Brutefocing Phones Using HID Devices"
---

# Preface
So my aunt recently died and left behind a perfectly good Samsung Galaxy S7 FE tablet. Unfortunately (or fortunately for you the interested reader) we did not know the password for the device which left us with one option: Factory Reset.

Sadly all modern Android devices employ a feature Google calls **FRP** (Factory Reset Protection) which requires you to either:
- Enter the correct PIN / the correct Pattern
- or sign in with the connected Google Account.

Neither of those where available to me so I had to find another solution:

### The Easy Way
Before going into the main topic of this blog post I want to point out that there are much easier ways to perform what I did. There are still many different FRP bypasses available and you might find a working one if you search for a while.
For this specific device and many others Samsung was nice enough to add a small backdoor in their baseband with can be used to enable adb and use it to uninstall the app responsible for FRP. The flow goes something like this:
- Open the Dialer (e.g. though some talkback trickery)
- enter `*#0*#` in the Dialer to open the Samsung debug menu which connects the phones modem in serial mode to a USB connected computer
- use specific AT commands to communicate with the modem and enable USB debugging
- we can then use the ADB shell to finish the setup manually (any bypass FRP)

If you want to learn more out this I recommend this riskeco.com blog post: [https://blog-cyber.riskeco.com/en/analysis-of-samsung-frp-bypass/](https://blog-cyber.riskeco.com/en/analysis-of-samsung-frp-bypass/). Note that for the Tablet in question you might need another AT code  then the one described in this blog.

# Human Interface Devices / Requirements on Linux
Human Interface Devices (HID) is a standard that devices can follow to communicate with your operating system to exchange information.
Without going too deep into [theory](https://usb.org/sites/default/files/hut1_4.pdf) the HID protocol gives a common protocol for input devices.
Among others, the USB protocol is implemented by mice, keyboards or similar input devices.

Luckily for us the Linux kernel also supplies the open to emulate such a device using the **HID Gadget Driver** [https://www.kernel.org/doc/html/latest/usb/gadget_hid.html](https://www.kernel.org/doc/html/latest/usb/gadget_hid.html).
Those gadget drivers can be enabled using **configFS** kernel feature Linux provides. ConfigFS support must be enabled in the kernel, which has to be done at kernel build time.
For an example on linux on how to do setup a HID  device check out this repo [https://github.com/qlyoung/keyboard-gadget/](https://github.com/qlyoung/keyboard-gadget/blob/master/gadget-setup.sh).

- further reading [https://www.kernel.org/](https://www.kernel.org/doc/html/v5.0/driver-api/usb/gadget.html)

## HID Gadget on Android
On Android configFS is enable on most custom ROMS (e.g. LinageOS) and there is acually an app which makes to configuration of HID devices easier called [USB Gadget Tool](https://github.com/tejado/android-usb-gadget).

Once a HID gadget device is created and enabled, it makes the `/dev/hidg0` device available which we can use to write to connected HID devices.

To test if your android test device works properly just enable the HID gadget and write to the newly added device like so:
```
echo -ne \\x00\\x64\\x00 > /dev/hidg0
```

You should should now see your mouse move by x units on the screen.



# The Attack, Introducing the existing brute force solutions
- Gets reset after factory reset

# The tool
- exponential growth
- adjusted wordlist
# Issues
- random resets?
- success?
- skipping inputs
- tracking location

- Hid test vs echo

# Improvements and Future Work
- raspberry with stepper motor
- attacks based on input (dialer?) FRP bypass?


