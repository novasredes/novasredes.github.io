---
title: Reticulum LoRa network
description: Learn about our LoRa-based network based on Reticulum
---

Nos provedemos "Reticulum network" access. With this we obviously want
to make use of alternative infrastructure that can run, _independemente_,
from existing infrastructure. One such technology that provides:

1. Long distance
2. Low bandwitdh

Is LoRa. That's a great physical layer but what about the network layer?
Well, that's where Reticulum comes in (a project o senhor has personally
worked on) - it provides a link-heterogeneous routed network for running
anything - voice included!

## Node configuration

As it currently stands, the RNode parameters we are using are as follows:

```
[[RNode test]]
type = RNodeInterface
enabled = yes
port = /dev/rnode1

txpower = 15
frequency = 868000000
bandwidth = 62500
spreadingfactor = 8
codingrate = 6
```

Your `port` will differ most likely as it is device enumeration dependent,
you should probably use something like `udevd` to match the device's Plug-and-Play
data against certain commands - such as device renaming - if you want to have
a predicatble name. I don't know if `udevd` can do that but I would not be
surprised if it could.

However the important _radio_ aspect to have set in **common** are:

1. `frequency`
2. `bandwidth`
3. `spreadingfactor`
4. `codingrate`

These effect how the radio is tuned and the range around the frequency
it cares about. Every other option is PHY-and-up configuration. The
`txpower` is how much _poder_ to push through the radio's antenna.

The `codingrate` and `spreadingfactor` are LoRa-specific but also
must be the same.

## Apps

You can use any Reticulum app that supports pairing with an RNode,
these include:

1. `rnsd`
	* Paired with this to `nomadnet` and you have a terminal-based client
2. [Columba](https://columba.network/)
	* Android app (seems partially AI-coded) which uses the native `reticulum-kt`
		library
	* This app is VERY nice
3. Sideband
	* Like Columba, but runs the `reticulum` Python reference implementation
	* Not that nice UX but it does work; works slow though - noticable UI lag

## Native hardware

If you have a T-Deck or some supported device then take a look at
[RatSpeak](http://ratspeak.org/) which is a firmware that can _easily_
be loaded onto your device and has Reticulum support.

Fully off the grid, no yog-slop Android needed.

See a [video of it in action](https://www.youtube.com/watch?v=F6I6fkMPxgI).
