---
title: 802.11s
description: Configuring hardware WiFI radios for 802.11s mesh operation
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

{{< image src="/hardware/ygg_mikrotik_map.webp" size="40%" caption="Cudy's AP300 router">}}
