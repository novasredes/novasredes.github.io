---
title: 802.11s
description: Introduction to 802.11s
---

This is a guide on how to setup a device for 802.11s operation. It firstly
explains _what_ 802.11s mode is and then, when we reach the stage on configuration,
also goes into detail about each parameter does.

This document will show you how to perform this on any device that can run OpenWRT
21 and up.

## Hardware

SOme of the devices you can get will vary by brand, but most importantly is that
they can run at the very least OpenWRT 21. Secondly, you will want a device with
at least **32MB** of flash - maybe even more (if you are reading this as a prerequisite
for setting up some _other_ protocol to run ontop of 802.11s then please consider
at least **64MB** of flash).

{{< image src="/hardware/ygg_cudy_ap300.webp" size="40%" caption="Cudy's AP300 router">}}

{{< image src="/hardware/ygg_mikrotik_map.webp" size="40%" caption="Mikrotik mAP">}}

## What is 802.11s or _mesh mode_?

The word "mesh" is thrown a lot depending on where you are reading about it,
it's often used to describe the topology of nodes to one another and can technically
be present at most levels of the layered networking model.

When we talk about mesh with regards to WiFi we are referring to either one
of the following technologies:

1. Ad-hoc mode
	* This mode was very popular back in the day but it seemed to either never
		be properly standardised and hence was only available on certain hardware
2. 802.11s mesh mode
	* The 802.11s standard introduced a stdandardised mesh mode that basically
		did what the ad-hoc mode.
	* It has additional functionality such as _forwarding_ via the `mesh_fwding`
		option that you will see later; however we won't be using that

I had found [this](https://wireless.docs.kernel.org/en/latest/en/users/documentation/modes.html#ad-hoc-ibss-mode) page whilst writing it which at least confirms the concept
of these two as being distinct.

The mesh mode allows one to configure every radio on a certain channel
and SSID such that they will see all traffic on that WiFi network appear
on their `wifi1` interface. The neat part is that there is no dedicated
access point and hence no central point of failure - no one node is
responsible for broadcasting the "network". But all nodes can see the
traffic of all neighbouring nodes.

As soon as a new device is in range, it "joins" the network. There is
no handshake as there is no central point to handshake with, you just
start transmitting. The same applies for reception of frames, as soon
as something is received on that channel+SSID then you accept it.

The main take-home point is that this results in one big Ethernet
segment containing Ethernet frames from Ethernet devices in the 
immediate radio vicinity of one another. 

The `mesh_fwding` option allows optional forwarding _via_ one of the
nodes in the case where you have `A <-> B <-> C` and `A` cannot see
`C`, it's frames will be forwarded _via_ `B` and vice-versa.

## Why is this a prerequisite?

We want to build mesh networks that require no central authority
at **any** layer of the OSI model. It's all good and well that
we have sub-projects such as:

1. [Yggdrasil](/yggdrasil)
<!-- 2. TODO: Add FIPS here -->

Where these are running at a layer 3 level. However, having just
layer 3 operate in a decentralized manner is not good enough if
the underlyring media access layer - layer 2 - requires us to
have a central access point that other _wireless-based_ nodes
connect to. If that access point (AP) goes down then the remaining,
what would be client nodes_ won't be able to form an Ethernet
broadcast domain (network) **despite** being able to otherwise
see each other as they are in radio range.

This is what 802.11s fixes.

## The setup process

The documentation on setting this up was written about a year ago,
I **highly doubt** much has changed since then. I also wanted to
keep it as a seperate document from this one as this one is more
of a rough introductory document.

Therefore if you want to jump straight into the technical documents
then I recommend that you [do so now](../setup).
