---
title: FC Trunking
description: Servicio de Trunking por Feri Caneca (routed Ethernet with POPs)
---

## Trunking - _O que esta_?

Imagine you have two endpoints that you want to connect and you want to move
Ethernet frames from one point-of-prescene to another point-of-prescene. You
_also_ want to have it be as transparent as possible - meaning things such
as VLANs (as many as you want) should be able to _ingress_ at the one POP
and _egress_ at the other and vice-versa.

This is what novasredes' trunking service aims to offer.

# POPs

TODO: These still need to be decided on

# Trunking

## Transit network details (radio)

The radio information is very important, for the point of planning with building
owners and so fourth. This information pertains to the links used between sites
to create the coverage _rather_ than any radios that may be used as point-of-prescence
radios to link to individual people who want to try out the trunking service.

### _em_ Worcester

These are the parameters used within Worcester.

For the 2.4Ghz network:

* WiFi operation modes used: `B`/`G`/`N`
* Band: 2.4Ghz
* Chosen frequency: 2467
* Channel width: 20Mhz
* `txpower` is locked to software's "regulatory domain" for the chosen region
	* Region: `southafrica`

For the 5Ghz network:

* WiFi operation modes used: `A`/`N`
* Band: 5Ghz
* Chosen frequency: 5600
* Channel width: 20Mhz
* `txpower` is locked to software's "regulatory domain" for the chosen region
	* Region: `southafrica`
