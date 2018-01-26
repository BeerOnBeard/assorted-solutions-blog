---
title: Home Server for Fun and TLD
subtitle: Who Doesn't Want Their Own Top-Level Domain?
date: 2018-01-21
---


In early 2012 I built myself a nice mid-tower with an AMD FX-8120 (Bulldozer), 16GB of RAM, and a video card that made my gaming friends cringe. I wasn't interested in playing games, I wanted to play with building multi-threading applications using the latest .NET Framework libraries. I wanted to virtualize. I wanted compute power. I did some of that, then the computer sat for years. . . 

Fast-forward to this past Christmas. I had some vacation time so I started looking for a fun project. I needed a place to put my production code that was safe and private. I wanted to have a place to drop files on my network that my wife and I could share. I wanted to have my own top-level domain.

So I decided I was going to break out that old computer, install Linux on it and set up a DNS server, Git server, and file share. I'm also a fan of Docker containers, so I thought I would run everything in Docker. I wanted redundancy, so the drives should use RAID mirroring. I'm familiar with all these concepts, but I had not executed on them before. Perfect!

Each one of these topics warrants a small post. So I'm going to split each of them out and link them all here. Happy reading!

[Setting Up RAID 0 on Ubuntu 16.04 LTS Fresh Install aka "UEFI RAID Blows GRUB"](/2018/01/26/UEFI-RAID-Blows-GRUB)
[Setting Up a Bind9 Server in Docker (Coming Soon)]()
[Setting Up a Samba Server in Docker (Coming Soon)]()
[Setting Up a GitLab Instance in Docker (Coming Soon)]()
