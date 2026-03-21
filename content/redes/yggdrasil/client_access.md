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

