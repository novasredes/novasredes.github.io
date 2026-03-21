---
title: Yggdrasil client access
author: Tristan Brice Velloza Kildaire
description: A guide on how to configure a hotspot with SLAAC and DHCPv6 for client access
date: "2026-03-21"
---

This section should look at client access
- DHCP usage via the `/64` or via some NAT
- DNS that access Yggdrasil and clearnet
- Adblocking value-add

Both of the following can be done so long. But the Ethernet one requires you have at least **two** WiFi interfaces.

### Creating a bridge

In order to simplify the setup process specifics we create a bridge and then enslave one or both of the WiFi and Ethernet interfaces to it. From_here_ we then perform the required network configuration as then it isn't tied to any of the one slave interfaces.

Let's add the following entry in `/etc/config/network`:

```
config device 'accessbr0'
        option name 'accessbr0'
        option type 'bridge'
        list ports 'eth0'
        list ports 'accesswifi0'
```

This will define a bridge interface named `accessbr0` and then will enslave the following interfaces:
- `eth0`
- `accesswifi0`

Only add a `list ports` entry for the interfaces you have/want bridged (take note of the opening paragraph of this section).

### On ethernet

Really it is just added as a slave interface to the `accessbr0` bridge so nothing needs to be done.

### On wifi

What we will be doing is using the *same* `wifi-device` or *WiFi radio* and adding an **additional** `wifi-iface` entry that *uses that `wifi-device`*.

So therefore, let's add an entry like follows to our `/etc/config/wireless`:
```
config wifi-iface 'access_radio0'
	option device 'radio0'
	option ifname 'accesswifi0'
	option mode 'ap'
	option ssid 'Yggdrasil Access Node 2'
	option encryption 'sae'
	option key 'HateTheStateMate69'
```

Some of the parameters are:
- `encryption` is set to `sae` for the standard encryption mechanism. It's the same one as used for the 802.11s (mesh) `wifi-iface`.
- the `key` is then the actual passphrase
- We set the `mode` to `ap` as we want to create an access point (and not a mesh point like we did earlier)
	- The `ssid` is the name that will be advertised for our WiFi network
- the `ifname` is the network interface name that will be created when this interface is brought up and will be how we send and receive traffic over this WiFi network
- lastly, we specify which `wifi-device` or *radio* to use for this. We have set it to the same radio we used earlier; `radio0`.

This is all that is needed to be done.

