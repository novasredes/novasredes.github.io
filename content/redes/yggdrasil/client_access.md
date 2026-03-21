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

### Configuring the bridge

We now want to configure the `accessbr0` bridge with IP addressing so that it can serve the Yggdrasil `/64` prefix. This prefix lives in the `300::/8` subnet (it is a subset of it).

In order to configure our _bridge device_ we need to add a new entry in our `/etc/config/network` in the form of an _interface config_ like so:

```
config interface 'accessBridge'
	option device 'accessbr0'
	option proto 'static'
	option ip6assign '64'
	list ip6class 'ygg0'
```

The options are:

1. We set the `device` to `accessbr0` so as to state that we want to configure the addressing of _this_ interface
2. Then we will be using _static addressing_ on this interface as we won't be _receiving_ any DHCP in order to configure the addressing, therefore we set the `proto` option to `static`
3. Lastly we choose _how_ we are going to assign the static addressing. We don't want to manually set it to Yggdrasuil `/64` prefix as that may change if we ever regenerate our Yggdrasil's public-private key pair. So then - is there anyway in order to get such addressing information **dynamically** from what the Yggdrasil daemon's current keypair is?
	- Well, yes! Infact the `luci-proto-yggdrasil` actually hooks into the UCI system in order to advertise an IPv6-PD (prefix delegation). What this means is that our prefix will be placed in the prefix pool which then any interface can grab at in order to have it applied as their addressing info.
	- The `ip6assign` means that from the pool we want to grab any prefix from the pool that matches the given prefix size, in this case `/64` which we **know** is what will be made available from the `ygg0` interface via the `luci-proto-yggdrasil`
		- TODO Clarify how it works on non-exact matches
	- The `list ip6class` entry is used to filter down the prefix pool search query down to prefixes being delegated by the given given interface. This is useful so that we don't grab any _other_ `/64` that may be being advertised by interfaces other than our `ygg0`.

### Configuring SLAAC

TODO The parameters section may as well be split into mega-sections as each of these deserves having its own explanation

Our `accessbr0` interface has addressing information assigned to it already via the IPv6-PD (prefix delegation) that we sourced from the `ygg0` interface. However, this doesn't mean anything for people connecting to this interface.

For that we want to enable SLAAC on `accessbr0` so that this prefixing information and some (additionally configured) routing information can be advertised to connecting clients.

In order to configure SLAAC we must open the configuration file that deals with DHCP, RaDV etc. . This file is located at `/etc/config/dhcp`. We must create a new entry in it as follows:

```
config dhcp 'accessSlaac'
	option interface 'accessBridge'
	option start '100'
	option limit '150'
	option leasetime '12h'
	option ra 'server'
	list ra_flags 'none'
	option ra_default '2'

	list dns '324:71e:281a:9ed3::53'
	list dns '302:db60::53'
	list dns '300:6223::53'
	list dns '302:7991::53'
```

We must specify the _interface config_'s name that we wish to enable RaDV (router advertisements) for, in this case it would be `accessBridge` (see the earlier section for this).

We set the `ra` option to `server` as we want to be serving RaDV, as compared to relaying RaDV from other routers

We then set the `ra_flags` to  `none`, this is important because by default it is set to `other-config` which is a mode whereby it will advertise options to the SLAAC client such as an external DHCPv6 server to use. We **obviously** don't want this as we want the client to be forced to use SLAAC and not some yet-to-configure DHCPv6 server.

You should then see something like this (if you restart with `service dnsmasq restart`):

{{< image src="/yggdrasil/network_manager_initial.png" caption="kak" size="30%" >}}
