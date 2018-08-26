---
title: How I Pay Less for Hosting on Amazon
subtitle: tl;dr Lightsail is down to $3.50 per month for a small Linux VPS.
date: 2018-08-25 17:24:12
---

I've been paying between $10 and $12 per month for a t2.micro EC2 instance for a while now. It has a single, virtual core, 1GB of ram, and 10G of storage. It was the cheapest thing I could find on AWS at the time and it has worked fine. However, there's a new product offering that can help reduce my bill and make it a static cost. Amazon Lightsail offers a VPS with a single, virtual core, 512MB of ram, and 30G of storage for roughly a third of the price of my t2.micro, $3.50. I had to take a look to see if it would meet my needs.

The sites I run are all containerized and I have a setup script that bootstraps a new server when I need to spin one up. To test, all I had to do was go to the Lightsail console, ask it to spin up a new instance for me, and SSH to it with the new SSH key pair that Lightsail provided. I chose the Ubuntu 16.04 LTS image, so after a quick `apt update` and `apt dist-upgrade`, I could run my install script and have a new instance of my production system ready to test.

There were a few differences between managing my EC2 instance and the new Lightsail instance. There's a completely different console for Lightsail that has far fewer options. For someone looking for simplicity, Lightsail is probably the way to go. No more separation of instance management, Security Group management, storage management, etc. in a huge menu of options. There are a few tabs on the instance management page in Lightsail that put it all together for you.

Additionally, Lightsail does not provide the same functionality that I had for managing inbound traffic through security groups. For example, in EC2 I used Security Group Inbound Rules to restrict what IP ranges could access port 22. In Lightsail, I can only enable or disable the port to the internet. Now I'll need to leverage `iptables` on each instance to make sure that only specific IP ranges can access specific ports.

I also was not able to reuse my existing SSH keys, even though I spun up the instance in the same region as my existing security groups and key pairs. The Lightsail console has a section on the Account page that allows me to manage a new set of SSH key pairs that are specific to Lightsail instances.

For me, the differences are minor and do not affect me too much. After running basic tests, my sites performed fine. I updated my DNS entries and waited for the global DNS propagation. I'm now running on Lightsail and saving some money. The race to the lowest price VPS continues!
