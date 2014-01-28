---
layout: post
title: "Dell DRAC remote console access from Mac OS X"
description: ""
category: ""
tags: ["howto","java","drac","idrac"]
---
In the event that you have the misfortune to administer a Dell server with one of their [DRAC](https://en.wikipedia.org/wiki/Dell_DRAC) out-of-band management controllers (oops, am I editorializing already?), you may encounter difficulty accessing the remote console from a Mac OS X client, given that it's implemented as a Java applet.  I found various documents offering a range of solutions for these problems, but none of them matched the issue I encountered, so I'm documenting it here.  Hopefully this will save you some time and frustration.

## The Problem

After connecting to the DRAC web interface, you try to launch the remote console (I was using Firefox, with Java 7 Update 51 64-bit) and are prompted to do something with a [JNLP](https://en.wikipedia.org/wiki/JNLP) file.  You sensibly tell your browser to open it with Java Web Start (on my Mac OS X 10.7 box that lives at `/System/Library/CoreServices/Java Web Start.app`, but then oh no! you see an error message:
    
    Unable to launch the application.
    ...
    Error: Missing required Permissions manifest attribute in main jar:

## The Quick but Unrighteous Solution

1. Open the Java Control Panel (accessible via System Preferences).
2. Select the Security tab.
3. Make sure "Enable Java content in the browser" is checked.
4. Set Security Level to "Medium".

## The Annoying but Virtuous Solution

1. Open the Java Control Panel (accessible via System Preferences).
2. Select the Security tab.
3. Make sure "Enable Java content in the browser" is checked.
4. Leave Security Level at "High" or "Very High".
5. Click "Edit Site List..."
6. Add an entry in the Exception Site List that looks like `https://<hostname or IP of the DRAC interface>`
7. Click "OK", then "OK" again.

{% include JB/setup %}
