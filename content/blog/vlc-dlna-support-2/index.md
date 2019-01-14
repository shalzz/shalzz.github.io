+++
title="DLNA/UPNP casting support in VLC - Part 2"
date=2019-01-12
[extra]
tags="upnp, dlna, upnp/dlna, dlna/upnp, vlc, casting, stream, media, media renderer, renderer, dlna renderer"
+++

DLNA renderers, mainly TV’s are DLNA certified devices that are able to play
any media from the local network and is built upon the UPnP A/V Architecture technology.
This allows you to cast any video or audio from any other device to your connected TV.

<!-- more -->

## Content
1. [Discoverying Devices](#discoverying-devices)
1. [Casting to the Device](#casting-to-the-device)
1. [Device Media Formats](#device-media-formats)
1. [Future](#future)
    1. [Demux Filter](#demux-filter)
    1. [DLNA v1.5](#dlna-v1-5)
1. [Code](#code)
1. [Release](#release)

## Discoverying Devices

The next step in adding DLNA support was adding a `renderer_discovery`
module responsible for searching and adding devices that are Media Renderers i.e.
capable of playing a video/audio stream that we may cast to it.

To do that we use the `libupnp` library to find the devices and call the
VLC `vlc_renderer_item_new()` function for each device with relevant information
so that they can be show to the user in a menu and then select which device to cast to.

Someone had partially done this before me but not yet merged, partly due to the problem
I explained and solved in the previous post of this series. After adapting that
patch we were now able to discover and list any DLNA renderers available
on the same network.

{{ resize_image(path="vlc-dlna-discover-clip.jpg") }}

## Casting to the Device

Now to actually be able to cast to a device we need to create a `stream_out`
VLC module with http as the transport method. Much of that logic is similar to
how chromecast creates a `stream_out` except that we use UPnP Actions represented
as SOAP requests instead of the horror that is Google's protobuf to talk to the device.

After going through the DLNA guidelines and the UPnP AV spec, I added functions
that called the actions required to talk to any DLNA device implementing the AVTransport Service
and we had a working implementation of VLC acting as a UPnP control point that
was able to cast to almost all DLNA renderers.

I say almost all because legacy DLNA renderers (below DLNA v1.5) and renderers
not implementing the `AVTransport::SetAVTransportURI` action are not yet supported.

## Device Media Formats

Having a working tranport is still half of story, another important aspect of casting is to make
sure the device is able to decode the file that you are trying to play.
To solve this I had to call the `ConnectionManager::GetProtocolInfo()` action
to get a CSV list of all the Media formats supported by the device, maintain
a manual list of all possible Media format profiles defined by DLNA.org and others within VLC,
find a profile that best describes the file the user is trying to play and then
compare that profile to the device profiles to make sure if it can play it.

And if a device does not support the profile of the file we are trying to play
then we transcode the file to a format that the device supports, on the fly,
and stream it to the device.

As you can imagine this is quite an involved process more so with the transcoding
since we had to make sure the transcoding will work in a reasonable amount of time
on every platform VLC is run i.e. on Linux, Windows, MacOS, iOS and android
each of which has different platform APIs and SoCs/Processors
and hence took up majority of my time.

## Future 

There are still a few things left to add for a seamless experience with DLNA casting
in VLC.

### Demux Filter
Due to the nature of VLC's module system and its media processing pipeline, `stream_out`
modules cannot control the rate at which a video is decoded and sent out for display.
So things like pause, fast-forward and track sync between VLC and the DLNA device doesn't
work properly yet.

There's a workaround to use `demux_filter` to control the pacing between the `demux`
and the `access_out` module using an IPC mechanisuim between the two modules,
even though they both run in the same process.

### DLNA v1.5
DLNA v1.5 is a legacy DLNA spec version that has very strict requirements and cannot
easily play any supported media from the cloud. These requirements were removed
in later versions of the DLNA spec.

To support DLNA v1.5 Media Renderers (many old Samsung and Sony TVs) 
we need a DLNA v1.5 compliant HTTP server with the appropriate DLNA specific headers.
This is related to the `demux_filter` and can be easily implemented once we have that.

## Code

The merged commits can be found here:
1. [upnp: add renderer discoverer][1]
1. [chromecast: refactor out encoder option functions][2]
1. [dlna: add a DLNA stream out][3]
1. [dlna: add GetProtocolInfo action][4]
1. [dlna: add PrepareForConnection action][5]

## Release

You can start casting to DLNA compliant devices using the next VLC 4.0 release
or the latest nightly[^fn-warn]
releases for Windows, MacOS and Linux.

[^fn-warn]: Warning: with any nightly release there may still be bugs that are not yet fixed.

[1]: http://git.videolan.org/?p=vlc.git;a=commit;h=0d89fe3fd7d27d7c3f349bb46a915dbae65c02f8
[2]: http://git.videolan.org/?p=vlc.git;a=commit;h=ec61edc0d0292ab37bb1dbafb23a8aed49e966bb
[3]: http://git.videolan.org/?p=vlc.git;a=commit;h=7da4464ca093604f2a507e3b39330fed17838e62
[4]: http://git.videolan.org/?p=vlc.git;a=commit;h=0a34ce334a7c8b2d3926148be30f5b69fd253e41
[5]: http://git.videolan.org/?p=vlc.git;a=commit;h=b671d3b3270790fd11aa3bc76cfe42f75ca25c52
