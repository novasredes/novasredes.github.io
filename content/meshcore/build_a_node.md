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
	* [Room servers](meshcore/room_servers.md)

We will be focusing on the _repeater_ role today as we want to be able
to build a node which will partake in the building and maintenance of
routes for the purpose of forwarding packets from one node to another
where an in-direct hop is the only possible path.

## Path management

TODO: Play this in some other section

The nice thing about Meshcore is that one can override the dynamically
discovered next-hop router used in path forwarding **on** the client
node _itself_. So if I am node `A` and want to send a message to node
`D` and I have two intermediary nodes:

```
	-------- B --------
 /                   \
A                     D
 \                   /
  -------- C --------
```

One possibility is that your client node `A` will choose the next-hop
`B` in order to get your packet to client node `D`. This means your
path will, in effect, be `A -> B -> D`.

The interesting thing about Meshcore is that it will still allow you
to choose your next-hop. So although it configured `B` as the next-hop
in order to get to the destination `D`, you _can_ manually override
this and select `C` as the next-hop. The only condition is that `C`
is a router that your client is aware of - meaning you have received
an announcement of its prescense (the same applies to any next-hop
including that of the dynamically selected one).
