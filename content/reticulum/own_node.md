---
title: Creating your own node
description: How to create your own RaspberryPi-based Reticulum node with LoRa capability
---

## Prerequisites

This setup will be using Docker so ensure you have that installed on your system.
After doing so you will then want to update your Docker configuration at `/etc/docker/daemon.json`
and adding the following to the file:

```json
{
  "experimental": true,
  "ip6tables": true,
  "ipv6":true,
  "fixed-cidr-v6": "fd10:c8c6:3b63::/48"
}
```

After doing so, save it and then restart Docker in order to apply the new configuration.
If you are on a systemd-based system then use:

```bash
sudo systemctl restart docker
```

Make sure all is fine by running and checking the status:

```bash
sudo systemctl status docker
```

## RNode configuration

TODO: Add this - but put it on a seperate page
but with a link here

## Reticulum configuration

TODO: Add this

## LXMD configuration (optional)

TODO: Add this into its own page
