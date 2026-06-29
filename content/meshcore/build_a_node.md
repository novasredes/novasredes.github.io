---
title: Building a repeater
description: How to build your very own Meshcore repeater
---

## Introduction

The MeshCore network is made up of several different device roles that
are meant to do one thing and _do it well_. This very much follows the
UNIX philosophy paraphrased below:

>Software should do one thing and do it well

The roles are split into two main categories:

1. Repeaters
	* These are nodes acting as routers
	* These perform routing between _other_ routers and client nodes
2. Clients
	* Normal companion nodes
	* [Room servers](../room_servers)

We will be focusing on the _repeater_ role today as we want to be able
to build a node which will partake in the building and maintenance of
routes for the purpose of forwarding packets from one node to another
where an in-direct hop is the only possible path.

If you want to learn more about how routing works in Meshcore and what
functionality it offers then take a look at the [routing section](../routing).

## Building the node

We will be building a node that can be run outdoors. However, if
it needs to be run indoors and _with_ a grid-supplied electrical
connection then this guide will still work for you. This is because
the design integrates the housing for the battery, the embededded
device, electrical wiring and antenna wiring.

### Parts list

TODO: Turn this into a generic parts list which can be used for
building both Meshcore _and_ microReticulum

1. ABS outdoor case with pole mount
	* TODO: get name
2. WizBlock	4631
	* This is the base board
3. WizBlock 4630
	* This is the MCU which incoporates the CPU, storage, RAM
		and the LoRa modem

Below you can see what the WizBlock 4631 base board looks like
along with the WizBlock 4630 MCU screwed onto it. If you ordered
the "mesh starter kit" then I _presume_ (I don't know) the MCU
comes pre-screwed on for you - which is probably good as those
screws are incredibly tiny and difficult to screw on because
of their size.

We also have the uFL to SMA adaptor cable already connected to
the LoRa modem.

{{< image src="/hardware/wizblock/wizblock_4631.webp" caption="The WizBlock 4631 and 4630" size="20%">}}

And in case you wanted to know what the bluetooth antenna looks
like then this us it:

{{< image src="/hardware/wizblock/2_4ghz_bluetooth.webp" caption="The 2.4Ghz antenna" size="20%">}}

1. 868Mhz antenna
	* This is used for LoRa communications
2. uFL to SMA female adaptor
	* This is required because the on-board LoRa modem does not have
		an SMA connector but rather only a uFL connector
	* Link: (TODO: If the mesh starter it is used then this is included)
4. 2.4Ghz antenna
	* https://www.robotics.org.za/BAT-P60-WIFI-2?search=bluetooth
5. USB-C grommet
	* We want to be able to have a way to connect to the board without
		having to open up the case everytime just to be able to access
		the board's USB-C port. Therefore an extension cable like this,
		which exposes itself on the **outside** of the case, would work
		really well
	* Link: https://www.robotics.org.za/USB-C-IP65?search=usb%20c&page=4
6. (Optional) Solar panel
	* This will be connected to the USB-C grommet and used to
	supply USB-C-based power to the WizBlock 4631
	* **Optional** as you only need this if you are building
		a node not powered by mains
	* Link: https://www.robotics.org.za/6W-5V-USBC?search=solar%20panel
7. (Optional) 
	* A 5000mah battery should last long enough especially considering
		that we will be using a device known to have low power consumption.
	* A large capacity battery is also important for when the sun goes
		down and we no longer have any active power supply; we will want
		to be able to run through the night for as long as possible
	* **Optional** as you only need this if you are building
		a node not powered by mains
	* Link: https://www.robotics.org.za/955565?search=battery%205000mah

TODO: Add voltages for battery and solar panel

### The case



### Configuration



### The mast

TODO: Get the needed poles
