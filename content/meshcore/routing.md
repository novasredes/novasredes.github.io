---
title: Routing
description: How routing works in Meshcore
---

## Path management

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
