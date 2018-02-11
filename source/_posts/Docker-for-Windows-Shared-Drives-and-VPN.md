---
title: Docker for Windows, Shared Drives, and VPN
subtitle: Mounting shared drives in a hostile VPN environment
date: 2018-02-11 17:33:26
---

My team at work started working with Docker for Windows (version 17.12.0-ce-win47) recently. All of my side projects have been based in Linux, so it took a little bit to get used to working with it in Windows. Once I got the system running, I used PowerShell to run all my normal commands. Pretty cool!

One of the issues I hit when setting up the system was sharing a drive while using a VPN connection to my company's network. When I tried to enable drive sharing to enable volume mounts on containers, I received an error from the Docker for Windows system that [pointed me to their docs site](https://docs.docker.com/docker-for-windows/#firewall-rules-for-shared-drives). It sent me searching for firewall rules that would allow the system to work. However, when I went looking to add the required firewall rules, there were no issues. . .

That sent me on a wild goose chase that ended up coming back, once again, to the way my company's VPN was set up. The default network that Docker for Windows uses is 10.0.75.0 with a mask of 255.255.255.0. It turns out that the default network conflicts with my company's VPN. Once I updated the subnet address, I was able to successfully map the shared drive!

I ended up choosing a 192.168.x.0 subnet address that did not conflict with the existing networks that I usually connect with. The common networks seem to be 192.168.0.0 and 192.168.1.0 for most home networks, and 192.168.50.0 for some others (like newer Asus routers). Time to spin up some containers!
