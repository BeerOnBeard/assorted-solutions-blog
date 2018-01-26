---
title: UEFI RAID Blows GRUB
subtitle: Setting up software RAID on Ubuntu 16.04 LTS
date: 2018-01-26
---

FOUR letter acronyms?!?! I thought we agreed to stick with THREE letter acronyms! I can think of a few choice four letter words to describe my experience with my RAID setup. But first. . . what are they?

**UEFI:** Unified Extensible Firmware Interface
**RAID:** Redundant Array of Independent Disks
**GRUB:** Grand Unified Bootloader

Now to decrypt the cryptic title of this post. I wanted to use a machine I built in 2012 for something useful. I decided to build out a Linux system with git, a samba server, and a DNS server so I could have my own top-level domain. I outline the basics in a previous post, ["Home Server for Fun and TLD"](/2018/01/21/Home-Server-for-Fun-and-TLD). I hit some snags. Now it's time to retrospect and leave some notes for my future self and any other poor soul that happens to stumble upon this post.

tl;dr: Do not select `Force UEFI` when installing Ubuntu on a software RAID system if you want to set a partition as bootable during install.

The system I was building on was an AMD FX-8120 with 16GB of RAM and two disks; one 1TB disk and one 1.5TB disk. It's what I had lying around and I didn't want to buy anything new. I figured I would just leave the extra 500GB alone and hope the 1TB disk ate it before the 1.5TB disk did and buy another 1.5TB disk when the time came. Then I could expand the RAID partition to use that extra space.

After reading up on [Ubuntu's Swap FAQ](https://help.ubuntu.com/community/SwapFaq), and realizing this system would never hibernate, I decided to use an 8GB swap partition and use whatever was left for the main partition. Since I had two different sized drives, I would first partition the 1TB drive, get the exact size of the larger partition, and set the main partition on the 1.5TB drive to the same size. That would make RAIDing easy.

Like any good engineer, I set up a test system before deploying to production. I decided to use [VirtualBox](https://www.virtualbox.org/wiki/VirtualBox) to install Ubuntu 16.04 LTS on top of two RAIDed virtual drives to get the hang of installing Ubuntu on a RAID system using [mdadm](https://linux.die.net/man/8/mdadm). I made the two virtual drives different sizes to emulate what I was going to deal with in the production run.

After downloading the ISO, I spun up the VM with the virtual disk mounted. The installation process is pretty much the same as any other install of Ubuntu recently. Just keep clicking `Enter` until it asks you for your name and password, then keep clicking `Enter` some more. The difference when installing on a new software RAID comes when you hit the partitioning section of the install. Using the [Ubuntu page on Software RAID installation](https://help.ubuntu.com/community/Installation/SoftwareRAID) as a guide, I chose the `Manual` partitioning method.

I set up my partitions with the swap partition in the front. That way, in the future, I can expand the main partition to the end of the disk and have a single, contiguous partition. If I put the main partition at the beginning, I would have to move the swap partition to the end of the drive before expanding the main partition.

I created the partitions on both drives. For the swap partitions, I set the `Use as` option to `swap area`. For the main partitions, I set the `Use as` option to `Ext4 journaling file system`, set the `Mount point` to `/`, and set the `Bootable flag` to `On`. I made the main partition on the larger drive the same size as the main partition on the smaller drive. I just left the rest of the space to hang out un-allocated.

Then I set up RAID. I selected the `Configure software RAID` option and then `Create MD device`. I chose `RAID1`, then `2` active devices, then `0` spare devices. When asked to choose the active devices, I chose both swap partitions. Then I did the whole thing again for the main partitions. I chose `Yes` to write the changes to the storage devices and configure RAID and selected `Finish` when presented with `Software RAID configuration actions`. I ended up with RAID1 device #0 as my 8GB swap and RAID1 device #1 as my main partition.

Back at the original `Partition disks` screen, I saw the RAID1 devices #0 and #1 at the top and the physical drives listed below. I selected the first partition on RAID1 device #0 and set `Use as` to `swap area`. Then selected the first partition on RAID1 device #1, set `Use as` to `Ext4 journaling file system`, set the `Mount point` to `/` then selected `Done`.

Finally, I could `Finish partitioning and write changes to disk`! Boom! RAID1! The rest of the installation was clicking `Enter` and setting up user name and password. Easy peasy.

With great confidence, I started up my production machine with a USB stick containing Ubuntu 16.04 LTS. Little did I know, I would spend hours trying to figure out why everything didn't go as easily as the virtual system. . . 

Instead of boring you with the four letter words I yelled at my inanimate machine, I'll skip to the solution. Basically, my motherboard supports UEFI. Ubuntu, earlier in the installation process, asked if I would like to force UEFI. I selected `yes` without thinking. I found out later, I could not set the partitions to be bootable if I selected `Force UEFI` earlier in the Ubuntu install. I wanted to set each partition as bootable so that, in the case that one drive failed, the system would still be able to boot on either living drive. Lesson learned. And now, lesson archived for my later self to remember.

Happy RAIDing!
