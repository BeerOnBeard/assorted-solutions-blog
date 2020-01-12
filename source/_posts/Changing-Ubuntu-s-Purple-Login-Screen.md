---
title: Changing Ubuntu's Purple Login Screen
date: 2020-01-12 11:07:45
---

I'm not a fan of the purple that Ubuntu puts on the lock screen. Time to update it!

I'm using Ubuntu 19.10 with Gnome. Gnome uses CSS for many of its styles and, lucky for me, that includes the background of the lock screen.

The file `/usr/share/gnome-shell/theme/gdm3.css` contains a definition for `#lockDialogGroup`. I commented out the existing definition and added my own.

```css
/*
#lockDialogGroup {
  background: #4f194c url(noise-texture.png);
  background-repeat: repeat;
}
*/

#lockDialogGroup {
  background: #000000;
}
```

After a quick reboot, the background of the lock screen is now black!
