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

Lastly I place the `reticulum` container on the `retNet`
network. I also then setup the Docker TCP/UDP proxy to
bind on port `4242` for TCP and UDP and forward that
traffic (proxying) into the container at port `4242`
TCP/UDP as well:

```yaml
    networks:
      - retNet
    ports:
      - :::4242:4242/tcp # TCP server interface port
      - :::4242:4242/udp # UDP server interface port
```

These ports are what people will use when they want to
connect to my peer. The use case of that will be so
that they can make use of my node as a _transport node_.

### Config

You will want to create a file named `config` inside
of `RETICULUM_DATA_PATH`. The beginning of this config
we can set the configuration level:

```toml
[logging]
  loglevel = 4
```

#### Transport mode

The point of running our Reticulum node is to be a
router meaning that we will be able to transport
Reticulum packets on behalf of other peers. So
node `A` can forward a packet via node `B` (us)
and `B` can forward it to `C`. This means `A`
can reach `C` despite not having each other as
direct peers. However, `A` and `C` are both peered
with `B`. This is why we enable the _transport mode_
option:

```toml
[reticulum]
  enable_transport = True
```

#### Instance control et al.

We will want to have the ability to control
our instance for the sake of obtaining statistics
via tools such as `rnstatus` _or_ diagnostic
tools such as `rnpath`.

TODO: Double check this

```toml
  share_instance = Yes
  shared_instance_port = 37428

  instance_control_port = 37429
```

#### Interfaces

Next we will want to configure the various
interfaces over which we can connect to
other peers _or_ have them connect to us.

These peers will be either people we are
directly wanting to reach, maybe by request
of another peer forwarding a packet _via_ us
or - and this is the main focus - _other_
transport nodes who may be one hop closer
to the eventual destination.

The first one we setup will only make sense
in a Docker setup with more than one Reticulum
instance available on the same network; in
our case `retNet`. This is for automatically
discovering LAN peers via link-local multicast.
We will be enabling this so that the Reticulum
instance in the `lxmd` service can discover
us - the transport node:

```toml
[interfaces]
  [[LAN Multicast interface]]
    type = AutoInterface
    enabled = yes
```

I also want to accept inbound peerings. This
is useful as it allows people to per their
normal node (or transport node) with mine
if they know my domain/address and port:

TODO: Enable UDP port (or fix it)

```toml
  [[TCP Inbound server]]
      type = TCPServerInterface

      enabled = yes
      listen_ip = ::
      listen_port = 4242

  [[UDP Inbound server]]
      type = UDPInterface
      enabled = no # FIXME: Enable once fixed
      listen_ip = ::
      listen_port = 4243
```


## LXMD configuration (optional)

This step is optional but worth setting up as it is rather
useful. LXMF is the messaging (like IM) layer that runs on
top of Reticulum.

It's what people use for messaging one another and supports
various message types. It _also_ provides a way to place
messages on a _propagation node_ `A` which will sync with
other propagation nodes, until the message is on all propagation
nodes (TODO: is that rigth?). Then the recipient user can
sync with his propagation node (hopefully if he has one
configured) and grab that message. In affect it is useful
for messaging when the recipient is offline.

The LXMD daemon allows one to setup a messaging endpoint
but _also_ allows, amongst such features, for one to run
a propagation node. Such a service adverises a Reticulum
_destination_ with certain metadata which allows clients
to understand that it is offering a _**propagation**
node service_.

Let's first define the basics of the service:

```yaml
...

  lxmd:
    container_name: lxmd
    restart: unless-stopped
```

Next thing is that I want this service to depend on the
`reticulum` service so that this service (`lxmd`) only
starts up _after_ `reticulum` has started. This is not
a hard requirement but the main reason I have done it
is because if it starts earlier then it will send the
announcement for its _propagation node service_ to all
interfaces and since `reticulum`'s `rnsd` is not running
yet it won't reach the wider network (as `reticulum`
has all the upstream peers). Now it will _eventually_
reach the wider network when `lxmd` broadcasts/announces
its service at its interval (which is configurable).

```yaml
    depends_on:
      - reticulum
```

We specify the source used to build the Docker image.
The definition thereof can be found [here](/reticulum/lxmd_image).

```yaml
    build:
      context: ../../../images/lxmd
```

We set a start-up sleep time in much the same way as
we did for the `reticulum` service:

```yaml
    environment:
      # AutoPeering over link-local didn't work
      # if you didn't make startup wait a little
      # bit (clearly for some Docker networking interfaces
      # to properly initialize)
      - SLEEP_TIME=5
```

As for storage, this one is quite important as you
will be storing a lot of files which will contain
message data pending to be synced with other propagation
nodes. The storage path can be configured inside
your `.env` file as the interpolation variable
`LXMD_DATA_PATH`.

The path for storing the Reticulum data, important
to be persisted as this is how the _destination_
address will be derived, is set via the `LXMD_RNS_PATH`
interpolation variable.

```yaml
    volumes:
      # Configuration and storage for lxmd
      - ${LXMD_DATA_PATH}:/data:z

      # Local RNS instance configuration path
      - ${LXMD_RNS_PATH}:/rns:z
```

Lastly, we set the effective user and group
via the interpolation variables:

1. `USER_UID`
2. `USER_GID`

```yml
    user: ${USER_UID}:${USER_GID}
```

We also place it onto the `retNet` network
which is the same network that the `reticulum`
container is on. This is so that they can
discover one another via link-local multicast
automatic peering - hence making our LXMF
propagation node service available _via_
our Reticulum node's uplink.

```yml
    networks:
      - retNet
```

### Config

Lastly you will want to place a file named
`config` inside of your `LXMD_DATA_PATH`.
The contents should be:

```toml
[logging]
  loglevel = 4
```

This sets the logging level. You can make it
lower but it does help to see what is going
on with your propagation node. Such as, who
it is peering with in order to sync messages.

The next options are to enable the propagation
node feature (as mentioned `lxmd` can do many
things and this is just one of its features).

We also control how frequently announcements
of the propagation node service _destination_
should be sent out. We also ask it to announce
upon startup of the `lxmd` process:

```toml
[propagation]
  enable_node = yes

  # Announce propagation node destination
  # every 4 hours
  announce_at_start = yes
  announce_interval = 10
```

We want to automatically peer with other
propagation nodes. This is done by picking
up on similiar announcements coming through
our peering interface (setup earlier).

```toml
  # Peer automatically with other
  # propagation nodes
  autopeer = yes
```

Lastly we set some limits on the syncing
process. These are pretty self-explanatory.

```toml
  # Peer with other propagation nodes
  # that are at maximum 10 many hops away
  autopeer_maxdepth = 10

  # Maximum size of a message
  # that we will accept to
  # be propagated to us
  # I set mine to 1MB
  propagation_transfer_max_accepted_size = 1000

  # Storage limit for
  # messages saved to store.
  # I set mine to 10 Gigabytes
  message_storage_limit = 10000
```
