---
title: Install Docker CE on Elementary OS
subtitle: It's Ubuntu all the way down. . .
date: 2018-02-04 18:02:39
---

I just tried to install Docker CE on my new Elementary OS install (0.4.1 Loki) following the steps outlined on [the Docker docs web site](https://docs.docker.com/install/linux/docker-ce/ubuntu/) and ran into a small issue. The scripts provided use `lsb_relese` when setting up the apt repository. Sadly, there's no Loki-specific apt repository hosted at download.docker.com. Since this version of ElementaryOS is based on Ubuntu 16.04 LTS, I just hard-coded the command to add the apt repository to use `xenial` instead of `lsb_release` and kept on going. Here's the full script.

```bash
#!/bin/bash
set -e

##########################################################
# Install script for Docker-CE on ElementaryOS 0.4.1 Loki.
# Had to update the repository to point to xenial instead
# of using 'lsb_release -cs' because there's no loki
# repository at download.docker.com.
##########################################################

sudo apt-get update;

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common;

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -;

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable";

sudo apt-get update;

sudo apt-get install docker-ce;

sudo systemctl enable docker;

echo 'All done!'
```

I also have it hosted [as a Gist on GitHub](https://gist.github.com/BeerOnBeard/3828ab5c68fd77fdd8dc0d6c2079e245).
