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

