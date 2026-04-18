---
title: Architecture of trunkserv
description: How the FC-trunking service works
---

This document will become available at a later stage. The basic idea I have
is to combine the following protocols into a cohesive whole:

1. Ethernet-based link layer technology
	* Some method of linking networks into a LAN
	* Wirelessly and wired
2. Spanning tree protocol
	* A way of ensuring a loop-free switching network
3. Multiple VLAN Registration Protocol (MVRP)
	* A way to ensure that traffic flooded in a given VLAN only
		reaches the switches that are known to have ports members
		of such VLANs _or_ only reachable via said switch
4. 802.1q
	* VLAN support
	* The ability to do VLAN tag stacking which is _crucial_ for supporting
		customers who want to have their virtualized links acts as transparent
		Ethernet links - which _includes_ being able to use whatever VLANs
		they want
