+++
title2="Self-hosted GSM modem server for forwarding calls and texts internationally"
title="Path to Sovereign Individual: Owning your Communications Stack"
date=2022-05-09
draft=true
[extra]
tags="asterisk, asterisk server, sip server, Twilio, openwrt, sms, sip, voip, sip phone, sms to email, email, gsmbox, gsm, modem"
+++

On the path to being a [Sovereign Individual][3], I believe being digitally sovereign
is equally important. Owning and decoupling your digital identify from your mobile
number is a step in that direction.

As a sovereign individual, traveling and staying overseas for long periods of time
it can certainly be helpful to retain the local number of your home country 
to receive calls on your same number or at a minimum still being able to receive 
your bank account and OTP related text messages.

The solution here is not a pitch for [Google Fi][2] or a rant about it being 2022
and we're nowhere close to having a global mobile carrier available to everyone.
However that is far from what this article is about, instead this article 
describes a much more practical, extendable and globally-available solution to the above problem.

<!-- more -->

In the same vein, I recently twitted about taking control of your messaging platforms to have custody
over your chat messages while maintaining a level of privacy via an additional layer
of isolation and at the same time improving your quality of life by unifying your
chatting experience. This article though focuses purely on voice and text communication.

{{ tweet(url="https://twitter.com/shalzzj/status/1499689706367643649") }}

## Overview 

The solution we have here is to move on to a [SIP]/[RTP] stack for all our voice calls and
relying on our own PBX server depending on our use case in addition to using a VOIP service provider.

When we have our complete VOIP stack in place we'll have a pipeline like this:

```
PSTN Carrier <-> 3G/4G Dongle <-> Asterisk Server <-> Twilio (VOIP Provider) <-> VOIP Softphone
```

## Benefits 

With this approach we redirect traditional PSTN calls over VOIP to us anywhere across
the globe. Plus since there's no longer a SIM card required to be tethered to a single device,
we call make and receiver calls across multiple devices.

This also serves as a mitigation to the ever growing threat of SIM swap attacks
both physical, as the SIM card associated with your number is no longer required to be always in your
phone, and operator social engineering attacks since most VOIP providers have a higher 
security level than most traditional telecom providers.

While it's quite cheap to just buy a new number when you move to a new location,
unlike earlier times, in the post modern era with increasingly
higher number of online services treating your phone number as your
digital identity and 2FA device, its no longer a simple task.

## Asterisk PBX Server

While it's relatively easy in some countries including the US to just buy a consumer feature
level number from a VOIP provider like [Twilio] that lets you receive and make calls
as well as SMS. The feature set starts to degrade drastically from what we are used to,
in most other countries.

If you are following this guide with a US number or any other country that allows
first class support to VOIP providers then you can skip this section on setting up an
asterisk server.

If however you find yourself with a number from a country which restricts VOIP providers
from providing full feature-set local numbers then you can circumvent that via
a 3G/4G USB dongle with voice calling support. This allows you to then bring up
a PBX server connected to the internet with the ability to route traditional PBX calls
via the USB dongle holding your own local number SIM card.

The first step is to install asterisk on a machine. We need the machine to be running
24/7 with the maximum uptime. I've setup on my OpenWRT router but you do so on any
machine you have lying around including a Raspberry Pi.

With an already setup OpenWRT router, you need to install the following packages to
install asterisk:
```sh
opkg install asterisk asterisk-pjsip asterisk-bridge-simple asterisk-codec-alaw asterisk-codec-ulaw asterisk-res-rtp-asterisk asterisk-res-srtp asterisk-chan-dongle
```

### asterisk-chan-dongle

Now comes the tough part of getting and setting up a USB dongle.
Make sure you have the `asterisk-chan-dongle` package installed.

### Configuration

Once we have everything installed and running, comes the time of configuring the 
asterisk PBX server to do two things:

1. Set up extensions so asterisk knows how and where to route incoming and outgoing calls
1. Authenticate and interface with Twilio as a BYOC trunk

The configuration I share here are just minimal examples that should work for most
cases, it possible you'll have to tweak and adjust to suit your setup and requirements.

#### SMS Forwarding

```
opkg install asterisk-app-system asterisk-app-verbose asterisk-func-base64
```

### NAT settings (optional)

## Twilio

## Softphone

[Twilio]: https://www.twilio.com/
[SIP]: https://en.wikipedia.org/wiki/Session_Initiation_Protocol
[RTP]: https://en.wikipedia.org/wiki/Real-time_Transport_Protocol
[1]: https://www.reddit.com/r/selfhosted/comments/q7uint/selfhosted_alternative_to_simbox_gsm_modem_for/
[2]: https://fi.google.com/about/
[3]: https://dougantin.com/the-sovereign-individual-what-you-need-to-know-why/

