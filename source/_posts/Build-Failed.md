---
title: Build Failed!
subtitle: Raspberry Pi Build Notifier and Scrum Board
date: 2018-06-01 22:03:59
---

![Build Failed!](/images/build-failed.jpg)

I recently started working on a new team within the same company. Some are new hires, some are veterans, all are ready to get into team mode and get some stuff done. We recently went out on a team bonding adventure to a bowling alley / arcade and found some neat prizes that sparked an idea.

As we were counting our tickets and looking at the (mostly cheap-o) prizes, a couple of flashy siren lights caught our eye. I mentioned that it would be cool to hook those up to a Raspberry Pi and make them go off when the build failed. Two of my new teammates thought that would be awesome and promptly grabbed two of them and bought them with their tickets.

Now I was on the hook. :)

I had a Raspberry Pi 3 B+ that my brother had bought me for Christmas sitting in the closet waiting for a project. I had opened it up on Christmas, set up Linux, and attached some lights that I could blink via SSH commands . . . then it sat for a while. This was the perfect opportunity to play.

Luckily, the Raspberry Pi has some sockets that are available in a Linux installation that can be set up easily and play nice with `echo` commands from bash. Here's how easy it is to set a pin to high.

```bash
# Set up the pin you want to work with
echo "21" > /sys/class/gpio/export

# Set that pin as an output
echo "out" > /sys/class/gpio/gpio21/direction

# Set the pin to high
echo "1" > /sys/class/gpio/gpio21/value
```

Setting the pin to low is as easy as `echo "0"` to that value. Cake!

Now I needed to `curl` a response from our TeamCity build server and check for a `SUCCESS` or `FAILURE` value on specific builds. As a side note, I only set this up on the master branch. These lights would be going off constantly if we were alerting on our feature branch builds!

```bash
curl -i -H "Accept: application/json" http://teamcity.example/guestAuth/app/rest/builds/buildType:<buildType>,branch:%3cdfeault%3E | grep -Po '"status":\K"[A-Za-z0-9._]*"'
```

The `<buildType>` needs to be replaced by something from the [TeamCity API docs](https://confluence.jetbrains.com/display/TCD10/REST+API). They're not the easiest to follow, but the definitely have the information in there. The most useful section for me was [the section on Build Locator](https://confluence.jetbrains.com/display/TCD10/REST+API#RESTAPI-BuildLocator). The bit with `branch:%3cdfeault%3E` is only encoding `<default>` so I can target only the default branch: master.

I could then take the status from the TeamCity and set the pin to high or low. Here's what the final result looked like:

```bash
#!/bin/bash
status=$(curl -i -H "Accept: application/json" http://teamcity.example/guestAuth/app/rest/builds/buildType:<buildType>,branch:%3cdfeault%3E | grep -Po '"status":\K"[A-Za-z0-9._]*"');

if [ $status = "\SUCCESS\"" ]; then
  echo "0" > /sys/class/gpio/gpio21/value;
else
  echo "1" > /sys/class/gpio/gpio21/value;
fi;
```

I set it up as a crontab job that runs every five seconds. Since crontab can only go as granular as one minute, I used a loop and sleep.

```bash
* * * * * for i in {1..12}; do /usr/local/bin/build-notifier.sh; sleep 5; done
```

Now I needed a relay to turn on and off depending on a pin state. Since this is going to be at work, and I'd rather not be responsible for burning it down if my soldering skills failed me, I opted for a packaged controller I found on Adafruit. The [controllable four outlet power relay module V2](https://www.adafruit.com/product/2935) came to the rescue. I could make a relay for much cheaper, but the peace of mind for something at work is well worth the $25 + shipping.

![Controllable four outlet power relay module V2](/images/power-relay.jpg)

I decided to take a Friday afternoon at work and get the machine running. I brought in my Raspberry Pi, some wires, and the relay, installed Ubuntu Core, and had a working light show in a couple hours. It was a big hit and we had beers.

![Stock photo of beers and cheers](/images/beers-stock-photo.jpg)

"But what about the TV in the picture at the top?", you ask. Well, that was part two.

Not content with only running some lights (and having an old TV taking up space in my basement) I decided to set up a rotating board of team status. We use Atlassian's Jira for task tracking at work. Jira has a dashboard feature with widgets that you can set up in a rotating slideshow that looks decent in a browser when you hit F11 (full screen).

I set up a few dashboards that showed the scrum board, the days until the end of the sprint, and a feed of activity in our Jira project. I installed Ubuntu Mate on the Raspberry Pi in order to have a desktop and everything went smoothly... for a while...

...until I decided to update the distribution to all the latest packages. Apparently, I picked the perfect time to upgrade and get a version of Firefox that is not compatible with the latest Raspberry Pi. Suuuuuper lame. I installed the Epiphany browser in hopes that it would "just work". Epiphany kind of works, but it's definitely a hog on that tiny machine. I plan on installing a different OS soon and getting back to Firefox.

All in all, great fun project. It was fairly straightforward and a great talking piece at work. I highly recommend it.
