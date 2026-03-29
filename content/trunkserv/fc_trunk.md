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

We have planned wireless PoP. 

TODO: Can we use `bridge-ap` (infrastructure mode?) This may not
be able to trnasfer - in fact **no** it _won't_ work for multi-mac
addresses behind a single station connecting to us. Therefore we 
need WDS mode.

# Trunking

## Transit network details (radio)

The radio information is very important, for the point of planning with building
owners and so fourth. This information pertains to the links used between sites
to create the coverage _rather_ than any radios that may be used as point-of-prescence
radios to link to individual people who want to try out the trunking service.


