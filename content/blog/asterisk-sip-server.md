+++
title2="Self-hosted GSM modem server for forwarding calls and texts internationally"
title="Owning your Communications Stack on the Path to a Sovereign Individual"
date=2022-05-09
draft=true
[extra]
tags="asterisk, asterisk server, sip server, Twilio, openwrt, sms, sip, voip, sip phone, sms to email, email"
+++

As a traveller and an expat when staying overseas for long periods of time
it can certainly be helpful to retain the local number of your home country 
to receive calls on your same number or at a minimum still being able to receive 
your bank account and OTP related text messages.

The solution here is not a pitch for [Google Fi][2] or a rant about it being 2022
and we're nowhere close to having a global mobile carrier available to everyone.
However that is far from what this article is about, instead this article 
describes a much more practical, extendable and globally-available solution to the above problem.

<!-- more -->

## Overview 

## Benefits 

This also serves as a mitigation to the ever growing threat of SIM swap attacks
both physical, as the SIM card associated with your number is no longer required to be always in your
phone, and operator social engineering attacks since most VOIP providers have a higher 
security level than most traditional telecom providers.

You might think why go through all this trouble when it's quite cheap to just buy
a new number. Well unlike earlier times when it was relatively easy to switch to a new number
and share it with just your friends, in the post modern era with increasingly
higher number of online services treating your phone number as your digital identity
and 2FA device, its no longer a simple task.

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
