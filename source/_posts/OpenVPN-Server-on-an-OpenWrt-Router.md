---
title: OpenVPN Server on an OpenWrt Router
date: 2020-01-13 09:03:51
---

I recently [set up an OpenWrt router in my house](/2020/01/08/setting-up-openwrt) and now I would like to get a VPN server running on it. Here are the steps I took to get a simple OpenVPN server running on my OpenWrt router using the [custom certificate authority (CA) that I set up](/2020/01/12/become-a-certificate-authority-with-easy-rsa). I'm running Ubuntu 19.10 on my development machine and OpenWrt 18.06.5 on the router.

## Goals and Assumptions

The goal is to run an OpenVPN server on an OpenWrt router that provides access to the local LAN.

### Assumptions

- The router has a WAN and a LAN (this is the default on a new OpenWrt installation)
- The LAN is on the subnet `192.168.20.0/24` (this is not the default on a new OpenWrt installation)
- The router is available at `192.168.20.1` to devices on the LAN

- The OpenVPN server will be available publicly on UDP port 1194
- The OpenVPN server will use a custom Certificate Authority as defined in the article [Become A Certificate Authority with Easy-RSA 3](/2020/01/12/become-a-certificate-authority-with-easy-rsa)
- The server certificate created for the OpenVPN server uses the common name `openwrt`

## Setting Up the OpenVPN Server

First up, the system needs certificates. Check out my article [Become a Certificate Authority with Easy-RSA](/2020/01/12/become-a-certificate-authority-with-easy-rsa). I'll assume all those steps are complete from here on out.

Before we actually SSH to the router, there's a simple step in the [Hardening OpenVPN Security](https://openvpn.net/community-resources/hardening-openvpn-security/) article provided by OpenVPN we can use to add an extra security layer: TLS auth. I chose to store the key generated from this procedure in my CA folder. The following will generate the TLS auth key. I ran it in my CA folder in the `pki` sub-folder.

```bash
openvpn --genkey --secret ta.key
```

Now it's time to install OpenVPN on the router. SSH to the router and run the following.

```bash
opkg update
opkg install openvpn
```

There are many tutorials that would have you also install the Easy-RSA packages and possibly OpenSSL. Since we generated our CA and our TLS auth key on a separate computer, there's no need to take up any space on the router with those packages.

We also want a place we can drop our keys that the OpenVPN server will use. Let's create a directory.

```bash
mkdir /etc/openvpn/keys
```

Next up, exit the SSH session and navigate to the CA folder we set up above. We'll need to copy the files OpenVPN needs to run. The system needs the certificate and key for the server `openwrt`, `openwrt.crt` and `openwrt.key`. We'll also need our CA certificate, `ca.crt`, the Diffie-Hellman key, `dh.pem`, the OpenVPN control list, `crl.pem`, and the TLS auth key we just generated, `ta.key`.

```bash
cd pki
scp ca.crt root@openwrt.example.lan:/etc/openvpn/keys
scp issued/openwrt.crt root@openwrt.example.lan:/etc/openvpn/keys
scp private/openwrt.key root@openwrt.example.lan:/etc/openvpn/keys
scp dh.pem root@openwrt.example.lan:/etc/openvpn/keys
scp crl.pem root@openwrt.example.lan:/etc/openvpn/keys
scp ta.key root@openwrt.example.lan:/etc/openvpn/keys
```

Time to SSH back to the router. I like to use a config file in the style OpenVPN publishes in their examples. I've seen many examples of configuring OpenVPN on OpenWrt that use the service definition to define the options. Instead, I like to use the service definition to point to an OpenVPN configuration file.

Let's create the OpenVPN configuration file first.

```bash
touch /etc/config/openvpn.conf
```

Using your favorite editor, open the file and add the following configurations.

```bash
port 1194
proto udp
dev tun
ca /etc/openvpn/keys/ca.crt
cert /etc/openvpn/keys/openwrt.crt
key /etc/openvpn/keys/openwrt.key
dh /etc/openvpn/keys/dh.pem
tls-auth /etc/openvpn/keys/ta.key 0
server 10.8.0.0 255.255.255.0
route 192.168.20.0 255.255.255.0
push "route 192.168.20.0.255.255.255.0"
push "dhcp-option DNS 192.168.20.1"

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
```

Now we need to connect the OpenVPN service to the configuration file. Open the file `/etc/config/openvpn` (notice the lack of extension) in your favorite editor and add the following.

```bash
config openvpn myvpn
  option enabled 1
  option config /etc/config/openvpn.conf
```

Restart the service and we should be in business.

```bash
service openvpn restart
```

## Firewall Setup

I'm going to handle the firewall setup using LuCI on OpenWrt. We'll set up a new interface, a new firewall zone, and a set of traffic rules that allow the outside world to access the VPN and users of the VPN to access the LAN.

Go to `Network > Interfaces` and create a new interface called `VPN` using the protocol `unmanaged` on the interface `tun0`.

Save & Apply

Go to `Network > Firewall` and create a new zone called `vpn` with `Incoming` and `Outgoing` accepted, but `Forwarding` rejected. Set the covered networks to `VPN`.

Save & Apply

Go to `Network > Firewall > Traffic Rules` and create the following rules:

```
Name: Allow-OpenVPN
Protocol: UDP
Source zone: WAN
Destination zone: Device (input)
Destination port: 1194

Name: VPN->LAN
Source: VPN
Destination: LAN

Name: LAN->VPN
Source: LAN
Destination: VPN
```

Save & Apply

That's it! Now the OpenVPN service should be available publicly on port 1194. Go create a client configuration and try it out. OpenVPN provides [an example client configuration in their GitHub project](https://github.com/OpenVPN/openvpn/blob/master/sample/sample-config-files/client.conf) that will help get you started.
