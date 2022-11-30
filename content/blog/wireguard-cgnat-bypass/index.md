+++
title="Circumventing CG-NAT with Wireguard"
date=2022-10-17
[extra]
tags="wireguard, cg-nat, openwrt, vps, lan, proxy, wireguard tunnel, bypass, cg-nat bypass, circumvent, ip forwarding, ipv4, ipv6"
+++

With the increasing exhaustion of IPv4 addresses across the globe, various
ISPs have resorted to implementing IPv4 Carrier Grade Network Address
Translation (CG-NAT) as a solution to this problem.

What this means is ISPs do not assign a publicly accessible IPv4 address to
an end-user's router and/or modem but rather a private IPv4 address that is behind
a carrier network wide <abbr title="Network Address Translation">NAT</abbr> implementation.

<!-- more -->

For those who host servers on their home network or for just having remote
access to the home network for CCTV access, etc. CG-NAT prevents this since
one no longer has a publicly accessible IPv4 address and forwarding incoming
requests through CG-NAT is incredibly hard, even if the ISP makes an effort
of supporting extensions that allow this on their NAT implementation.

The saving grace in this situation is that ISPs have at the same time started
rolling out IPv6 addresses across their network which inherently does have
public addressability down to the individual client devices via [Prefix
Delegation][1] built-in.

However, we're still a far way out from having IPv6 address availability and
accessibility across every network throughout the globe. Especially on mobile
devices with Android's non-functioning/incompatible IPv6 stack, we need a
fallback IPv4 addressing solution that works around CG-NAT.

## Wireguard

With the invention and implementation of [Wireguard][2] within the Linux kernel we
don't need to rely on complicated setups like NAT64, IPv6in4, IPv4in6, etc.

With Wireguard we can have a setup where, we rent a <abbr title="Virtual Private Server">VPS</abbr>
server with a public IPv4 address that we can connect to as
a <abbr title="Virtual Private Network">VPN</abbr> client,
and it routes our private home network requests to our actual home router
across the public internet.

<img src="wireguard-vps.svg">Network Topology</img>

This can be achieved by having our home router connect and establish a
connection with the VPS, since the VPS is the one having a public **and**
static IPv4 address, which works flawlessly with CG-NAT since the router is making an
outgoing connection and not waiting for an incoming connection. This solves a
whole class of issues with CG-NAT, dynamic IP addresses and DDNS.

## VPS Configuration

To install Wireguard related packages and create private keys follow this
[guide][3].

For a more automated VPN setup on a cloud provider, checkout this [ansible
script and guide][5] with a pre-setup cloud shell environment.

The minimal configuration for getting the above solution working is as follows

Here we are assuming your router creates and uses the 192.168.1.0/24 subnet for
your home network. Correspondingly we create and use the 192.168.9.0/24 subnet
space for the Wireguard network.

The first peer we have below is our home router that connects with the
`192.168.9.1` IP address and routes all home network i.e. `192.168.1.0/24` subnet requests
to this peer.

```config
# /etc/wireguard/wg0.conf
[Interface]
Address = 192.168.9.2/32
ListenPort = 51871
PrivateKey = <vps-privkey>

[Peer]
PublicKey = <router-pubkey>
AllowedIPs = 192.168.9.1/32, 192.168.1.0/24

[Peer]
PublicKey = <peer1-pubkey>
AllowedIPs = 192.168.9.3/32

[Peer]
PublicKey = <peer2-pubkey>
AllowedIPs = 192.168.9.4/32
```

You can create and add more peers to the above config in the similar fashion.
You can also optionally create and add the `PreSharedKey` option to each peer
for increased security, particularly quantum resistance.

The peers, other than the home router peer, all directly connect to the VPS and
by the way of IP forwarding can also connect to the home router and all devices on
the home network.

## Home Router Configuration

For the other side of the configuration on the home router, we just need to
set up one peer that connects and persists the connection to the VPS wireguard
server.

This configuration is for the OpenWrt OS, if you're using any other router OS
you'll have to adapt to its configuration format.

```config
# /etc/config/network
config interface 'wg0'
        option proto 'wireguard'
        option private_key '<router-privkey>'
        option listen_port '51820'
        list addresses '192.168.9.1/24'

config wireguard_wg0 'wgclient1'
        option public_key '<vps-pubkey>'
        list allowed_ips '192.168.9.0/24'
        option endpoint_host '<vps-public-ip-address>'
        option endpoint_port '51871'
        option persistent_keepalive '25'
        # set to 0 to disable home clients
        # from accessing the VPS peers directly
        option route_allowed_ips '1'
```

The `allowed_ips` or `AllowedIps` option of the wireguard config also acts as
an ACL or access control method. So if we want to allow other peers, other than
just the direct VPS peer `192.168.9.2`, to access this network we need to allow
their IP addresses as well. We can do that by whilelisting the whole subnet
with `192.168.9.0/24` or allow individual IP's one by one.

That last thing to do is to make sure the Wireguard interface (`wg0` in the example)
is assigned to the `LAN` firewall zone for letting the wg peers access clients
within the LAN zone (192.168.1.0/24).

Run this command to do so on OpenWrt:
```bash
export WG_INTERFACE_NAME=wg0
uci add_list firewall.lan.network="${WG_INTERFACE_NAME}"
```

## IP Forwarding

The most crucial step for this setup is to enable IP forwarding on the VPS
host. This forwards the packets received from the 192.168.1.0/24
clients back to the connected peer with Wireguard handling the routing
based on our `AllowedIPs` config.

```config
# /etc/sysctl.conf
net.ipv4.ip_forward=1
```

Reboot the VPS for this to take effect.

You should now be able to access your home router and clients over the public internet!


[1]: https://en.wikipedia.org/wiki/Prefix_delegation
[2]: https://www.wireguard.com/
[3]: https://wiki.archlinux.org/title/WireGuard
[5]: https://shalzz.gumroad.com/l/self-host-vpn
