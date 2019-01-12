+++
title="DLNA/UPNP casting support in VLC - Part 1"
date=2018-08-31
[extra]
tags="upnp, dlna, upnp/dlna, dlna/upnp, vlc, casting, stream, media, media renderer, renderer, dlna renderer"
+++

The latest project that I have been working on lately is adding support for 
casting to DLNA/UPNP renderers in VLC.

DLNA renderers, mainly TV’s are DLNA certified devices that are able to play
any media from the local network and is built upon the UPnP A/V Architecture technology.
This allows you to cast any video or audio from any other device to your connected TV.

<!-- more -->

This is similar to chromecast but was designed a lot earlier and unlike chromecast it is
a completely open protocol.

A lot of consumer electronics and television sets in the market are already 
DLNA certified even though they may not be marketed as a Smart TV.
You can search for existing DLNA certified products [here][5].

### The Approach

VLC already has support for [libupnp] and playing content from a DLNA/UPnP Media Server
through it's `services_discovery` interface. Through the addition of the 
`renderer_discovery` interface it was now also possible to add support for 
discovering and casting media to a DLNA/UPnP Media Renderer.

Of course possibility and reality as often quite distant. 
The upnp module in VLC was quite bloated and had already around 2K lines of code in a single file.
It was impossible to add any more features without some cleaning and refactoring
to make sure the code was maintainable and readable
and any addition did not break existing functionality.

The [libupnp] library provides some nice functions and abstractions over the UPnP
spec but is also highly opinionated which is not necessarily a bad thing.

One of the quirks of [libupnp] since it creates its own server and listens on a port
for all UPnP operations as well as has internal thread management is we cannot initialize
the instance multiple times and hence need to keep track of the SDK state.

One solution that VLC already employed was to have a wrapper class around the [libupnp]
SDK and the `UpnpClient_Handle` that also was a singleton. But it was tightly coupled
with the existing module and had module specific members persist along with the singleton.

I therefore proceeded to refactor out the wrapper class as a standalone singleton
and added methods for anyone interested to listen to callbacks received from the [libupnp]
SDK. The patches were merged in the mainline VLC tree and call be found here:

* [upnp: add and use a callback listener interface][1]
* [upnp: move UpnpInstanceWrapper to upnp-wrapper][2]

This made the code drastically simpler and understandable as well as providing unlimited
scalability towards libupnp. I could now easily write more modules that depend on libupnp
and reside as their own modules in independent and isolated files.

[libupnp]: http://pupnp.sourceforge.net/ 
[1]: http://git.videolan.org/?p=vlc.git;a=commit;h=66839225ff9cea419bd9d578278d35eb3c4db800
[2]: http://git.videolan.org/?p=vlc.git;a=commit;h=da4e3c45c04c7cedc692a5fa09af9c511e000365
[3]: https://openconnectivity.org/developer/specifications/upnp-resources/upnp/mediaserver4-and-mediarenderer3
[4]: http://upnp.org/specs/av/UPnP-av-AVArchitecture-v2.pdf
[5]: https://spirespark.com/dlna/products
