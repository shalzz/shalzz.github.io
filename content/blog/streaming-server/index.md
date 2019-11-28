+++
layout="post"
title="Home Media Streaming Server"
date=2019-11-28
draft=true
[extra]
tags=""
+++

### Background and motivation

Used to be that Netflix was the only streaming service who's subscription you needed,
sparing sharing with your friends and family providing access to every video content out there.
Long gone are the good ole days. If you haven't noticed now-a-days there's a different streaming
service by every big producer and publisher with exclusive content as the only selling
point to drive adoption.

For some it's perfectly reasonable subscribing to every service exclusively streaming
the new TV series. For other's it the constant juggle between one or two services every month.

<!-- more -->

If like me you don't have any qualms with piracy and are safe from law enforcement
then you might say there's an easy solution.

This fragmentation of the streaming market it the single biggest reason for the
subsequent re-rise of the piracy industry. (citation)

Pirating leads to problems of it's own, that of storage space. Deleting old files to make space for
new downloads, being tied down to a single device, scrubbing to 
exactly where you left off an episode on another device after taking the pains to 
copy the gigabytes of data between devices.

This inspired me to come up with an alternative that solves most if not all
of the paper cuts from pirating.

### Abstract

This is an overview of how you can effectively have your own streaming server 
with zero running cost, setup at your home with consumer hardware you most likely
already own allowing you to stream your entire media library via your WI-FI 
connection when your at home and through a [Wireguard] VPN connecting to you home
router when you on the move.

### Prerequisites

Most of the hardware used here is pretty standard and should be easy to acquire.
But this whole setup is contingent on having a router with [OpenWRT] installed
or one that is supported by the OpenWRT project and you are not afraid of tinkering
with you router to install OpenWRT on it yourself.
Besides the hardware requirements general familiarity with *nix systems and
command line would make this very easy for you. 

### Disclaimer

This is not going to be a step-by-step tutorial since the underlying technology
and the CLI is constantly evolving and changing rather I'm going to just
describe the architecture and link to the OpenWRT wiki where necessary.

### Ingredients/Requirements

The actual hardware requirements are:
* A recent router with at least one USB port and OpenWRT installed.
* A portable USB hard-drive/SSD or a NAS box with USB connection.
* An Ethernet cable in case you mess up your WiFi/firewall config.

Software
* Samba server
* Samba Client
* filesystem ext4
* wireguard peers
* transmission
* DDNS

### Architecture

<figure style="width:75%;margin:auto;">
    {{ resize_image(path="Streaming Server@2x.png") }}
<figcaption style="text-align:center;">Fig 1: Network Architecture</figcaption>
</figure>

### Resources

[Wireguard]: https://wireguard.com
[OpenWRT]: https://openwrt.org
