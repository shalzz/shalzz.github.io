+++
title2="Self-hosted GSM modem server for forwarding calls and texts internationally"
title="Path to Sovereign Individual: Owning your Communications Stack"
date=2022-06-04
[extra]
tags="Asterisk, Asterisk server, sip server, Twilio, openwrt, sms, sip, voip, sip phone, sms to email, email, gsmbox, gsm, modem, rtp, srtp, sms hijacking, sim swap, sim swap attack, mitigate sim swap, gsm box, huawei dongle, Asterisk-chan-dongle, Asterisk-chan-quectel, BYOC trunk"
+++

<!-- On the path to being a [Sovereign Individual][3], being digitally sovereign -->
<!-- is equally important. Owning and decoupling your digital identity from your mobile -->
<!-- number is a step in that direction. -->

As a [Sovereign Individual][3] when traveling and staying overseas for long periods,
I've found it helpful to still have a working phone number of my home country instead
of switching numbers every time you travel internationally.
Doing so lets you forward and make local home calls or at a minimum still be able to receive 
bank account and OTP related text messages.

The solution I've come up with isn't location dependent like [Google Fi][2] or using 
expensive international roaming plans but instead gives you ownership and
freedom to retain, switch or dispose of your phone numbers across jurisdictions.

<!-- more -->

In the same vein, I recently tweeted about taking control of your messaging
platforms to have custody over your chat messages. Helping you maintain a new
layer of privacy via isolation as well as improving your quality of life by
unifying your chatting platforms. This article though focuses only on voice and text communication.

{{ tweet(url="https://twitter.com/shalzzj/status/1499689706367643649") }}

<!-- toc -->

## Overview 

The solution to the above problem is to move onto a [SIP]/[RTP] stack for all our voice calls by
switching to a VOIP service provider and possibly relying on our PBX server (depending on the jurisdictions regulation). 

When we have our complete VOIP stack in place we'll have a pipeline like this:

```
PSTN Carrier <-> 3G/4G Dongle <-> Asterisk Server <-> Twilio (VOIP Provider) <-> VOIP Softphone
```

## Benefits 

With this approach, we can redirect traditional PSTN calls over VOIP to us anywhere across
the globe. Plus since there's no longer a SIM card required to be tethered to a single device,
we can make and receiver calls across multiple devices.

This also mitigates the ever-growing threat of SIM swap attacks.
Both physical attacks, as the SIM card associated with your number, is no longer required to be always in your
phone, and operator social engineering attacks since most VOIP providers have a higher 
security level than most traditional telecom providers.

## Asterisk PBX Server

While the regulation of some countries like the US allow buying a consumer feature
level number from a VOIP provider like [Twilio] with unrestricted functionality for
calls and SMS/MMS.
The feature set starts to degrade drastically from what we are used to
when buying local numbers for most other countries.

If you are following this guide with a US number or any other country that allows
first-class support to VOIP providers then you can skip this section on setting up an
Asterisk server.

If however you find yourself with a number from a country that restricts VOIP providers
from providing full feature-set local numbers then we can circumvent that via
a 3G/4G USB dongle with voice calling support. This allows us to then bring up
a PBX server connected to the internet with the ability to route traditional PBX calls
via the USB dongle holding your own local number SIM card.

The first step is to then install an [Asterisk][13] server on a machine that can be left running
24/7 for maximum uptime. This guide uses OpenWRT OS and packages in examples
but you can use any machine you have lying around including a Raspberry Pi.

With an already set up OpenWRT router, you'll need the following packages to
install Asterisk:
```sh
opkg install Asterisk Asterisk-pjsip Asterisk-bridge-simple Asterisk-codec-alaw Asterisk-codec-ulaw Asterisk-res-rtp-Asterisk Asterisk-res-srtp Asterisk-chan-dongle
```

### Asterisk-chan-dongle

Now comes the tough part of getting and setting up a USB dongle. We're 
using [wdoekes/Asterisk-chan-dongle][4] an Asterisk channel driver for interfacing
with the USB dongle from the Asterisk server. 

The tricking part here is that the channel driver doesn't work with every 3G/4G USB
dongle, only Huawei 3G dongles and a few 4G dongles. And even with the dongles that it
does work, not every supported dongle has voice support available or firmware unlocked.

The best way is to look at the list of supported dongles on the project's [README][5] file
and try to get your hands on one of them. I bought an E1750 Huawei 3G dongle which 
you still find in stock in many online stores. If it's unavailable in your local markets/
e-commerce sites, try searching it on eBay or AliExpress. I was able to get a
second-hand one from eBay with SIM and voice support unlocked.

Once you have a dongle, insert your SIM card and plug it into the machine you have Asterisk
installed.  Make sure you have the `asterisk-chan-dongle` package installed.

To configure, we edit the `dongle.conf` file found in the configuration folder of Asterisk.
The configuration folder is defined via the `astetcdir` option in the main `asterisk.conf`
file and is usually `/etc/asterisk/`.

If there's no `dongle.conf` file located in the configuration folder, create one.
The default dongle.conf can be found [here][6].

Make a note of the `context` option in `dongle.conf`, the default value of
`context` is `default` which we'll be referencing later.
You can change it to something more appropriate like `dongle-incoming`
to make it easier to remember, which is what we're doing in the below example.

The main configuration requirements are specifying the correct audio and data
device ports of the dongle.

```ini
[dongle0]
context   =   dongle-incoming
audio     =   /dev/ttyUSB1       ; tty port for audio connection;    no default value
data      =   /dev/ttyUSB2       ; tty port for AT commands;         no default value
```

The exact value here will depend on your dongle and the distribution you're running. You might
need to install the `usb_modeswitch` package to switch the dongle from the initial
CD-ROM/mass storage mode to the serial mode. You should then be able to start the Asterisk
server and watch the logs for errors. Make sure there are no dongle-related errors. 

Note: To enable logging you might need to edit `logger.conf` to increase the log
level.
(You can find the `messages` log file in the `astlogdir` folder) : 
```
console => notice,warning,error
messages => notice,warning,error,debug
```

Check the status of your dongle and network registration via these two commands
in the Asterisk console. (You can attach a console to a running Asterisk
server via the command `asterisk -r`)
```
dongle show devices
dongle show device state dongle0
```

If the output shows the device state as "Free", then you're good to go.

### SMS Forwarding (Optional)

The easiest and simplest way to get acquainted with Asterisk and how to configure
it is to set up SMS forwarding from our USB 3G dongle to an email ID.

We can do so by editing `extensions.conf`. The file is already well documented
via comments (any line preceded by `;` is a comment and is ignored).

Append the following lines at the end of the file to set up SMS forwarding.

```ini
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

Here `dongle-incoming` is the name of the context we defined in `dongle.conf`,
and we define extensions within that context for receiving SMS by the way
of `exten => sms`.
You can read more about how [Context, Extensions, and Priorities][7] work in Asterisk from
their wiki.

Once we craft an email message we defer to a sendmail compatible SMTP client to
send the email. This example uses `msmtp` here, you can instead use any
other client and configure it accordingly.

The above extension configuration calls a few Asterisk built-in apps and functions that
you might need to install separately. For OpenWRT install the following packages:
```
opkg install asterisk-app-system asterisk-app-verbose asterisk-func-base64
```

If you just want SMS forwarding to your email and don't care about PSTN voice calls
then congratulations you should have SMS-to-email forwarding working at this point.
If you want voice call functionality as well, continue reading.

### Configuration

Once we have everything installed and running, it's time to configure our
Asterisk PBX server to do two things:

1. Set up call extensions so that Asterisk knows how and where to route incoming and outgoing calls
1. Authenticate and interface with Twilio as a BYOC trunk

While we don't need to use Twilio or a VOIP service provider to act as a
proxy for our PBX server, using one adds a level of security and reliability
to our setup as we don't need to whitelist dynamic IP addresses for communicating with
a mobile softphone and Twilio's global presence and edge locations enable
much better performance and reliability than just directly talking to our home
Asterisk server from across the globe.

The configuration shared here are just minimal examples that should work for most
cases, you may have to tweak and adjust them to suit your setup and requirements.

There are a few files that we'll need to edit mainly, `pjsip.conf`, `pjsip_wizard.conf` and `extensions.conf`

The first thing that we need to do is set up a "transport" for Asterisk to use for SIP
signaling in the `pjsip.conf` file:

```ini
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0:5060
```

The next step is connecting our PBX to Twilio as a Trunk using their BYOC Trunk option.
There is no direct guide provided by Twilio on how to set up a BYOC Trunk, especially
with Asterisk, but the process is similar to setting up a Twilio Elastic SIP Trunk.

Twilio does, fortunately, provide a good enough guide for setting up an Elastic SIP trunk
[here][8], from where we can adapt the configuration shared in the "Asterisk Provisioning" section
to work with the BYOC trunk.

For connecting with the Twilio network as a SIP trunk we need to have a user created
with the corresponding credentials in addition to a SIP domain and a BYOC trunk domain.
We can do so via the Twilio console dashboard, the exact steps for which are described
in the "Twilio" section below.

> Note: The values highlighted in bold in the example configuration will be unique to your
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

Next, we set up the extensions required to receive incoming calls from the PSTN network
and forward them as a SIP call via the Twilio SIP domain to any registered softphones.
And another extension for when we want to make a call from our softphone to then
use the dongle for completing the call to a PSTN number.

`extensions.conf`:
```ini
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

> Note: Enabling Jitterbuffer and Automatic Gain Control (AGC) may require
> additional asterisk packages to be installed.
> On OpenWRT the package names are: `asterisk-func-jitterbuffer` and
> `asterisk-func-speex`.

### NAT settings (optional)

If you're running your Asterisk server on a private network and is behind
Network Address Translation (NAT) then you'll need to enable additional
configuration options for Asterisk's `res_pjsip` module to work reliably.

The most basic configuration required for networking behind a NAT is specifying
the external media address in the transport configuration section of `pjsip.conf`

```diff
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0:5060
+; NAT settings
+local_net                  = 127.0.0.1/24
+local_net                  = 192.168.1.0/24
+external_media_address     = home.example.com ; or a public static IP
+external_signaling_address = home.example.com ; or a public static IP
+allow_reload=no
```

You can find more details on how to configure Asterisk behind a NAT from their
[wiki][10]

## Twilio

The last step in our setup is to configure Twilio SIP domain and BYOC trunk
from the Twilio console. Twilio has a nice tutorial for setting up a [SIP phone][9]
for incoming/outgoing calls directly from Twilio's SIP domain.

We adapt from the tutorial for setting up two SIP domains:

The first SIP domain will be set up similar to the one in the tutorial but
with a different webhook URL for the "A Call Comes In" configuration option.
The Webhook URL can be the URL of a Twilio Function or self-hosted server of the
following Nodejs script:

```javascript
/* URL Parameters
URL parameters: defaultCountry=[international country code - ISO alpha2]
Feel free to remove any console.log statements.
*/
// Require `PhoneNumberFormat`.
const PNF = require('google-libphonenumber').PhoneNumberFormat;
const url = require('url');
// Get an instance of `PhoneNumberUtil`.
const phoneUtil = require('google-libphonenumber').PhoneNumberUtil.getInstance();

exports.handler = function(context, event, callback) {
    const client = context.getTwilioClient();
    let twiml = new Twilio.twiml.VoiceResponse();
    const { From: fromNumber, To: toNumber, SipDomainSid: sipDomainSid } = event;
    let mergedAggregatedE164CredentialUsernames = [];
    let regExNumericSipUri = /^sip:((\+)?[0-9]+)@(.*)/;
    let regAlphaSipUri = /^sip:(([a-zA-Z][\w]+)@(.*))/;
    // Change the defaultCallerId to a phone number in your account
    // and BYOC_SID to your byoc trunks SID.
    const BYOC_SID = 'BY6f6abd4dadcf7d0de2ec4f828a167bac';
    const DEFAULT_CALLER_ID = '+919876543210';  // Also our BYOC number
    let defaultCountry = event.defaultCountry || 'US';
    let fromSipCallerId = (fromNumber.match(regExNumericSipUri)
    ? fromNumber.match(regExNumericSipUri)[1] :
    fromNumber.match(regAlphaSipUri)[2]);
    if (!toNumber.match(regExNumericSipUri)) {
        console.log('Dialing an alphanumeric SIP User');
        twiml.dial({callerId: fromSipCallerId, answerOnBridge: true})
        .sip(toNumber);
        callback(null, twiml);
    }
    let normalizedFrom = (fromNumber.match(regExNumericSipUri)
    ? fromNumber.match(regExNumericSipUri)[1] : DEFAULT_CALLER_ID);
    let normalizedTo = toNumber.match(regExNumericSipUri)[1];
    let sipDomain =  toNumber.match(regExNumericSipUri)[3];
    console.log(`Original From Number: ${fromNumber}`);
    console.log(`Original To Number: ${toNumber}`);
    console.log(`Normalized PSTN From Number: ${normalizedFrom}`);
    console.log(`Normalized To Number: ${normalizedTo}`);
    console.log(`SIP CallerID: ${fromSipCallerId}`);
    // Parse number with US country code and keep raw input.
    const rawFromNumber = phoneUtil.parseAndKeepRawInput(normalizedFrom, defaultCountry);
    const rawtoNumber = phoneUtil.parseAndKeepRawInput(normalizedTo, defaultCountry);
    // Format number in E.164 format
    fromE164Normalized = phoneUtil.format(rawFromNumber, PNF.E164);
    toE164Normalized = phoneUtil.format(rawtoNumber, PNF.E164);
    console.log(`E.164 From Number: ${fromE164Normalized}`);
    console.log(`E.164 To Number: ${toE164Normalized}`);

    function enumerateCredentialLists(sipDomainSid) {
        return client.sip.domains(sipDomainSid)
            .auth
            .registrations
            .credentialListMappings
            .list();
    }
    function getSIPCredentialListUsernames(credList) {
        return client.sip.credentialLists(credList)
            .credentials
            .list();
    }
    enumerateCredentialLists(sipDomainSid).then(credentialLists => {
        Promise.all(credentialLists.map(credList => {
            return getSIPCredentialListUsernames(credList.sid);
        }))
        .then(results => {
            results.forEach(credentials => {
                // Merge together all SIP Domain associated registration
                // credential list usernames prefixed by + into one array
                mergedAggregatedE164CredentialUsernames.push
                .apply(mergedAggregatedE164CredentialUsernames,
                credentials.filter(record => record["username"].startsWith('+'))
                .map(record => record.username));
            });
            console.log(mergedAggregatedE164CredentialUsernames);

            if (mergedAggregatedE164CredentialUsernames.includes(toE164Normalized)) {
                console.log('Dialing another E.164 SIP User');
                twiml.dial({
                    callerId: fromSipCallerId,
                    answerOnBridge: true
                })
                .sip(`sip:${toE164Normalized}@${sipDomain}`);
            } else if (fromE164Normalized === DEFAULT_CALLER_ID) {
               console.log('Dialing a PSTN Number via BYOC');
               twiml.dial({callerId: fromE164Normalized, answerOnBridge: true})
                 .number({byoc: BYOC_SID}, toE164Normalized);
            } else {
               console.log('Dialing a PSTN Number');
               twiml.dial(
                   {callerId: fromE164Normalized, answerOnBridge: true},
                   toE164Normalized
                );
            }
                callback(null, twiml);
            }).catch(err => {
                console.log(err);
                callback(err);
            });
    });
};
```

Change the `DEFAULT_CALLER_ID` variable value to your SIM card number and `BYOC_SID`
value to the SID you get after creating a BYOC trunk SIP domain described below.

The above script calls a PSTN number via our BYOC trunk when calling from our
BYOC number or via a Twilio number when calling from a username formatted as an E.164 Twilio number.
And makes a SIP call when the To number is a username in our credentials list.

The second SIP domain will have the selection "BYOC trunk" as the configuration
under the "Call Control Configuration" section.
The details for setting up a Twilio BYOC trunk
can be found in Twilio's [BYOC trunk][11] documentation.

You can also enable voicemail on incoming calls to your number by adding an `action`
field to the `<Dial>` verb followed by the `<Sip>` noun

```diff
@@ -77,6 +77,7 @@ exports.handler = function(context, event, callback) {
            if (mergedAggregatedE164CredentialUsernames.includes(toE164Normalized)) {
                console.log('Dialing another E.164 SIP User');
                twiml.dial({
+                   action: url.resolve(context.PATH, 'voicemail'),
                    callerId: fromSipCallerId,
                    record: "record-from-answer",
                    answerOnBridge: true

```

You then need to have a `/voicemail` webhook available which then records a message.
Here's an example: [https://gist.github.com/shalzz/3046edd4d2dca123875ac84853f1cbc1](https://gist.github.com/shalzz/3046edd4d2dca123875ac84853f1cbc1)

Twilio provides a generous amount of trial credits, letting you test and correct your
setup before moving onto a paid plan which is when you start paying for usage.
You can use my [Twilio referral link][12] to get $10 in credit when you upgrade.

## Softphone

The last remaining piece of the puzzle is a softphone app through which you'll
receive and make calls. There are a few good softphone apps that work across platforms:
* [Linephone](https://linphone.org/)
* [Acrobits Softphone](https://www.acrobits.net/sip-client-ios-android/)
* [Bria](https://www.counterpath.com/bria-solo/)

## Further Improvements

* [SRTP Media transport](https://www.twilio.com/docs/voice/api/secure-media)
* [Twilio region](https://www.twilio.com/docs/global-infrastructure/understanding-twilio-regions)

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
[10]: https://wiki.asterisk.org/wiki/display/AST/Configuring+res_pjsip+to+work+through+NAT
[11]: https://www.twilio.com/docs/voice/bring-your-own-carrier-byoc
[12]: https://www.twilio.com/referral/KrXNxn
[13]: https://www.asterisk.org/
