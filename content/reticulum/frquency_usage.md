---
title: Frequency plan
description: The frequency plan for this service
---

This is the frequency plan for the Reticulum service.

| TX power | Frequency  | Bandwidth |
|----------|------------|-----------|
| 15			 | 890000000  | 62500			|

## Why `890mhz?`?

The center frequency of `890mhz` because this reduces
collisions with Meshtastic and Meshcore devices. This
is a gentleman's agreement simply because I don't want
to stump on their low-traffic communications with the
significantly higher Reticulum traffic.

### Testing results

The way I tested this was by generating Meshcore traffic
(by sending messages on a channel) and then monitoring
the `rnsd` daemon (with `-vvvv` flags to increase the
logging verbosity).

When doing this I got a message saying "Radio detected
interferance".

{{< image src="/reticulum/general/interface_collisions.webp" caption="RNSD reporting interface collisions" size="20%">}}

**After** shitfing the LoRa parts of my network over
to the `890mhz` center frequency those messages drastiscally
dropped in their rate even when spamming the Meshcore network
(still running at `868mhz`).
