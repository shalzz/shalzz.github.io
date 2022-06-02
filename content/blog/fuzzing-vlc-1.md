+++
layout="post"
title="Fuzzing libVLC - The Architecture"
date=2017-06-10
updated=2017-06-24T00:00:00+05:30
[extra]
tags="libfuzzer, libvlc, vlc, fuzzing, gsoc-2017, gsoc"
+++

>_Introduction to this blog post [here](@/blog/why-fuzz-vlc.md)_

### The Architecture

I met with the VLC developers and my mentor at the VideoLabs office 
in Paris and after a few meetings and discussions we had a pretty good
idea on how we could fuzz test libVLC and the VLC core most appropriately.

In VLC, except the core, everything is a module.
There are over 200+ modules in VLC along with libVLCCore and libVLC.

The main module categories that take an input are:
* Access
* Access-demuxer
* Demuxer
* Packetizer
* Decoder
* Video filter

<!-- more -->

The VLC core tries to load the modules of these categories in the following order:

```
Access => Demux => [Packetizer] => Decoder => [Filter] => Out
```

We decided we would fuzz test at least the demux 
<abbr title="Application Programming Interface">API</abbr>, the decoder API,
the packetizer API and possibly the video filter API's which loads modules with these
capabilities.

Along with writing the fuzz targets for these API's, I also need to provide a 
corpus of sample input so that libfuzzer can provide a structured input according
to the video format/codec and not trip up the parse just because the basic headers and
magic numbers were not in place.

We have an end goal of eventually setting up an continuous fuzzing server either on ClusterFuzz
by OSS-Fuzz or a server hosted by VideoLan.
Using OSS-Fuzz has a lot of advantages such as their 1000+ CPU infrastructure,
streamlined process, automated issue tracker, well tested and good documentation.
But we have yet to decided which direction we'll go from here.
