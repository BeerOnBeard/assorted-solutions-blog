---
title: It's Elementary!
subtitle: Installing and setting up ElementaryOS
date: 2018-02-04
---

I won a decommissioned engineering laptop from work a while back. I had upgraded the Windows 7 install that was originally on it to Windows 10 because it was free. However, I already had a Windows laptop. I've been pretty sick of using VirtualBox for all my Linux-based development, so I thought I would install a distribution directly on the hardware to have a dedicated Linux machine.

ElementaryOS came up in one of my news feeds a while back. I thought it was pretty, then forgot about it. Then, a couple days ago I was shoulder surfing a colleague while we were working on spinning up a Linux build machine for work and noticed he had a very pretty OS running in a virtual machine. Lo and behold, ElementaryOS!

This weekend I downloaded the ISO and spent a day working on stuff using the LiveCD mode. After a day, I realized that the distribution was pretty slick and it would be worth the trouble of wiping a hard drive and committing.

The install was pretty straightforward, like most modern Linux distributions. Select your language and keyboard layout, allow it to blow away your hard drive and partition as it pleases, connect to WiFi and let it automagically install and update all the packages it needs. 10 to 15 minutes later, new OS.

When I restarted for the first time, I got the error `Invalid partition table!` in white text on a black background. . . not a great start. Luckily, it was only my BIOS that was the issue. I jumped into the BIOS using F2 at post and updated the boot setting from legacy boot to UEFI. I restarted and life was good.

The first thing I changed when I started using ElementaryOS was the single-click to open a file. Coming from a world of operating systems that have required me to double-click for my entire life, I found myself accidentally navigating into folders and executing applications all over the place. It's easy to disable, so I did.

```bash
gsettings set org.pantheon.files.preferences single-click false
```

As a side note: using the application `dconf` worked in the LiveCD environment. However, once I actually installed ElementaryOS and booted to it for real, `dconf` wasn't working correctly. I would write the setting as false, then open up the file manager and see that I still could single-click to open a folder. I would read the value for `single-click` again and it would be reverted. Lame. `gsettings` for the win. Moving on . . .

I have a DisplayLink docking station that has monitors and peripherals all set up to dock my home system, my work system, or my wife's system via USB. To get ElementaryOS working with that, I had to install the DisplayLink drivers from the [DisplayLink Ubuntu downloads page](http://www.displaylink.com/downloads/ubuntu). ElementaryOS 0.4.1 Loki is based on Ubuntu 16.04.03 LTS. The page lists that the drivers are compatible with Ubuntu 16.04 and 17.10. Awesome.

I downloaded the zip folder with a `.run` file, a license, and an empty `_MACOSX` folder. I only needed the file `displaylink-driver-4.1.9.run`. I updated the properties of the file to allow execution (`chmod +x displaylink-driver-4.1.9.run`). Before I could actually run the installer successfully, I had to install the dependency [DKMS](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support).

```bash
sudo apt-get install dkms
```

Then I could execute the `.run` file.

```bash
sudo ./displaylink-driver-4.1.9.run
```

Finally, I plugged in the docking station. I had to go to System Settings > Displays and enable the displays before they would start working. Now I have my monitors, keyboard, mouse and headphones. Sweet. Speaking of headphones . . .

The default browser, Epiphany, does not support running the Spotify web player. So, I went searching for a way to install Spotify as a native application. Lucky for me, Spotify has [a dedicated web page for installing on Linux](https://www.spotify.com/us/download/linux/). I followed the instructions for installing via a Debian package. Time to crank up the music.
