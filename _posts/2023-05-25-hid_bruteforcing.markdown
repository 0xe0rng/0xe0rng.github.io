---
layout: post
title:  "Brutefocing Phones Using HID Devices"
---

# Preface
So my aunt recently died and left behind a perfectly good Samsung Galaxy S7 FE tablet. Unfortunately (or fortunately for you the interested reader) we did not know the password for the device which left us with one option: Factory Reset.

Unfortunatly all modern Android devices employ a feature Google calls **FRP** (Factory Reset Protection) which requires you to either:
- Enter the correct PIN / the correct Pattern
- or sign in with the connected Google Account.

Unfortunatly neither of those where available to me so I had to find another solution:

### The Easy Way
Before going into the main topic of this blog post I want to point out that there are much easier ways to perform what I did. There are still many different FRP bypasses available and you might find a working one if you google a while.
For this specific device and many others Samsung was nice enough to add a small backdoor in their baseband with can be used to enable adb and use it to uninstall the app responsible for FRP. The flow goes something like this:
- Open the Dialer (though some talkback trickery)
- enter `*#0*#` in the Dialer to open the Samsung debug menu which connects the phones modem in serial mode to a USB connected computer
- use specific AT commands to communicate with the modem and enable usb debugging
- we can then use the shell to finish the setup manually (any bypass FRP)

If you want to learn more out this I recommend this riskeco.com blog post: [https://blog-cyber.riskeco.com/en/analysis-of-samsung-frp-bypass/](https://blog-cyber.riskeco.com/en/analysis-of-samsung-frp-bypass/). Note that for the Tablet in question you might need another AT code  then the one described in this blog

# Theory of Human Interface Devices
Human Interface Devices (HID) are a standard that devices can follow to communicate with your operating system to exchange information.
Without going too deep into [theory](https://usb.org/sites/default/files/hut1_4.pdf) the hid protocol allows the OS to interpret inputs from US the user.
Among others, the USB protocol is normally implemented by mice, keyboards or similar input devices. Luckely the Linux kernel also supplies the open to emulate such a device using the **HID Gadget Driver** [https://www.kernel.org/doc/html/latest/usb/gadget_hid.html](https://www.kernel.org/doc/html/latest/usb/gadget_hid.html)


Once enabled this makes the `/dev/hidg0` device available.

# Requirements for android / Linux Kernel

# The Attack, Introducing the existing brute force solutions
- Gets reset after factory reset

# The tool
- exponetuinal growth
- adjusted wordlist
# Issues
- random resets?
- sucesss?
- skipping inputs
- tracking location

- Hid test vs echo

# Improvements and Future Work
- raspberry with stepper motor
- attacks based on input (dialer?) Frp bypass?


