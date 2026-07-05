+++
title="NextDNS: A Modern DNS Firewall for Your Whole Network"
date=2026-07-05
[extra]
tags="nextdns, dns, openwrt, android, privacy, security, doh, dot, dns-over-https, dns-over-tls, privacy, ech, sni"
+++

I've been using [NextDNS](https://nextdns.io/?from=83bgqgfw) for over a year now and it's one of those services that once you set up, you wonder how you lived without it. It's a cloud-based DNS resolver with built-in filtering, analytics, and parental controls — think Pi-hole as a hosted service, but with global anycast routing and way more features.

<!-- more -->

## What's Wrong with Plain DNS?

Traditional DNS (port 53, UDP) sends your queries in plaintext. Anyone on your network, your ISP, or anyone snooping on the wire can see every domain you visit. It's also trivially easy for ISPs or governments to hijack DNS responses and redirect you to phishing pages or censorship pages.

DNS over HTTPS (DoH) and DNS over TLS (DoT) fix this by encrypting your DNS queries. Instead of plain UDP packets, your queries travel inside an encrypted TLS tunnel (DoT) or are wrapped in HTTPS requests (DoH). This gives you:

- **Privacy** — your ISP can't see what domains you're resolving
- **Integrity** — nobody can tamper with DNS responses in transit
- **Authentication** — you verify you're talking to the real DNS server

## Why NextDNS?

There are other encrypted DNS providers (Cloudflare 1.1.1.1, Quad9, etc.), but NextDNS goes much further:

**1. Blocking & Filtering**  
You can block malware, tracking domains, adult content, ads, and more — all at the DNS level. It works like Pi-hole but requires no hardware, no configuration on each device, and it applies to every device on your network. It uses multiple blocklists (OISD, Energized, etc.) out of the box.

**2. Analytics**  
The dashboard shows you exactly which domains are being queried, how many were blocked, which devices made the requests, and a timeline of activity. It's incredibly useful for spotting rogue IoT phones home or checking if a block is too aggressive.

**3. Allow/Deny Lists**  
You can whitelist domains that get falsely blocked and blacklist entire domains or TLDs. Changes propagate in seconds.

**4. Parental Controls**  
Built-in categories for restricting adult content, social media, gaming, etc. You can also schedule when certain categories are blocked (e.g., block social media during school hours).

**5. Response Policy Zones (RPZ)**  
RPZ is a powerful DNS-level policy mechanism that lets you redirect, block, or log queries based on domain patterns. NextDNS supports it natively, giving you advanced control far beyond simple blocklists — you can, for example, redirect queries for known malware domains to a local sinkhole, override DNS responses for specific domains across your entire network, or quarantine IoT device domains.

**6. Native DoH/DoT/DoQ/DNSCrypt**  
No need to run a local proxy — every device can talk to NextDNS directly over encrypted DNS.

**7. No Hardware Required**  
It Just Works on your existing router, phone, or laptop. You don't need a Raspberry Pi or a separate server.

**8. Global Anycast**  
NextDNS has servers all over the world. Latency is consistently lower than running a Pi-hole over a VPN or a self-hosted Unbound instance.

**9. Censorship Resistance** 
Compatible browsers enable encrypted Server Name
Indication (SNI) on top of DoH to prevent leaking of DNS hostnames and thus
thwarting commonly used internet censorship attempts by ISP's and governments.


They have a generous free tier (300,000 queries/month) and the paid plan is cheap — $1.99/month or $19.90/year for unlimited queries and up to 50 devices per configuration.

## Setting Up NextDNS on OpenWrt with Stubby

This is my recommended setup because it protects every device on your network — including smart TVs, IoT devices, game consoles, and guest devices that you can't configure individually.

OpenWrt has native support for `stubby`, a DNS-over-TLS stub resolver. The setup is straightforward.

### Step 1: Get Your Configuration ID

Sign up at [NextDNS](https://nextdns.io/?from=83bgqgfw), create a configuration, and note your **Configuration ID** — it looks like `abc123`. You'll find it under the Setup tab.

### Step 2: Install Stubby on OpenWrt

SSH into your router and run:

```sh
opkg update
opkg install stubby luci-app-stubby
```

### Step 3: Configure Stubby

Edit `/etc/stubby/stubby.yml`:

```yaml
resolution_type: GETDNS_RESOLUTION_STUB

dns_transport_list:
  - GETDNS_TRANSPORT_TLS

tls_authentication: GETDNS_AUTHENTICATION_REQUIRED
tls_query_padding_blocksize: 128
edns_client_subnet_private: 1

idle_timeout: 10000

listen_addresses:
  - 127.0.0.1@5353
  - 0::1@5353

round_robin_upstreams: 1

upstream_recursive_servers:
  - address_data: 45.90.28.0
    tls_auth_name: "abc123.dns.nextdns.io"
  - address_data: 2a07:a8c0::0
    tls_auth_name: "abc123.dns.nextdns.io"
  - address_data: 45.90.30.0
    tls_auth_name: "abc123.dns.nextdns.io"
  - address_data: 2a07:a8c1::0
    tls_auth_name: "abc123.dns.nextdns.io"
```

Replace `abc123` with your actual Configuration ID.

The IP addresses are NextDNS's anycast servers. Using addresses instead of hostnames avoids a chicken-and-egg problem where DNS needs DNS to resolve the DNS server. The `tls_auth_name` is what Stubby will validate against the TLS certificate, and it includes your config ID so NextDNS knows which filtering policy to apply.

### Step 4: Configure dnsmasq (or unbound)

In LuCI, go to **Network → DNS → Forwards**, set "DNS forwardings" to `127.0.0.1#5353`.

Or from the CLI, edit `/etc/config/dhcp`:

```
list server '127.0.0.1#5353'
```

Then restart the services:

```sh
/etc/init.d/stubby restart
/etc/init.d/dnsmasq restart
```

### Step 5: Verify It Works

Visit [NextDNS's test page](https://nextdns.io/?from=83bgqgfw) or check the analytics dashboard. You should see queries coming in from your home IP.

---

## Setting Up NextDNS on Android

Android has supported DNS-over-TLS (DoT) since Android 9 (Pie) and DNS-over-HTTPS (DoH) since Android 11. You don't need any third-party app for basic encrypted DNS.

### Using Private DNS (DoT, Android 9+)

1. Go to **Settings → Network & internet → Private DNS**
2. Select **Private DNS provider hostname**
3. Enter: `abc123.dns.nextdns.io` (replace `abc123` with your config ID)
4. Tap **Save**

That's it. All DNS queries from your phone will now be encrypted with TLS and filtered by your NextDNS policy. This works on both Wi-Fi and mobile data.

### Using the NextDNS App (DoH, More Features)

If you want DoH instead of DoT, or you want per-app filtering, logging, or the ability to pause protection, install the NextDNS app from F-Droid or the Play Store:

1. Install [NextDNS](https://nextdns.io/?from=83bgqgfw)
2. Open the app and sign in
3. It will automatically detect your configuration
4. Enable the VPN-based DNS — this creates a local VPN that intercepts DNS queries and sends them over HTTPS

The app approach gives you more visibility (you can see real-time queries on your phone) and works on Android versions older than 9.

### Verifying Android Setup

Visit `https://test.nextdns.io` in your browser. It will show whether you're connected to NextDNS, your config ID, protocol (DoT or DoH), and latency.

---

## DNS-over-HTTPS vs DNS-over-TLS: Which to Use?

Both achieve the same goal — encrypted DNS. The differences are subtle:

| Aspect | DoT | DoH |
|--------|-----|-----|
| Port | 853 (dedicated) | 443 (shared with HTTPS) |
| Protocol | TLS over TCP | HTTPS |
| Ease to block | Easy (block port 853) | Hard (looks like normal web traffic) |
| Performance | Slightly lower overhead | Slightly more overhead |
| Android support | Native (Private DNS) | Via app only |

On OpenWrt, I recommend **Stubby (DoT)** because it's lightweight and OpenWrt has excellent support for it. On Android, the native **Private DNS (DoT)** is the simplest. If you're on a restrictive network that blocks port 853, the NextDNS app with **DoH** will work because it tunnels through port 443.

---

## Billing

If you're based in India and trying to signup for their paid plans, note that
their stripe integration for payments does not have OTP authentications (Verified-By-Visa
enabled, thus making all Visa cards fail being charged.

Any MasterCard with international payments enabled should work fine however!

## Final Thoughts

NextDNS is one of those rare services that's simultaneously simple enough for non-technical users and powerful enough for network nerds. The OpenWrt setup takes 10 minutes and gives your entire home network ad-blocking, malware protection, and parental controls without any ongoing maintenance.

The free tier covers a typical household, and even the paid plan is cheaper than a coffee. If you've been considering setting up Pi-hole but haven't gotten around to it, give [NextDNS](https://nextdns.io/?from=83bgqgfw) a try instead.
