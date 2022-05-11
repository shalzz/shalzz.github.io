+++
title2="Self-hosted GSM modem server for forwarding calls and texts internationally"
title="Path to Sovereign Individual: Owning your Communications Stack"
date=2022-05-09
draft=true
[extra]
tags="asterisk, asterisk server, sip server, Twilio, openwrt, sms, sip, voip, sip phone, sms to email, email, gsmbox, gsm, modem"
+++

As a traveller and an expat when staying overseas for long periods of time
it can certainly be helpful to retain the local number of your home country 
to receive calls on your same number or at a minimum still being able to receive 
your bank account and OTP related text messages.

The solution here is not a pitch for [Google Fi][2] or a rant about it being 2022
and we're nowhere close to having a global mobile carrier available to everyone.
However that is far from what this article is about, instead this article 
describes a much more practical, extendable and globally-available solution to the above problem.

On the path to being a [Sovereign Individual][3], I believe being digitally sovereign
is equally important. Owning and decoupling your digital identify from your mobile
number is definitely a step in that direction.

<!-- more -->

In the same vein, I recently twitted about taking control of your messaging platforms to have custody
over your chat messages while maintaining a level of privacy via an additional layer
of isolation and at the same time improving your quality of life by unifying your
chatting experience.

{{ tweet(url="https://twitter.com/shalzzj/status/1499689706367643649") }}

## Overview 

The solution we have here is to move on to a SIP/RTP stack for all our voice calls and
relying on our own PBX server and/or use a VOIP service provider.

When we have our complete VOIP stack in place we'll have a pipeline like this:

```
PSTN Carrier <-> 3G/4G Dongle <-> Asterisk Server <-> Twilio <-> VOIP Softphone
```

## Benefits 

Using this approach we redirect traditional PSTN calls over VOIP to us anywhere across
the globe. Plus since there's no longer a SIM card required to be tethered to a single device,
we call make and receiver calls across multiple devices.

This also serves as a mitigation to the ever growing threat of SIM swap attacks
both physical, as the SIM card associated with your number is no longer required to be always in your
phone, and operator social engineering attacks since most VOIP providers have a higher 
security level than most traditional telecom providers.

While it's quite cheap to just buy a new number, unlike earlier times, in the
post modern era with increasingly higher number of online services treating your
phone number as your digital identity and 2FA device, its no longer a simple task.

## Asterisk with chan-dongle

While it's relatively easy in some countries including the US to just buy a consumer feature
level number from a VOIP provider like [Twilio] that lets you receive and make calls
as well as SMS. The feature set starts to degrade drastically from what we are used to,
in most other countries.

### NAT settings (optional)

## Twilio

## Softphone

[Twilio]: https://www.twilio.com/
[1]: https://www.reddit.com/r/selfhosted/comments/q7uint/selfhosted_alternative_to_simbox_gsm_modem_for/
[2]: https://fi.google.com/about/
[3]: https://dougantin.com/the-sovereign-individual-what-you-need-to-know-why/
