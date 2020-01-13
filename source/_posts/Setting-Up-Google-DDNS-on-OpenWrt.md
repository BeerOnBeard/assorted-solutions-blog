---
title: Setting Up Google DDNS on OpenWrt
date: 2020-01-13 12:09:46
---

I recently set up [a new router with OpenWrt](/2020/01/08/setting-up-openwrt) and [installed an OpenVPN server on it](/2020/01/12/openvpn-server-on-an-openwrt-router). In order to easily connect to that OpenVPN server when I'm on the move, I need to set up dynamic DNS support. I use Google to host my domains and I'll use that to host my dynamic DNS that points to my house.

First, a synthetic record must be set up in Google Domains. Let's assume we want to set up the hostname `myhouse.example.com`. Follow the [instructions provided by Google support to set up a Dynamic DNS synthetic record](https://support.google.com/domains/answer/6147083?hl=en).

Now we're ready to set up our OpenWrt router. I'll be using LuCI to setup and configure the system.

Go to `System > Software`. We need to install `luci-app-ddns`, `ca-certificates`, and `libustream-openssl`. `luci-app-ddns` provides a GUI for us to configure DDNS in LuCI. `ca-certificates` contains the certificate authorities used to check the authenticity of SSL connections. `libustream-openssl` is the SSL library used by the DDNS service to create an SSL connection.

Once installed, a new menu item `Services > Dynamic DNS` will become available. You may need to refresh the browser to see it.

Go to `Services > Dynamic DNS` and add a new entry named `MyHouse`. In the `Basic Settings` tab, check the Enabled box. Set the lookup hostname to `myhouse.example.com`. Set the DDNS Service provider to `google.com`. Set the domain to `myhouse.example.com`. Fill in the username and password fields with the username and password provided by Google Domains. Check the Use HTTP Secure checkbox. Set the Path to CA-Certificate to IGNORE.

Save & Apply and we're off to the races.

Navigate back to the `Services > Dynamic DNS` menu item to get to the Overview screen. If the Process ID column has a Start button, click the start button to start the service. Check Google to make sure the IP configured matches the IP given to you by your ISP. Enjoy your new DNS entry pointing to your house!

If the system doesn't seem to be working, check the Log `File File Viewer` tab on the details page for `MyHouse`. Get there by going to `Services > Dynamic DNS` and clicking the Edit button on the `MyHouse` entry.
