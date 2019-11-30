+++
layout="post"
title="Home Media Streaming Server with Wireguard and OpenWRT"
date=2019-11-28
draft=true
[extra]
tags="openwrt, wireguard, streaming, video, media, home media, samba server, samba, torrent, transmission"
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

### Architecture

Our Software stack would comprise of mainly these three main technologies.
* Samba server: Filesystem over the Network.
* Wireguard peers: A secure VPN
* libtransmission: A torrent client
* DDNS: Dynamic DNS to remotely access our home router

<figure style="width:75%;margin:auto;">
    {{ resize_image(path="Streaming Server@2x.png") }}
<figcaption style="text-align:center;">Fig 1: Network Architecture</figcaption>
<br/>
</figure>

#### Samba

The premiss of our setup is having a Network Attached Storage (NAS). Off the shelf
NAS boxes are generally more expensive but they do allow you setup hard-drives
in a RAID configuration providing redundancy and backup. If you already have a 
NAS box you can use it instead otherwise if you're not doing any content creation
and don't need high write throughput then a single drive is more than enough.

There are two main filesystem-over-the-network solutions
* NFS: the Linux native Network File System.
* Server Message Block (SMB): A widely supported protocol with Microsoft's
  own implementation for Windows.

Since SMB is a more widely supported protocol and we need to be able to access our
NAS from any client device including Windows and MacOS we will be using Samba,
a SMB implementation for Linux.

You can find the details on [how to setup Samba][1] on OpenWRT on the wiki.

To Access the Samba drive from a client point to the IP address of your router
for example if you router has an IP address `192.168.1.1` then you can access the
drive at `smb://192.168.1.1`.

#### Wireguard

Wireguard is a new modern and secure VPN in-built into the Linux kernel that is based
on the public key cryptography for peer verification and authentication.
Wireguard is very simple to setup yet very performant making it the ideal VPN
to use on embedded devices like home routers unlike OpenVPN which has a 
huge dependency on OpenSSL taking up a lot more flash space as well as being 
very CPU intensive.

Setting up wireguard on OpenWRT is not very different from setting it up on any
other device besides installing the required packages which you can find [here][2].

For a better more general guide to configuring wireguard see this excellent [blog
post][3] by Stavros Korokithakis.

#### Transmission

libtransmission is a small library to easily download torrents onto any Linux
devices though for it to be useful you often have to pull in at least one
additional library that interacts with libtransmission over RPC.
Only caveat with libtransmission I faced especially on my router was that 
it put significant CPU load while actively downloading a torrent, I'm not sure
if it's because of my CPU not capable enough or I have something misconfigured.
But it's generally not an issue if you don't have something downloading 24/7.

Details on how to install and configure transmission are [here][4].

Once you have everything setup correctly along with wireguard you can effectively
login for anywhere and queue up an torrent to download directly onto your hard drive
connected to you router downloading with the full bandwidth of your home connection.

<figure style="width:85%;margin:auto;">
    {{ resize_image(path="transmission-web.png") }}
<figcaption style="text-align:center;">Fig 2: Transmission Web running on a home router</figcaption>
<br/>
</figure>

#### DDNS

### Resources

[Wireguard]: https://wireguard.com
[OpenWRT]: https://openwrt.org
[1]: https://openwrt.org/docs/guide-user/services/nas/cifs.server
[2]: https://openwrt.org/docs/guide-user/services/vpn/wireguard/start
[3]: https://www.stavros.io/posts/how-to-configure-wireguard/
[4]: https://openwrt.org/docs/guide-user/services/downloading_and_filesharing/transmission
