---
title: Two Routers the Easy Way... After Failing Hard
subtitle: DD-WRT For Me
date: 2017-08-18
---

## What was I thinking?

I had been toying with the idea of putting a second router on the network for years. You are probably familiar all the different forms of procrastination I used, so I'll just skip that portion and get to the meat. I wanted to try DD-WRT because I had heard so many good things from my networking buddies. It is supposedly fairly reliable and opens up lots of possibilities with your network. Awesome!

After some initial research, I decided to go with the [Wireless Distribution System (WDS)](https://www.dd-wrt.com/wiki/index.php/WDS) pattern. It promised to allow me to extend my network with an extra router wirelessly. No extra CAT5 cables running through the house. Just Internet everywhere. Little did I know, I was an idiot for thinking life could be so easy.

## Supplies

My guinea pig for DD-WRT. The glorious D-Link DIR-601!

![D-Link DIR-601](/images/DLinkDIR601.jpg)

My current router/provider of Internet goodness. The ZyXEL VMG4380-B10A.

![ZyXEL VMG430-B10A](/images/ZyXELVMG4380.jpg)

Stop judging me for my DSL connection . . . and of course

![Google Logo](/images/googlelogo.png)

## The Install

The hardest part of the install of DD-WRT was finding the firmware to install onto my router. When I tried searching for my router model with the keywords "DD-WRT" and "firmware", I was brought to a landing page that looked legit. However, all the download links failed. Long story short, I finally found the [DD-WRT Router Database](http://dd-wrt.com/site/support/router-database). After a quick search for "dir-601", the firmware I needed was right there!

WARNING: Do not go to "other downloads". It's a trap.

![It's A Trap!](/images/ItsATrap.jpg)

From the table at the bottom of the page, I selected the row "DIR-601 A1 Firmware - Webflash image for first installation". There was another row for the upgrade firmware, too. At the time of writing this, no one has updated this specific firmware since 2013. I'm guessing I'm never going to get to use the second row for upgrades...

The DD-WRT site has a page specifically for [flashing the D-Link DIR-601](http://dd-wrt.com/wiki/index.php/DIR-601). This is the first page I stumbled upon when I was looking for firmware. The download link is dead. Thumbs down.

Step 2 tells you to hold down the reset button for 30 seconds to restore factory defaults before flashing on the new firmware. **Do not hold down the reset button for 30 seconds.** It puts your router in [firmware recovery mode](https://superuser.com/questions/809333/d-link-blinking-orange-light-and-cannot-access-router-ip). Firmware recovery mode is a bit more complicated and totally unnecessary for what we're doing here. Instead, hold down the reset button for 10 to 15 seconds. Once you see the light change from orange to yellowish, you win. If you do get into firmware recovery mode, just unplug, take a sip of coffee, and plug back in. Then hold down the button for 10 to 15 seconds, you overachiever.

After following the steps (except for step 2... overachievers), I had a shiny new DD-WRT install to play with! After clicking around a bit and looking at the fancy graphs for CPU utilization, RAM usage, and network saturation, I moved on to actually accomplish something.

## My Downfall

I'm going to preface this section with the following: read the fucking manual.

As noted at the beginning of the journey, I had decided to use WDS to fill in some Internet gaps in my life. DD-WRT's page on [WDS Linked Router Network](https://www.dd-wrt.com/wiki/index.php/WDS) is an awesome resource for setting up a network in this way. I followed the instructions to the letter, had everything running correctly on each router, and went for the final connection that would fulfill my life's dream of having internet in my backyard. And then nothing happened.

I thought to myself, "I must have done something wrong.". I'm not a network engineer. I really have no idea what I'm doing. I'm mostly just a monkey following instructions. I took a peek around the control panels in both routers and eventually sat down to try it again. I found issues that would prevent my system from working. For example, the instructions tell you to set preamble to "short". Sadly, my crappy little ZyXEL does not support a short preamble. I thought I was making progress!

After hours of resetting the D-Link and trying again, I finally had an epiphany. Well... I finally read the one paragraph that was my downfall. The third section of DD-WRT's page on [WDS Linked Router Network](https://www.dd-wrt.com/wiki/index.php/WDS) is a note about WDS and different chip manufacturers. In my excitement, I had only checked the chip that was on my D-Link. I had completely foregone checking the chip on the ZyXEL. The D-Link uses a Qualcomm Atheros. The ZyXEL uses a Broadcom. There is no supported, secure way to make these two talk using WDS. Crap.

## The Recovery

After cursing myself for a while, I had to devise another plan. I had some Powerline adapters kicking around that I had purchased a couple years ago when I needed to set up a (now retired) server that did not have a wireless card.

![Trendnet Powerline Adapter](/images/TrendnetPowerline.jpg)

Then I remembered that Scott Hanselman wrote a blog a while back about [configuring two wireless routers with the same SSID](https://www.hanselman.com/blog/ConfiguringTwoWirelessRoutersWithOneSSIDNetworkNameAtHomeForFreeRoaming.aspx). Thank you, Scott! I reset the D-Link back to a fresh install of DD-WRT. I plugged in those powerline adapters. I followed the instructions Scott wrote and I was in business! I could walk around my property and my phone would switch from router to router as the connection from one overtook the other. No re-authentication. Just a quick blip when the switch occurs.

## RTFM

Did I really need to do this? Not really. But it was great fun hacking on a router and actually accomplishing something. Mostly, this served as a reminder of how important it is to read thoroughly before wasting tons of time on something that is (very clearly documented as) not going to work.

