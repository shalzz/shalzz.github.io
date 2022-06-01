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
Instead this article describes a much more practical,
extendable and globally-available solution to the above problem.

<!-- more -->

In the same vein, I recently twitted about taking control of your messaging platforms to have custody
over your chat messages while maintaining a level of privacy via an additional layer
of isolation and at the same time improving your quality of life by unifying your
chatting experience. This article though focuses purely on voice and text communication.

{{ tweet(url="https://twitter.com/shalzzj/status/1499689706367643649") }}

<!-- toc -->

## Overview 

The solution we have here is to move on to a [SIP]/[RTP] stack for all our voice calls and
relying on our own PBX server (depending on our use case) in addition to using a VOIP service provider.

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

With an already setup OpenWRT router, you'll need the following packages to
install asterisk:
```sh
opkg install asterisk asterisk-pjsip asterisk-bridge-simple asterisk-codec-alaw asterisk-codec-ulaw asterisk-res-rtp-asterisk asterisk-res-srtp asterisk-chan-dongle
```

### asterisk-chan-dongle

Now comes the tough part of getting and setting up a USB dongle. We're are
using [wdoekes/asterisk-chan-dongle][4] a asterisk channel driver for interfacing
with the USB dongle from an asterisk server. 

The tricking part here is that the channel driver doesn't work with every 3G/4G USB
dongle, only Huawei 3G dongles and few 4G dongles. And even with the dongles that it
does work, not every supported dongle has voice support available or firmware unlocked.

The best way is to look at the list of supported dongles on the projects [README][5] file
and try and get your hands on one of them. I bought a E1750 Huawei 3G dongle which 
you still find in stock in many online stores. If it's unavailable in your local markets/
e-commerce sites, try searching it on ebay or aliexpress. I was able to get a
second hand one from ebay with SIM and voice support unlocked.

Once you have a dongle, insert you SIM card and plug it in the machine you have asterisk
installed.
Make sure you have the `asterisk-chan-dongle` package installed.

To configure we edit the `dongle.conf` file found in the asterisk configuration folder.
The asterisk configuration folder is defined via the `astetcdir` option in the main `asterisk.conf`
and is usually `/etc/asterisk/`.

If there's no `dongle.conf` file located in the configuration folder, create one.
The default dongle.conf can be found [here][6].

Make a note of the `context` option in `dongle.conf`, the default value of
`context` is `default` which we'll be referencing late, you can change it to something
more appropriate like `dongle-incoming` to make it easier to remember, which is
what we're doing below.

The main configuration requirements are specifying the correct audio and data
device ports of the dongle.

```
[dongle0]
context   =   dongle-incoming
audio     =   /dev/ttyUSB1       ; tty port for audio connection;    no default value
data      =   /dev/ttyUSB2       ; tty port for AT commands;         no default value
```

The exact value here will depend on your dongle and the distribution your running. You might
need to install the `usb_modeswitch` package to switch the dongle from the initial
CD-ROM/mass storage mode to the serial mode. You should be able to start the asterisk
server now and watch the logs for errors. Make sure there's no dongle related errors. 

Note: To enable logging you might need to edit `logger.conf` to increase the log
level.
(You can find the `messages` log file in the `astlogdir` folder) : 
```
console => notice,warning,error
messages => notice,warning,error,debug
```

Check the status of your dongle and network registration via these two commands
in the asterisk console. (You can attach a console to an already asterisk
server via the command `asterisk -r`)
```
dongle show devices
dongle show device state dongle0
```

If the output shows the device state as "Free", then you're good to go.

### SMS Forwarding (Optional)

The easiest and simplest way to get acquainted with asterisk and how to configure
it is to setup SMS forwarding from our USB 3g dongle to an email id.

We can do so by editing `extensions.conf`. The file is already well documented,
any line preceded by `;` is a comment and is ignored by the asterisk server.

Append the following lines at the end of the file to setup SMS forwarding.

```
[dongle-incoming]
exten => sms,1,Verbose(Incoming SMS from "${CALLERID(num)} ${BASE64_DECODE(${SMS_BASE64})}")
exten => sms,n,System(echo "To: user@example.com\nSubject: ${CALLERID(num)}\n\n${STRFTIME(${EPOCH},,%Y-%m-%d %H:%M:%S)} - ${DONGLENAME} - ${CALLERID(num)}: " > /tmp/sms.txt)
exten => sms,n,System(echo "${BASE64_DECODE(${SMS_BASE64})}" >> /tmp/sms.txt)
exten => sms,n,System(msmtp -t < /tmp/sms.txt)
exten => sms,n,System(rm -f /tmp/sms.txt)
exten => sms,n,Hangup()

exten => ussd,1,Verbose(Incoming USSD: ${BASE64_DECODE(${USSD_BASE64})})
exten => ussd,n,Hangup()
```

Here `dongle-incoming` is the name of the context we defined in `dongle.conf`
and we define extensions within that context for receiving SMS by the way
of `exten => sms`.
You can read more about how [Context, Extensions and Priorities][7] work in asterisk from
their wiki.

Once we craft an email message we defer to an sendmail compatible SMTP client to
actually send the email. This example uses `msmtp` here, you can instead use any other client and
configure it accordingly.

The above extension configuration calls some asterisk built-in apps and functions that
you might need to install separately. For OpenWRT install the following packages:
```
opkg install asterisk-app-system asterisk-app-verbose asterisk-func-base64
```

If you just want SMS forwarding to your email and don't care about PSTN voice calls
then congratulations you should have SMS-to-email forwarding working at this point.
If you want voice calls functionality as well, continue reading.

### Configuration

Once we have everything installed and running, comes the time of configuring our
asterisk PBX server to do two things:

1. Set up call extensions so that asterisk knows how and where to route incoming and outgoing calls
1. Authenticate and interface with Twilio as a BYOC trunk

While we don't really need to use Twilio or a VOIP service provider to act as a
proxy for our PBX server, using one adds a level of security and reliability
to our setup as we don't need to whitelist dynamic IP addresses for communicating with
a mobile softphone and Twilio's global presence and edge locations enable
much better performance and reliability than just directly talking to our home
asterisk server from across the globe.

The configuration shared here are just minimal examples that should work for most
cases, it's possible you'll have to tweak and adjust to suit your setup and requirements.

There are a few files that we'll need to edit mainly, `pjsip.conf`, `pjsip_wizard.conf` and `extensions.conf`

The first thing that we need to do is setup a transport for asterisk to use for SIP
signalling in the `pjsip.conf` file:

```
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0:5060
```

The next step is connecting our PBX to Twilio as a Trunk using their BYOC trunk option.
There is no direct guide provided by Twilio on how to setup a BYOC trunk especially
with asterisk but the process is similar to setting up a Twilio Elastic SIP trunk.

Twilio does fortunately provide a good enough guide for setting up an Elastic SIP trunk
[here][8], from where we can adapt the configuration shared in the "Asterisk Provisioning" section
to work with the BYOC trunk.

For connecting with the Twilio network as a SIP trunk we need to have a user created
with the corresponding credentials in addition to a SIP domain and a BYOC trunk domain.
We can do so via the Twilio console dashboard, the exact steps for which are described
in the "Twilio" section below.

> The values highlighted in bold in the example configuration will be unique to your
> environment and needs to be replaced with appropriate values to work correctly.

`pjsip_wizard.conf:`
<pre style="background-color:#fdf6e3;color:#657b83;">
<code><span>[user_defaults](!)
</span><span>type = wizard
</span><span>endpoint/context = from-twilio
</span><span>endpoint/allow = !all,ulaw,alaw
</span><span>endpoint/dtmf_mode = rfc4733
</span><span>endpoint/media_encryption = no
</span><span>endpoint/media_encryption_optimistic = no
</span><span>endpoint/trust_id_inbound = yes
</span><span>endpoint/send_pai = yes
</span><span>endpoint/language = en
</span><span>
</span><span>[<b>+919876543210</b>](user_defaults)
</span><span>aor/max_contacts = 5
</span><span>endpoint/callerid = <b>Alan Klein <+919876543210></b>
</span><span>remote_hosts = <b>byoc.twilio-asteriskpbx.sip.singapore.twilio.com, byoc.twilio-asteriskpbx.sip.tokyo.twilio.com</b>
</span><span>
</span><span>[trunk_defaults](!)
</span><span>type = wizard
</span><span>endpoint/transport=transport-udp
</span><span>endpoint/allow = !all,ulaw,alaw
</span><span>endpoint/trust_id_inbound=no
</span><span>endpoint/dtmf_mode=rfc4733
</span><span>endpoint/allow_subscribe = no
</span><span>aor/qualify_frequency = 60
</span><span>
</span><span>[twilio-apac](trunk_defaults)
</span><span>sends_auth = yes
</span><span>sends_registrations = yes
</span><span>remote_hosts = <b>twilio-asteriskpbx.sip.singapore.twilio.com </b>
</span><span>outbound_auth/username = <b style="color: red;">myasteriskpbx </b>
</span><span>outbound_auth/password = <b style="color: red;">myasteriskpbxzx11%VzX </b>
</span><span>endpoint/context = from-twilio
</span><span>aor/qualify_frequency = 60
</span>
</code></pre>

Next we setup an extension to receive incoming calls from the PSTN network
and forward it to our VOIP softphone via a SIP call to the Twilio SIP domain.
And another extension for when we want to make a call from our softphone to then
use the dongle for completing the call to a PSTN number.

`extensions.conf`:
```
[dongle-incoming]
exten => +919876543210,1,Dial(PJSIP/twilio-apac/sip:+919876543210@twilio-asteriskpbx.sip.twilio.com, ,b(dongle-incoming^outbound^1))

exten => outbound,1,Set(JITTERBUFFER(adaptive)=default)
same  => n,Set(AGC(rx)=4000)
same  => n,Return()

[from-twilio]
exten => _X.,1,Set(JITTERBUFFER(adaptive)=2000,1600,120)
same  => n,Set(AGC(rx)=4000)
same  => n,Dial(Dongle/dongle0/+91${EXTEN})
same  => n,Hangup

exten => _+91X.,1,Set(JITTERBUFFER(adaptive)=2000,1600,120)
same  => n,Set(AGC(rx)=4000)
same  => n,Dial(Dongle/dongle0/${EXTEN})
same  => n,Hangup
```

### NAT settings (optional)

## Twilio

Referral link

## Softphone

## Further Improvements

* SRTP Media transport
* Twilio region

[Twilio]: https://www.twilio.com/
[SIP]: https://en.wikipedia.org/wiki/Session_Initiation_Protocol
[RTP]: https://en.wikipedia.org/wiki/Real-time_Transport_Protocol
[1]: https://www.reddit.com/r/selfhosted/comments/q7uint/selfhosted_alternative_to_simbox_gsm_modem_for/
[2]: https://fi.google.com/about/
[3]: https://dougantin.com/the-sovereign-individual-what-you-need-to-know-why/
[4]: https://github.com/wdoekes/asterisk-chan-dongle
[5]: https://github.com/wdoekes/asterisk-chan-dongle#chan_dongle-channel-driver-for-huawei-umts-cards
[6]: https://github.com/wdoekes/asterisk-chan-dongle/blob/master/etc/dongle.conf
[7]: https://wiki.asterisk.org/wiki/display/AST/Contexts%2C+Extensions%2C+and+Priorities
[8]: https://www.twilio.com/docs/sip-trunking/sample-configuration#asterisk
[9]: https://www.twilio.com/blog/registering-sip-phone-twilio-inbound-outbound
