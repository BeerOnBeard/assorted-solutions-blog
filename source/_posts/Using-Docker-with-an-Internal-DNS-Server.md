---
title: Using Docker with an Internal DNS Server
subtitle: Passing Corporate or Personal DNS Servers Into Docker
date: 2018-01-14
---

A few months ago, I was working on a build pipeline for GitBook based on Docker. The basic concept was to have a Linux build agent that would build Docker containers and push them to a repository. As I was building that scripts that would build the build agent on a local VM, I ran into an inconsistent error that turned out to be DNS resolution. When the issue hit, it looked like the build system could not locate a resource on the network. I would SSH to the system and ping and find that I could not resolve the host name. However, I could ping the IP directly. Long story short, if I built the VM using my scripts while I was connected to the VPN, the system would function fine. If I built the VM when I was not connected to the VPN, but then later connected to the VPN, the system would not know how to resolve hosts on my corporate network.

As it turns out, Docker passes the DNS servers that are registered on the host into the Docker system for name resolution. When I spun up a new VM without a connection to my corporate network, the corporate DNS servers were not registered on the VM. I found a way to specify the DNS servers Docker should use to resolve DNS queries. I would not do this in a production environment because it would add an extra layer of complexity to the system that should not be needed. I only used this in my VMs to ease development as I connected and disconnected from many different networks and VPN tunnels. My production system was spun up on the correct network, received the correct DNS servers from the get-go, and did not need to "travel" to different networks on a whim.

OK! With the back-story and warning out of the way, let's see how to hack this system into submission!

First, we need to know the DNS servers we want to configure. Make sure the host is connected to the network you want to grab the DNS servers from and run the following commands. On a Windows host, you can run 

```cmd
ipconfig /all
```

to show all the connections. If you're on Linux, you can use the Network Manager CLI tool.

```bash
nmcli d show | grep 'IP4.DNS'
```

The previous command will show device information and filter the stream down to only the DNS servers that are configured.

I had 5 DNS servers configured: 3 internal, 2 external. We can take this information and update Docker's daemon.json file to use these DNS servers in containers. Update or create the file `/etc/docker/daemon.json` with the following entry:

```json
{
  "dns": ["192.168.2.1", "192.168.2.2", "8.8.8.8", "8.8.4.4"]
}
```

The first two are completely made up. Those should be replaced with the DNS servers for your internal network. The second two might be familiar. 8.8.8.8 and 8.8.4.4 are Google's public DNS servers. They will be used as a fallback if the first two servers do not provide anything useful.

Once the files is updated or created, restart the docker service.

```bash
sudo system restart docker
```

Then you can test to make sure the changes made it in. Alpine Linux is super light (5MB) and can be used for a quick test.

```bash
sudo docker run -it alpine
```

Once you have a terminal, you can run the following inside the running Docker container:

```bash
cat /etc/resolv.conf
```

You should see all the DNS entries you configured. While still on the terminal inside the running Docker container, we can run some tests to make sure we configured everything correctly:

```bash
ping google.com -c2
ping some-internal.source-control.server -c2
```

The first command will ping Google to see if it can be resolved. The second command should be updated to ping a local resource by DNS name.

Since everything came back AOK, we're ready to keep on hacking on our VM! Time to finish scripting out our build server before deploying to production!
