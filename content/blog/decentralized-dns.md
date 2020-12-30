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
require trusting a third party as a design requirement. That is until now...

## Handshake

With the advent of blockchain and it's echoing principles of the early internet
and cyberpunk, it provides a novel socio-economic-cryptographic solution for avoiding
the need to require placing trust in middlemen in inherently peer-to-peer systems.



[^fn-1]: [A Deep Dive on the Recent Widespread DNS Hijacking Attacks][1]:
    "...hijacking the DNS servers for these targets, so that all email and virtual private networking (VPN) traffic was redirected to an Internet address controlled by the attackers..."

[^fn-2]: [2016 Dyn cyberattack][2]: The attack caused major Internet platforms and services to be unavailable to large swathes of users in Europe and North America.

[1]: https://krebsonsecurity.com/2019/02/a-deep-dive-on-the-recent-widespread-dns-hijacking-attacks/
[2]: https://en.wikipedia.org/wiki/2016_Dyn_cyberattack
