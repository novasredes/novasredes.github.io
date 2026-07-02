---
title: Yggdrasil network specifications
description: Network parameters for Yggdrasil operation
---

This page describes how to setup your WiFi radio
at both the physical layer and the WiFi MAC layer.

It is split into two sections for _client access_
and _mesh peering_.

## Mesh peering

### Physical layer

| Band          | Channel | HTMode  |
|---------------|---------|---------|
| 2.4Ghz (`2g`) | `13`    | Default (don't specfify) |

TODO: Can you set `htmode` to anything?: See [this](https://openwrt.org/docs/guide-user/network/wifi/basic#mac80211_options)

Ensure that both the `wifi-device` entries, once
configured, are _actually_ enabled by setting
the `disabled` option from `1` to `0`.

### MAC layer

| Band    | Mode   | mesh_id              | mesh_fwding | encryption | key                   |
|---------|--------|----------------------|-------------|------------|-----------------------|
| 2.4Ghz  | `mesh` | `NovasRedes YGGMESH` | `0`         | `sae`      | `theyggdrasilnetwork` |

You should name the `wifi-interface` something like
`wifi_24ghz_mesh_backhaul` and then also ensure the
`radio` parameter matches the radio configured in
the previous _"Physical layer"_ section.

Set `ifname` to the name of the interface for the
WiFi network you want - so that you can talk to
associated clients over it. This you can name
_anything_.

Note that the `mesh_id` must be as stated above.

## Client access

This is **optional** and only if you want to allow
clients to connect to your network.

### Physical layer

Uses the same physical layer as what you configured
earlier for _"Mesh peering"_. However you could use
a different radio if you saw fit.

### MAC layer

| Band    | Mode   | ssid                           | 
|---------|--------|--------------------------------|
| 2.4Ghz  | `ap`   | `NovasRedes Yggdrasil Network` |

You should name the `wifi-interface` something like
`wifi_24ghz_mesh_access` and then also ensure the
`radio` parameter matches the radio configured in
the previous _"Physical layer"_ section.

Set `ifname` to the name of the interface for the
WiFi network you want - so that you can talk to
associated clients over it. This you can name
_anything_.

Note that the `ssid` can be anything but making it
self-descriptive like above is helpful.

There is no password - this allows ease of connecting
without foreknowledge of the network.
