---
title: Setup an Yggdrasil node
description: How to run your own Yggdrasil node
author: Tristan Brice Velloza Kildaire
---

This is a guide on how to create you own Yggdrasil node.

## Prerequisites

Before we get started it is important that you take a look at
first reading the setup process of a [802.11s node](/redes/802.11s/intro).
The important parts to take note of there are those pertaining
to the wifi radio and wifi interface configuration parameters
specific to the Yggdrasil experiment WiFi network.

## Specific

For the `wifi-iface` declared within this document pertaining
to client access, you may use any band and channel and even
SSID that you want. That's your personal choice.

We do recommend you name it something like `Novasredes (N: Y, T: CLIENT)`
but maybe something more basic and human readable would help.

## Yggdrasil - _the tree network_

Before we can begin configuring anything we first need to install support for the Yggdrasil routing protocol. This is done by installing the `yggdrasil` package with:

```bash
opkg install yggdrasil
```

We will also want the ability to perform configuration via UCI and have support for a new interface type known as `yggdrasil`, therefore we install the `luci-proto-yggdrasil` package:

```bash
opkg install luci-proto-yggdrasil
```

>Note: I had to reboot for the new protocol type to appear usable. I checked via LuCi just to confirm. Also thanks to [this guy's post](https://forum.openwrt.org/t/yggdrasil-network-connection/211884) for pointing it out.

### Generate some keys

Before we continue we need to generate both our *private key* and our *public key*. We can do both of these via the `yggdrasil` command.

The way I go about generating a *private key* is to generate a temporary configuration file in `/tmp/ygg.conf` with and from which I then grab the *private key*:
```bash
yggdrasil -genconf > /tmp/f.txt # generate new private key in config file
head /tmp/f.txt # get top of config file
```

Then to derive the *public key* you need to use the `-publickey` option and tell Yggdrasil which config file to use for obtaining the *private key* to derive from:

```bash
yggdrasil -useconffile /tmp/f.txt -publickey # derive public key
rm /tmp/f.txt # delete the config file
```

### Configuring an interface

We'll now create our `ygg0` interface which will provide our OpenWrt host with access to the Yggdrasil network. We name the *interface entry* `ygg0` as follows:

```
config interface 'ygg0'
	option proto 'yggdrasil'
    
	option private_key '<private key>'
	option public_key '<public key>'
	option node_info '{"name": "caldeira-map2", "contact": "deavmi@redxen.eu"}'
    
	list listen_address 'tcp://[::]:2000'
	list listen_address 'tls://[::]:2001'
```

A few things to note:
1. the *interface entry*'s name, `ygg0`, is what the name of the TUN adaptor will be
2. the `private_key` and `public_key` options must contain your *private* and *public* key pair
3. the `node_info` option allows you to place arbitrary JSON that can be requested by other nodes on the network via a *node info* request.
	a. It is convention to always fill in the `"name"` and `"contact"` fields with string-typed values as these allow you to be named on the [Yggdrasil map](https://github.com/Arceliar/yggdrasil-map) and the contact information makes it easy for others to get a-hold of you
4. the `list listen_address` can be appear **multiple times** and is for configuring the addresses you want to listen on for incoming peering connections (not from local discovery) *if any*

### Configuring beaconing and listening
We can also configure, as a seperate *yggdrasil entry*, what Ethernet interfaces we should ahve Ygdrasil peform link-local IPv6 multiocvast router discovery on.

The *yggdrasil entry* must be named as `yggdrasil_ygg0_interface` or more generally as `yggdrasil_<InterfaceNameHere>_interface` and will look something like this:
```
config yggdrasil_ygg0_interface
	list interface 'eth1'
	option beacon '1'
	option listen '1'
```

Some things to note:
- the `list interface` can appear **multiple times** for each interface you wish to apply this configuration to
- the `beacon` option means we will advertise our router on the provided links
- the `listen` option means that whenever we receive a beacon over *any* of the listed interfaces that we will consider it for connection.

---

One more thing to note, if you want to have different beaconing and listening policies for different interfaces then simply define another *yggdrasil entry* - and yes, the name must be the same (a duplicate)

In fact we *do* want to do this. I want to **also** have link-local multicast discovery happening over the `mesh0` interface - that's the whole point remember! Let's add another *yggdrasil interface* for that with:
```
config yggdrasil_ygg0_interface
	list interface 'mesh0'
	option beacon '1'
	option listen '1'
```

### Configuring Internet peers

I want to configure an uplink to the *wider Yggdrasil network*, we do this with a *peer entry* named in the form of `yggdrasil_<InterfaceNameHere>_peer` or in our case as `yggdrasil_ygg0_peer` which looks as follows:

```
config yggdrasil_ygg0_peer
	option address 'tls://[::1]:2112'
```

You may **also** define multiple *peer entries* but the name must be the same:

```
config yggdrasil_ygg0_peer
	option address 'tls://sto01.yggdrasil.hosted-by.skhron.eu:8884'

config yggdrasil_ygg0_peer
	option address 'tls://yggdrasil.deavmi.assigned.network:2001'
```

> **Note** Running `service network reload` doesn't seem to do it for yggdrasil, rather a restart is needed with `service network restart`.  

 
## Next steps?

That is enough to help join the Yggdrasil network and help
route traffic. The next step would be allowing people to
optionally access Yggdrasil _via_ your node.

For information on client access [read this guide](/redes/yggdrasil/client_access).
