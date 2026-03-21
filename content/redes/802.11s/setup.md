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

