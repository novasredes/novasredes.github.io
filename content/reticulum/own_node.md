---
title: Creating your own node
description: How to create your own RaspberryPi-based Reticulum node with LoRa capability
---

## Prerequisites

This setup will be using Docker so ensure you have that installed on your system.
After doing so you will then want to update your Docker configuration at `/etc/docker/daemon.json`
and adding the following to the file:

```json
{
  "experimental": true,
  "ip6tables": true,
  "ipv6":true,
  "fixed-cidr-v6": "fd10:c8c6:3b63::/48"
}
```

After doing so, save it and then restart Docker in order to apply the new configuration.
If you are on a systemd-based system then use:

```bash
sudo systemctl restart docker
```

Make sure all is fine by running and checking the status:

```bash
sudo systemctl status docker
```

## RNode configuration

You will first need to flash the RNode firmware onto your device.
A [guide](/reticulum/rnode_setup) has been created for that.

## Reticulum configuration

### Preparation

I will be configuring both contains:

1. `reticulum`
	* The Reticulum protocol router
2. `lxmd`
	* The LXMF propagation node daemon

... to both be placed on the same IPv6 network. This network
will make use of IPv6 NAT handled by Docker _however_ it does
imply that you would need to have an IPv6 network that your
Docker host is connected to.

If you do not want this then define the `retNet` network
below using IPv4 rather:

```yaml
networks:
  retNet:
    driver: bridge
    name: retNet
    enable_ipv6: true
    ipam:
      config:
        - subnet: fde4:492b:fc7f::/48
```

You will see later why it is that we created a custom network
as we will want the `lxmd` container ton be on the same LAN
as the `reticulum` one so that **both** their Reticulum
instances can automatically peer with each other.

### Basics

The most basic configuration is to tell the container
how its image will be built. In my case I have the image
defined [elsewhere](/reticulum/reticulum_image) which
you can take a look at if you wish to build using my
image.

We also set the `restart` policy so that it always
restarts if, for example, the container's process crashed.
However, if we manually stopped the container and restarted
our Docker host then the container will remain stopped.
This is what the `unless-stopped` option does.

```yaml
services:

  reticulum:
    container_name: reticulum
    restart: unless-stopped
    build:
      context: ../../../images/reticulum
      args:
        # This platform cannot use
        # the crypto library, hence
        # we must use the `rnspure`
        # package
        - PURE_INSTALL=false

        # If IPv6 should be preferred
        # whenever getaddrinfo() is
        # used
        - GAI_PREFER_IPV6=true

		...
```

We also want to use the version of Reticulum that uses
the native cryptographic libraries. These libraries only
run on 64Bit hosts. If you have a 32Bit host then you can
set `PURE_INSTALL=true` as that will then rather install
`rnspure` which uses hand-rolled cryptographic primitives
instead but which at least run on 32Bit hosts.

The next option, `GAI_PREFER_IPV6=true` controls how domain
names are resolved, typically via the LibC `getaddrinfo()`
function. If a domain has both an `A` and `AAAA` record then
you can control which one has preference. For some reason
I actually needed to add support for this as I was getting
odd behavior where the resolution process was actually
preferring `A` records over their `AAAA` counterparts (when
the latter was available).

### Environment variables

There are some environment variables that we should be
setting. Mainly one of them pertains to the custom image
I built.

```yaml
    environment:
      # AutoPeering over link-local didn't work
      # if you didn't make startup wait a little
      # bit (clearly for some Docker networking interfaces
      # to properly initialize)
      - SLEEP_TIME=5
```

One of the problems I had noticed was that if the
`rnsd` process started up too quickly when the container
had started then there would be certain networking
issues that occured. Specifically the link-local
aspect of the network interface didn't work correctly.
This was a fatal problem because I _need_ this to
be working for the `reticulum` and `lxmd` containers
to see one another for automatic peering via the
`AutoInterface` (it uses link-local multicast to
perform this).

Therefore the simplest hack was to introduce a
startup sleep time. Here I have that configured
to 5 seconds with `SLEEP_TIME=5`.

### Volumes and users

```yaml
    volumes:
      # Reticulum configuration and storage directory
      - ${RETICULUM_DATA_PATH}:/data:z
```

The Reticulum daemon stores a lot of local data, pertaining
not only to the identity it generates on first start (which
isn't much data) but also to the peers it has learn of, these
are stored so that there is a cache of information that can
be used and looked up from. This obviously needs to be stored
somewhere and hence I allow the user, via a `.env` file entry,
to set this by setting the `RETICULUM_DATA_PATH` interpolation
variable.


I also expose, in a similar manner, the effective user ID and
group ID for the process to run as. This is normally to be set
to the uid/gid of a user you have access to on the host-side.
Making thse match means the file metadata (available on the
bind-mount) matches a uid/gid pair that you have access to -
meaning you can **easily** manage the files.

```yaml
    user: ${USER_UID}:${USER_GID}
```

In have exposed these as interpolation variables:

1. `USER_UID`
2. `USER_GID`

### Devices

Optionally, if you have a device you want to make available (from
your host) and in your container, then this is where you would do
that.

The device in specific that I am interested in forwarding is the
device connected at the _device node_ of `/dev/ttyACM0`. I know
for a fact that (an `udev` rules aside, as I haven't made any
that _could_ be used to give it a deterministic name) my RNode
is always connected at that port. I therefore forward it to
be available at `/dev/rnode1` with the `rwm` rights. I presume
`r` is read, `w` is write and `m` is (manage?).

The next thing you will want to do is also to add an additional
_supplemantry group_ to the user the container's process is
running as. This is group with ID 20. This is because the
forwarded device inherits the ownership metadata as it is
on the host side. On the host side `/dev/ttyACM0` is owned
by `root` (uid 0) and group `dialout` (gid 20). Therefore
for the process running on the _container side_ to be able
to access such a device node (a file) it either needs to be
root (which we _know_ it isn't as we have set our own
custom effective UID). Therefore the only thing we can do
is to place ourselves in the group with gid `20` - this will
give us the rights to be able to access said device at
`/dev/rnode1`.

```yaml
    group_add:
      # On container-side the device node
      # (for the RNode) will be mounted with
      # GID 20. Hence ensure we add that to
      # the effective GID supplementary set
      # so that we have permissions to interact
      # with that device
      - 20
    devices:
      # Add RNode devices here
      - /dev/ttyACM0:/dev/rnode1:rwm
```

### Networking

TODO: Do this

```yaml
    networks:
      - retNet
    ports:
      - :::4242:4242/tcp # TCP server interface port
      - :::4242:4242/udp # UDP server interface port
```

## LXMD configuration (optional)

TODO: Add this into its own page

```yaml
  lxmd:
    container_name: lxmd
    restart: unless-stopped
    depends_on:
      - reticulum
    build:
      context: ../../../images/lxmd
    environment:
      # AutoPeering over link-local didn't work
      # if you didn't make startup wait a little
      # bit (clearly for some Docker networking interfaces
      # to properly initialize)
      - SLEEP_TIME=5
    volumes:
      # Configuration and storage for lxmd
      - ${LXMD_DATA_PATH}:/data:z

      # Local RNS instance configuration path
      - ${LXMD_RNS_PATH}:/rns:z
    user: ${USER_UID}:${USER_GID}
    networks:
      - retNet
```
