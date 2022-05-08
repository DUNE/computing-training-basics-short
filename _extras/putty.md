---
title: Putty Setup 
permalink: putty.html 
---


### PuTTY

This is the application we use in Windows to SSH into other systems. Download the appropriate Windows installer (MSI file) for your system from [here](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) (probably the 64-bit x86).

The default options for the installer should be fine.

## Kerberos Ticket Manager

Download the MIT Kerberos Ticket Manager MSI installer from [here](http://web.mit.edu/kerberos/dist/#kfw-4.1) ("MIT Kerberos for Windows 4.1" as of the time of writing). The installation is mostly automatic - if prompted for the type of installation, choosing "Typical" will be fine.

Once this is installed, you can launch it as an app to manage your Kerberos tickets. To prepare to login to the FNAL cluster, press "Get Ticket", and you can authenticate with `[username]@FNAL.GOV` and the Kerberos password that you should already have setup. This replaces the `kinit` command that's used to obtain a Kerberos ticket in unix systems, and the graphical interface shows you your active tickets instead of using `klist` as in unix systems.

## Xming

This is used for X11 graphics forwarding. Download from [here](https://sourceforge.net/projects/xming/) (see also [official notes](http://www.straightrunning.com/XmingNotes/)). Run the installer (keeping all default options is fine).

Once installed, run XLaunch to check the settings. Follow the XLaunch configurations specified [here](http://www.geo.mtu.edu/geoschem/docs/putty_install.html).

Once this is done, in the future when you need to use X11 graphics forwarding, simply launch Xming and let it run in the background.

## Configuring PuTTY

Once all the previous components are installed, open up PuTTY and configure as follows:

1. Under Connection/SSH/X11, check "Enable X11 Forwarding", and set the X display location to "localhost:0.0"
2. Under Connection/SSH/Auth/GSSAPI, check "Allow GSSAPI credential delegation". Make sure the MIT Kerberos GSSAPI64.DLL (or GSSAPI32.DLL, if you're using the 32-bit version) is in the list of "Preference order for GSSAPI libraries". If it's not, use the "User-supplied GSSAPI library path" option to navigate to where you've installed the MIT Kerberos ticket manager and select this library.
3. Under Session, Fill in the host name with [username]@dunegpvm[0-15].fnal.gov (or whatever the plan is for this tutorial?). 
4. Save this configuration by typing a name in the box labeled "Saved Sessions" and pressing "Save". You can load this configuration in the future to reuse these settings.

This should allow you to SSH to the FNAL clusters and follow the rest of the tutorial.

*Created on 20220508 using notes provided by Roger Huang.* 

[Go to Setup]({{ site.baseurl }}/setup.html) 
