+++
title="Circumventing CG-NAT with Wireguard"
date=2022-10-17
draft=true
[extra]
tags=""
+++

With the increasing exhaustion of IPv4 addresses across the globe, various
ISP's have resorted to implementing IPv4 Carrier Grade Network Address
Translation (CG-NAT) as a solution to this problem.

What this means is ISP's do not assign a publicly accessible IPv4 address to
end-users routers and modems but rather a private IPv4 address that is behind
a carrier network wide <abbr title="Network Address Translation">NAT</abbr> implementation.

<!-- more -->

For those who host servers on their home network or for just having remote
access to the home network for CCTV access, etc. CG-NAT prevents that since
one no longer has a publicly accessible IPv4 address and forwarding incoming
requests through CG-NAT is incredibly hard, even if the ISP makes an effort
of supporting these extensions on their NAT implementation.

The saving grace in this situation is that, ISP have at the same time started
rolling out IPv6 addresses across their network which inherently does have
public addressability down to the individual client devices via [Prefix
Delegation][1] built-in.

However we're still a far way out from having IPv6 address availability and
accessibility across every network through out the globe. Especially on mobile
devices with Android's non-functioning/incompatible IPv6 stack we still need a
fallback IPv4 addressing solution that works-around CG-NAT

## Wireguard

With the invention and implementation of [Wireguard][2] within the Linux kernel we
don't need to rely on complicated setups like NAT64, IPv6in4, IPv4in6, etc.

With Wireguard we can have a setup, where we rent a <abbr title="Virtual Private Server">VPS</abbr> server with a public
IPv4 address that we can connect to as a <abbr title="Virtual Private Network">VPN</abbr> client,
and it routes our private home network requests to our actual home router across the public
internet.

<img src="wireguard-vps.svg">Network Topology</img>


[1]: https://en.wikipedia.org/wiki/Prefix_delegation
[2]: https://www.wireguard.com/
