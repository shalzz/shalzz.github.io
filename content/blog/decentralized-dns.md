+++
title="Owning your DNS Stack: Towards a more Decentralized Internet"
date=2020-12-30
draft=true
[extra]
tags="dns, handshake, adguard, pihole, pi-hole, server, dns server"
+++

## Introduction

Domain Name System (DNS) is a protocol for naming servers on the internet that hosts
a website. Servers on the internet are recognizable only by their IP addresses
which are unique numerical labels. For convenience every website uses a
fairly recognizable domain name (www.ietf.org) to
point users to their own servers (4.31.198.44) hosting the website by the means
of the DNS system, where a DNS server provides the mapping between the domain
name and the server IP address.

However the current architecture of DNS servers and domain name issuers is highly 
centralized and controlled by a few organizations and corporations of the world.

<!-- more -->

## The Dark Underbelly

Having such a critical part of the Internet infrastructure be a central point
of failure can have devastating effects and is against the founding principles
of the internet envisioned by its creators.

There have been various instances where websites and user information has been 
compromised by hijacked DNS systems and domain names [^fn-1] as well as large scale
internet outages due to design flaws being exploited in the DNS protocol [^fn-2].

Much of the hierarchy and centralization of the DNS system stems from the need to
have a trusted party between domain/website owners and users, mainly TLD root domain
owners (e.g. ICANN for .com) who verify the ownership of domain names and make sure
only authorized personal can update DNS records of a website.

For a long time there hasn't been a technically better system available that wouldn't
require trusting a third party as a design requirement. That is until now…

## Handshake

With the advent of blockchain and it's echoing principles of the early internet
and cyberpunk, it provides a novel socio-economic-cryptographic solution for avoiding
the need to require placing trust in middlemen in inherently peer-to-peer systems.

The [Handshake network][3] is a decentralized, permissionless naming protocol
backed by a blockchain designed to solve the trusted 3rd party
problem with root DNS naming zones (TLD's) and Certificate Authorities.

If you are a domain owner wanting to use the Handshake network, owning a
Handshake name involves quite a few steps
for which there are already a lot of good resources available on the internet.
So, in this post we are going to focus on how you can use Handshake as an end user
and start relying on handshake for resolving traditional and handshake names.

## Setup

The Handshake networks is made up of nodes acting as peers and exchanging messages
with every other node in the network. To participate in the network you need to
run the node software yourself on a machine or server.

The Handshake org provides open source software that is
built according to the Handshake protocol spec and has mainly two kinds of nodes:

* Full node: [hsd][4]
* Light node: [hnsd][5]

We are gonna setup the full node `hsd` daemon as our DNS resolver. You can override
the DNS server settings for either your individual device on you home network
(desktop, Smart TV) but it's recommended to override the DNS server settings in
your Wi-Fi router if you have access to it.

Since it's a full node we are gonna need to maintain the full chain state which
can easily take multiple gigabytes of storage in addition to a 24x7 running system
on you home network. The requirement hardware for this is:

* A Raspberry Pi or any other single board computer with atleast 2 GB of Ram
* At least a 500GB portable HDD/SDD

Once you have the required hardware we can start setting up the software. I'd
recommended use docker images to avoid dependency hell.

hsd:

```yaml
version: "3"

services:
  hsd:
    container_name: hsd
    image: hsd:2.2.0-a1409dc4
    volumes:
      - '/media/sda1/hsd-data:/root/.hsd'
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "5300:5300/udp"
      - "5300:5300/tcp"
    command:
      - 'hsd'
      - '--ns-host'
      - '0.0.0.0'
      - '--ns-port'
      - '5300'
      - '--rs-host'
      - '0.0.0.0'
      - '--rs-port'
      - '53'
    restart: unless-stopped

```


```yaml
services:
  adguard:
    container_name: adguard
    image: adguard/adguardhome:latest
    ports:
      - "53:53/tcp"     ## dns
      - "53:53/udp"     ## dns
      - "67:67/udp"     ## dhcp
      - "68:68/tcp"     ## dhcp
      - "68:68/udp"     ## dhcp
      - "80:80/tcp"     ## dashboard
      - "443:443/tcp"   ## dashboard
      - "853:853/tcp"
      - "3000:3000/tcp"
    volumes:
      - './workdir:/opt/adguardhome/work'
      - './config:/opt/adguardhome/conf'
    restart: unless-stopped
```

[^fn-1]: [A Deep Dive on the Recent Widespread DNS Hijacking Attacks][1]:
    “…hijacking the DNS servers for these targets, so that all email and virtual private networking (VPN) traffic was redirected to an Internet address controlled by the attackers…”

[^fn-2]: [2016 Dyn cyberattack][2]: The attack caused major Internet platforms and services to be unavailable to large swathes of users in Europe and North America.

[1]: https://krebsonsecurity.com/2019/02/a-deep-dive-on-the-recent-widespread-dns-hijacking-attacks/
[2]: https://en.wikipedia.org/wiki/2016_Dyn_cyberattack
[3]: https://handshake.org/
[4]: https://github.com/handshake-org/hsd
[5]: https://github.com/handshake-org/hnsd
[6]: https://gist.github.com/shalzz/e30d41403f92feef0d2688d081336dbd
