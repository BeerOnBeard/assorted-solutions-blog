---
title: OpenVPN Server on a Raspberry Pi 1 Model B
date: 2020-01-17 09:33:22
---

I recently set up an [OpenVPN server on an OpenWrt router](/2020/01/13/OpenVPN-Server-on-an-OpenWrt-Router/). It worked until I put too many processes on that poor little machine. It only has one CPU core running at 480MHz and 64MB of RAM after all. So I decided to grab a Raspberry Pi 1 Model B that was gathering dust in my closet and put it to work.

This version of the Raspberry Pi uses an ARM v6 chip. Unfortunately, operating systems like Ubuntu Core are only available for the ARM v7 chips that are on the Raspberry Pi 2 and later editions. I decided to use a Debian-based operating system called [DietPi](https://dietpi.com/) that runs on ARM v6 chips and claims to be very efficient. I flashed the operating system to a SD card and stuck it in the Raspberry Pi.

I ended up installing, re-installing, ruining the networking stack, re-installing, and so on for a while. I ended up with a tiny machine that hosts an OpenVPN server that I can access from anywhere in the world. Here's what I found worked after all those attempts.

DietPi uses Dropbear as the SSH server. Although it is lightweight and works great for SSH access, it is not compiled with `scp` support. I used `scp` to transfer certificates and keys from [my certificate authority](/2020/01/12/Become-A-Certificate-Authority-with-Easy-RSA/) to the Raspberry Pi. I decided to install OpenSSH and ditch Dropbear for this reason. DietPi makes choosing a different SSH server easy during the initial setup.

![DietPi Software](/images/dietpi-software.jpg)

DietPi provides the ability to install and setup an OpenVPN server for you. However, it automatically creates the certificates and keys without asking. I tried using the provided OpenVPN installer during one of my attempts to set up this machine and waited over an hour for the Diffie-Hellman-Merkle key to be generated. In my article [Become A Certificate Authority with Easy-RSA 3](/2020/01/12/Become-A-Certificate-Authority-with-Easy-RSA/), I already generated all the certificates and keys I need on a fast computer that generated that Diffie-Hellman-Merkle key in seconds. For that reason, I skipped using the GUI to install OpenVPN and instead used `apt`.

To setup, SSH to the system and run the following:

```bash
apt update
apt install openvpn iptables

# IP forwarding rules
cat > /etc/sysctl.d/local-openvpn.conf << EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
net.ipv6.conf.default.forwarding=1
EOF

sysctl net.ipv4.ip_forward=1 net.ipv6.conf.all.forwarding=1 net.ipv6.conf.default.forwarding=1

# iptables rules
iptables -A INPUT -i tun+ -j ACCEPT
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -t nat -F POSTROUTING
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE

iptables-save > /etc/iptables.up.rules
cat > /etc/network/if-pre-up.d/iptables << EOF
#!/bin/sh
/sbin/iptables-restore < /etc/iptables.up.rules
EOF

chmod +x /etc/network/if-pre-up.d/iptables
```

The above script does two main things besides installing OpenVPN and iptables; it enables IP forwarding and sets up iptables rules that allow VPN clients to access the LAN. IP forwarding is disabled by default on DietPi and many other operating systems. Running an VPN server that allows access to the local LAN is one of the reasons we would enable this feature. Placing a file in `/etc/sysctl.d/` ensures these settings are configured as we want on every boot. iptables allows us to configure the firewall to allow traffic from the VPN subdomain, 10.8.0.0/24, to be forwarded onto the LAN. The scripts using `iptables-save` and `iptables-restore` ensure that the rules are added every time the network is brought up. iptables rules are not persistent. Placing an executable file in `/etc/network/if-pre-up.d/` ensures that the rules we saved to `/etc/iptables.up.rules` will be loaded whenever an interface comes up.

Before we move on to the VPN setup, this is a good time to add the certificates and keys from the certificate authority to the Raspberry Pi. The server will need the following files from the `pki` folder:

- ca.crt
- crl.pem
- dh.pem
- issued/server.crt
- private/server.key

I added those files directly to `/etc/openvpn` on the Raspberry Pi. Doing so allows us to use relative paths in the server configuration file. We'll also want a `ta.key` file for TLS-auth. Generate it with:

```bash
openvpn --genkey --secret ta.key
```

SSH back onto the Raspberry Pi to finish setting up the system.

```bash
cat > /etc/openvpn/server.conf << EOF
port 1194
proto udp
dev tun
ca ca .crt
cert server.crt
key server.key
dh dh.pem
tls-auth ta.key 0
server 10.8.0.0 255.255.255.0
push "redirect-gateway"
push dhcp-option DNS 192.168.1.1"

# Network topology
# Should be subnet (addressing via IP) unless windows
# clients v2.0.9 and lower have to be supported (then
# use net30, i.e.a /30 per client)
# Defaults to net30 (not recommended)
topology subnet

# Maintain a record of client <-> virtual IP address
# associations in this file.  If OpenVPN goes down or
# is restarted, reconnecting clients can be assigned
# the same virtual IP address from the pool that was
# previously assigned.
ifconfig-pool-persist ipp.txt

# The keepalive directive causes ping-like messages
# to be sent back and forth over the link so that
# each side knows when the other side has gone down.
# Ping every 10 seconds, assume that remote peer is
# down if no ping received during a 120 second time period.
keepalive 10 120

# Reduce the OPenVPN daemon's privileges after initialization.
# The persist options willy try to avoid accessing certain
# resources on restart that may no longer be accessible
# because of the privilege downgrade.
user nobody
group nogroup
persist-key
persist-tun

# Output a short status file showing current connections,
# truncated and rewritten every minute.
status openvpn-status.log

# Set the appropriate level of log
# file verbosity.
#
# 0 is silent, except for fatal errors
# 4 is reasonable for general usage
# 5 and 6 can help to debug connection problems
# 9 is extremely verbose
verb 3

# Notify the client when the server restarts so it can
# automatically reconnect.
explicit-exit-notify 1
EOF

systemctl stop openvpn
systemctl start openvpn
systemctl start openvpn@server
journalctl -u openvpn@server # check the logs to make sure everything went smoothly
```

The above script makes some assumptions that you may need to change.

- `port 1194` is the default port for an OpenVPN server
- `push "dhcp-option DNS 192.168.1.1"` assumes that your LAN network DNS provider (probably your router) is available at 192.168.1.1

The Raspberry Pi system is now ready to roll. The router that is connected to the ISP will need a hole poked in it to allow traffic on the configured UDP port. How to do that is dependant on the router and the management software provided. Port forwarding is a popular mechanism to do this and is the way I went.

Happy tunneling!
