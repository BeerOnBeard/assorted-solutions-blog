---
title: Setting up a Bind9 Server in Docker
subtitle: Ever want your own top-level domain?
date: 2018-02-11 17:55:23
---

As part of my [home server setup](/2018/01/21/Home-Server-for-Fun-and-TLD), I also wanted the ability to set up my own DNS entries for services I was running. Even better, I wanted my own top-level domain. Why? Why not?

I found an Alpine Linux Docker image that ran only Bind9 that I could mount my own config file to. Thanks to user resystit on Docker Hub for putting up the image [resystit/bind9](https://hub.docker.com/r/resystit/bind9/). The beauty of this image is that it doesn't do anything besides run Bind9 and it's on Alpine Linux. It's only 5MB!

Most of my time was spent researching how to configure Bind9 in the first place. After loads of research, I decided that I wanted a forwarding nameserver that would cache results locally (for super-fast DNS lookups after the initial hit) that I could also define my own zone on. First, let's talk about what that means.

A forwarding nameserver that caches results does two things. First, if the local server cannot find a DNS entry for a request, it forwards the query to upstream servers that I configured and asks them to find the DNS entry for the query. Another option exists where the local server will follow the tree of nameservers and keep asking upstream until it finds an answer, but forwarding allows me to offload that work to the upstream servers.

I also wanted to define my own zone that I could add my own DNS entries to for locally running services. In this case, I started with GitLab, a Samba server, and my router. The zone file looks a lot like the text equivalent of how I configure my DNS entries with Google Domains. I defined a nameserver, some A records, and some CNAME records to point DNS entries for services to one of the A records that represents a physical IP on my network.

It took a while to come up with named.conf settings that worked for me, so I'll walk through them now to hopefully help someone out there in the future.

I started with obfuscating the version returned when a system requests it. Version is not really needed and is a great way for attackers to find out what version of Bind9 I'm running and then google attack vectors. Instead. . .

```na
version "pound bricks";
```

I used auto DNSSEC validation because it's easy mode. Bind9 comes with a built-in copy of the [root zone key signing key](http://www.root-dnssec.org/) to validate DNS query results that are returned from upstream servers. Easy mode sounds good to me!

```na
dnssec-validation auto;
```

The next few lines are for DNS lookup forwarding. I used Google's DNS servers because they are highly-available, unlike my internet provider's.

```na
# allow this system to lookup and cache results
recursion yes;

# use forwarders to offload compute requirements to upstream servers
forwarders {
  8.8.8.8;
  8.8.4.4;
};
forward only;
```

Then, to protect myself from attacks from the outside, I restricted the computers that are allowed to query my DNS server. The address and mask I used here (100.100.100.0/24) is not the one I used in my configuration file. You would need to replace that will your local network address and network mask for this to work for you.

```na
allow-query { 100.100.100.0/24; };
```

The last line in the options section, that I configured, was to tell the server that there's no master/slave situation going on. My little DNS server is all alone. If there is a need for high-availability, I highly recommend redundant servers in master/slave to reduce downtime. In this case, I don't need the complexity. Worst case scenario, I update everyone in my house to use my router's DNS server instead of my own. I lose my top-level domain, but we can still do work and access services from the greater internet.

```na
allow-transfer { none; };
```

Now done with the options, I moved onto the zone definition. I wanted my own top-level domain, so I set it up like so. The name of the top-level domain has been changed. I recommend you do the same. It doesn't even have to be English. Mine wasn't :)

```na
zone "mytld." {
  type master;

  file "/etc/bind/mytld.zone";
};
```

The `file` directive points to a file that defines my zone, NS, A, and CNAME records. I put that file on my host and mapped that volume into the container. I wanted the configuration file to live on even if I killed my container and recreated it. The contents of the file would be something like this.

```na
$TTL 60
@         IN    SOA   mytld. root.mytld. (
                      2     ; Serial
                      60    ; Refresh
                      60    ; Retry
                      60    ; Expire
                      60 )  ; Negative Cache TTL
;

@         IN    NS    server.mytld.

server1   IN    A     100.100.100.200
server2   IN    A     100.100.100.201

service1  IN    CNAME server1
service2  IN    CNAME server1
service3  IN    CNAME server2
```

In the above case, I have a top-level domain for mytld. I set the time-to-live to a short 60 seconds to allow me to easily and quickly re-point services to different IPs without the computers on my network pointing to the old IPs for a long time. I defined two A records for server1.mytld and server2.mytld at 100.100.100.200 and 100.100.100.201, respectively. I then set up CNAMEs for three services. service1 and service2 are hosted on server1 and service3 is hosted on server2. Let's assume the three services are websites. I would use the following URLs to access each service: https://service1.mytld, https://service2.mytld, https://service3.mytld.
