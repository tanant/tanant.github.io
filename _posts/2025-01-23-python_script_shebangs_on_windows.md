---
layout: post
title:  "Python script shebangs (#!) on windows"
---


Hmm.. that was an interesting quick dive.

I'm currently building some pipeline out on a windows-based environment and as we move towards more and more structured workflows we're slowly starting to need more scaffolding.. but due to the maintenance load and complexification I'm trying to keep out of build-a-launcher territory for as long as possible. It's less slick by far, but it's also a lower challenge for someone else to get in and understand what's going on.

Obviously, being in a VFX-space python is one of the default languages so I was just working out how best to ship utility scripts when it struck me I didn't _actually_ know how the pathway works from someone in a powershell console typing `runme.py` and python grabbing and running that. It's actually a bit more involved than i thought, but also quiiite handy.

## Setup a rig...

Yeah, this is literally it - dump out version and path of executable. Save this as `runme.py`
```
import sys
print (sys.version)
print (sys.executable)
print ("---")
```
For plumbing purposes, open powershell up, and make sure you get something like this (i.e. you've got something that runs the script)
![powershell output](/assets/images/windows_python_runme.png)

## ...and roll into the registry
Fundamentally the loop involves `Explorer` doing the identification and dispatch of the file to a program to execute. Summarizing from this [_amazing_ answer](https://superuser.com/a/1715213) from [kxr](https://superuser.com/users/554351/kxr) on superuser

- Find the program ID associated. Check the registry at `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.py\UserChoice\ProgId` to be sure (but it'll be `Python.File`)
- Find what the shell open command is. Check the registry at `HKEY_CLASSES_ROOT\Python.File\shell\open\command`

and you should see it's ultimately `"C:\WINDOWS\py.exe" "%L" %*`

## oooo, a launcher?

So this makes sense, there's a launcher! This little `py.exe` stub does two things - it actually lets you direct it to various different versions on your system _and also intereprets the first line of your .py script as a shebang_

You can take advantage of this by inserting things like this:
- `#!C:/path/to/executable.python.thing.exe` (direct)
- `#!python3.11` (version specification)
- **`#!/usr/bin/env python3.11`** (yeap, on a _windows_ environment)

So that's sort of solving one of my future considerations around environment management - it's not a problem for me _yet_ but now I'm across how this works, I'm happier doing some of my dumber things and trusting on a baseline of "a python 3.3+ has been installed at some point on the machine" as that should give me access to the lanucher framework, and then via some fleet management I can configure and mangle this as needed. 


## more details!
Info is available if you invoke `py.exe` with the ol -h flag, and it'll suggest reading [https://docs.python.org/3/using/windows.html#python-launcher-for-windows](https://docs.python.org/3/using/windows.html#python-launcher-for-windows), but you can also skip straight to [PEP-397](https://peps.python.org/pep-0397/) for alll the neat gory configuration details.