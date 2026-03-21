---
title: 802.11s setup procedure
description: Configuring hardware WiFI radios for 802.11s mesh operation
author: Tristan Brice Velloza Kildaire
date: 2026-03-21
---

## Cleaning up

The initial configuration is not something that we want to build ontop of. We first need to clear some firewall rules, decouple some enslaved interfaces and then configure the ones we want.

We do all of this before we move on to the actual configuration of our device.

Before we start, ensure that your Ethernet cable is still plugged into the first ethernet port on your mAP. This port is what is referred to as `eth1`, the one next to it (rather confusingly), is named `eth0`.

>Note: I guess it isn't all that confusing, it depends on whether you are Portuguese or Arabic (but probably both)

### Clearing firewall rules

First let's clear out all but the bare minimum for our firewalling rules by editing the `/etc/config/firewall` and setting it just to:

```
config defaults
 	option input 'ACCEPT'
 	option output 'ACCEPT'
 	option forward 'REJECT'
 	option synflood_protect '1'
```

> Note: We disable forwarding as we won't be doing traditional IP forwarding with Yggdrasil - that is part of the Yggdrasil process itself. We do enable `OUTPUT` and `INPUT` for obvious reasons.  
> Note: he **whole** file must be *just* the above. The default OpenWRT rules are many and they are annoying so make sure to remove them **all*.  

You can now reload the firewall service with:

```bash
service firewall reload
```

### Removing the bridge and associated configuration

By default there will be a bridge device, named `br-lan`,which enslaves both `eth1` (which we are plugged into) and `eth0` (the other port).

We want to find the *device entry* that is for a bridge device named `br-lan` and we want to entirely remove it, it normally looks something like this:
```
config device
	option name 'br-lan'
	option type 'bridge'
	list ports 'eth0'
	list ports 'eth1'
```

We also want to remove the *interface entry* that is used to configure static networking configuration on the bridge, this will be an *interface entry* named `lan` and should appear as follows:
```
config interface 'lan'
	...
```

Remove it entirely. We don't want that.

> Note: I have not yet restarted the `network` service (which is responsible for this) as that would cut our connectivity off. We will do it later.  

### Removing the pre-configured DHCP server

Now let's open up the `/etc/config/dhcp` and there should be two *DHCP entries* named `lan` and `wan`, we want to remove both of these. They normally appear like this:

```
config dhcp 'lan'
	option interface 'lan'
	option start '100'
	option limit '150'
	option leasetime '12h'
	option dhcpv4 'server'
	option dhcpv6 'server'
	option ra 'server'
	option ra_slaac '1'
	list ra_flags 'managed-config'
	list ra_flags 'other-config'

config dhcp 'wan'
	option interface 'wan'
	option ignore '1'
```

## Setting up what we need

We now start to configure some additional things we will want.

### `eth1` - *our __uplink__*

We want to have `eth1` as our uplink port or *Wider area network* (WAN) port. This means several things but what concerns us now is that this port should be used tio receive an IPv4 or IPv6 (or both) address and network configuration from the network it is connected to on that port.

To do this we leave the `config device` option for `eth1` as is - that just configures the MAC address which is fine as is. If, however, a MAC address is **not** present then it is good to configure one statically else a random one is generated every time.
```
config device
	option name 'eth1'
    option macaddr '<mac here>'
```

> **Note:** You definitely will want a static MAC address if you enable `dhcpv6` in the upcoming section and want your SLAAC-derived IPv6 address to remain stable.  

What we want to do is to create a two new *interface entries* which will describe how to configure `eth1` in various ways. We create one that grabs an IPv4 network conmfiguration (if possible) with:

```bash
config interface 'wan4'
	option device 'eth1'
    option proto 'dhcp'
```

And then we also add another one to account for if the upstream provides any IPv6 network configuration with:

```bash
config interface 'wan6'
	option device 'eth1'
    option proto 'dhcpv6'
```

### Hostname

To make things clearer let's edit the hostname. This can be done by editing the `/etc/config/system` fiel and updating this line:

```
config system
	option hostname 'Node_2'
```

To apply the change run:

```bash
service system reload
```

## Moving over

We now have our basic configuration setup. Let's reload the services for the files we changed:
1. `/etc/config/dhcp`
	- Is the configuration used by the `dnsmasq` service (and also `odhcp`)
2. `/etc/config/network`
	- Is the configuration used by the `network` service

We can reload both of them with:

```bash
service dnsmasq restart && service network restart
```

> Note: It seemed to take *forever* to get a DHCPv6 lease.  

- NOW Fix this or check if it still occurs

**Your connection will now HANG!**

Now plug your computer into your home network (one with a DHCP server) and then also plug the Ethernet cable currently plugged into `eth1` on the mAP *also* into your home network. We're going to be working on our mAP from our home network from here forward.

> Note: To find your OpenWrt device you can use a network scanner  

TODO Honetsly for a noob this may be too much

