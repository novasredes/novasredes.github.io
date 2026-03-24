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
