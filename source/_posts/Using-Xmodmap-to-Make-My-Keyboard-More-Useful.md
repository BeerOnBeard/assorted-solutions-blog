---
title: Using Xmodmap to Make My Keyboard More Useful
date: 2020-01-04 12:40:35
---

I've been using a ThinkPad as my daily driver for a while now and absolutely love how the keyboard feels. However, I miss my dedicated `Menu` key. Thankfully, any Linux distribution running an X windowing system can use the tool `xmodmap` to remap keys.

First, I needed to find the keycode for the key I wanted to change. Normally, I would use the tool `xev` to monitor for key input and grab the keycode from the log provided. However, the PrtSc key on my keyboard did not seem to register a KeyPress or KeyRelease event. Instead, I had to grep the output of my current key mapping looking for the keyword "Print". "Print" is the term used in xmodmap to refer to print screen.

```bash
~ xmodmap -pke | grep Print
keycode 107 = Print Sys_Req Print Sys_Req
keycode 218 = Print NoSymbol Print NoSymbol
```

I had to do some Googling to figure out that keycode 107 is the key I wanted to target. Now that I know, I can create a script I can run whenever I want to change that keymap.

```bash
#!/bin/bash
xmodmap -e "keycode 107 = Menu";
```

Unfortunately, Ubuntu (the distro I'm using now) [lost the ability to automatically load user-defined xmodmap commands around Ubuntu 13.10](https://bugs.launchpad.net/ubuntu/+bug/1243642). If that was not the case, I would have this loaded automatically. All of the workarounds I've seen require much more in-depth changes to startup, sleep, and hibernate commands. Instead, I'll keep it simple so I don't have to remember all that crazy stuff the next time I reinstall my OS.
