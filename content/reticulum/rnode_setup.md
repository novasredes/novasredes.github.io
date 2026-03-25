---
title: Setting up an RNode
description: How to configure an embedded device to run the RNode firmware
---

## What is it?

An RNode is any [supported device](https://unsigned.io/guides/2022_01_25_installing-rnode-firmware-on-supported-devices.html) (see the “Device variations” section) that
runs Mark Qvist’s firmware.

What does the firmware let you do?

    LoRa sniffer
    LoRa TNC a. Think of a point-to-point IP link over LoRa b. This will be covered in part 2
    LoRa interface for a Retciulum node

You can see all of this in the “One tool, many uses” section [here](https://unsigned.io/hardware/RNode.html).

## Parts list

{{< image src="/reticulum/rnode/1.jpeg" caption="LillyGo T3-S3" size="40%">}}

{{< image src="/reticulum/rnode/2.jpeg" caption="LillyGo T3-S3" size="40%">}}

I decided to purchase 2 of the **LillyGo T3-S3**’s which make use of the SX1276 chipset.

* This is important as this is a chipset which has support in the RNode firmware.
* The model I am buying is the 868Mhz model as that was all I could get my hands on in ZA

I specifically bought 2 because I doubt there is anyone in my area running the RNode firmware and doing Reticulum over it. Secondly, even if there was someone then I wouldn’t have full control over my tests.

This device can be purchased locally [here](https://www.robotics.org.za/development-boards/esp32-lilygo-boards/H596).

