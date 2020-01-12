---
title: Updating the GRUB Background
date: 2020-01-12 10:51:50
---

I used this to update the background from the garish purple to a custom image using Ubuntu 19.10.

First, move an image to `/boot/grub`. According to the [Grub Manual](https://www.gnu.org/software/grub/manual/grub/grub.html#Simple-configuration), the background image must end with `.png`, `.tga`, `.jpg`, or `.jpeg`. Then run `update-grub`.

For example,

```bash
sudo cp /home/user/pictures/new-background.png /boot/grub
sudo update-grub
```
