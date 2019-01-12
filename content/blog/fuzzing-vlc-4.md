+++
title="Fuzzing libVLC - Final Report"
date=2017-08-28
[extra]
tags="libfuzzer, libvlc, vlc, fuzzing, gsoc-2017, gsoc"
last_modified_at=2018-01-10T00:00:00+05:30
+++

> Update: My fuzz targets have been merged into the VLC mainline tree by commits [74e7bd2][14], [b83e9f2][15].

>_Introduction to my GSoC project [here](/blog/why-fuzz-vlc)_

Majority of the parsing code of VLC has been successfully setup to be fuzzed by:

* The demux fuzz target which creates an input stream from the fuzzed input provided by libfuzzer,
probes and loads an appropriate demux module and demultiplexes the input into the various output
elementary streams, minimally handling all the ES callbacks and calling various demux_Control's to
increase code coverage.

<!-- more -->

* The decoder fuzz target by creating a packetizer and a decoder instance while ignoring the
stream out, packetizing the input block and then proceeding to decode the various packetized blocks.
Hardware decoding as been disabled, so that we only test the codecs included in VLC and the module itself
though many of the codecs are provided by 3rd party libraries.

These two fuzz targets cover over 35 demux modules and 64 different codec modules or 163k lines of code
which is 20.7% of the VLC code base.

Actual code coverage as measured by libFuzzer would be different if not more but since as of now
VLC is unable to build with the clang linker, I do not have that coverage information.

#### Some additional challenges and features were

* adding first class support for sanitizers in the VLC build system.
You can now compile VLC with sanitizers just by using the --with-sanitizer switch.
These changes are merged into the mainline, more details [here][1]
* finding and adding sample inputs of various video file formats and containers
and elementary streams encoded with different codecs for the fuzz targets
* Creating a dictionary of various tokens and FourCC's used to separate the various
chunks and atoms in container formats.

#### Code

Since VideoLAN has still not decided whether to use Google's OSS-fuzz or agree to their terms
and doesn't want this code to be misused by hackers, they have decided not to keep this in
the mainline tree to minimize exposure.

And so here are all of my out of tree patches:

* [57949e7a9b9da4d9130db0359199b16fca1b061c](http://git.videolan.org/?p=vlc.git;a=commit;h=57949e7a9b9da4d9130db0359199b16fca1b061c) (merged)
* [2bc9957c615d9505f89d88511a0d6ef3889b392b](http://git.videolan.org/?p=vlc.git;a=commit;h=2bc9957c615d9505f89d88511a0d6ef3889b392b) (merged)
* [583b4b6ed83e64cf52ab75d0d5163a360d626ec6](https://code.videolan.org/GSoC2017/shalzz/vlc/commit/583b4b6ed83e64cf52ab75d0d5163a360d626ec6)
* [89cc31145c47cb3769d9c3258a1d2baa4d563c37](https://code.videolan.org/GSoC2017/shalzz/vlc/commit/89cc31145c47cb3769d9c3258a1d2baa4d563c37)
* [c893f6597efc23cf234af12f09d05510c0c744c3](https://code.videolan.org/GSoC2017/shalzz/vlc/commit/c893f6597efc23cf234af12f09d05510c0c744c3)
* [ef80ea254005e84589dd310abe87ce0d3d2a0c90](https://code.videolan.org/GSoC2017/shalzz/vlc/commit/ef80ea254005e84589dd310abe87ce0d3d2a0c90)
* [f03556549bd074f724d47901bd58d730d12998d6](https://code.videolan.org/GSoC2017/shalzz/vlc/commit/f03556549bd074f724d47901bd58d730d12998d6)
* [23f2e30f4a6241bc3476414ac187d1e7e8169dd3](https://code.videolan.org/GSoC2017/shalzz/vlc/commit/23f2e30f4a6241bc3476414ac187d1e7e8169dd3)

My main work repo has been [code.videolan.org/GSoC2017/shalzz/vlc][11] with all my commit history
in the master branch and final patches in the [release][12] branch.
There is a clone of the repo on [github][13] as well.

To run the code,
1. clone the repo and build vlc instrumented with one or more of the sanitizers using the new
`--with-sanitizer=` switch.
2. Get and build libFuzzer by changing into the test/fuzz/ directory and using the `./build-libfuzzer.sh` script
3. Build the fuzz target binaries by running `make check`.
4. Run the fuzz targets manually or by using the `./run-fuzzer.sh` helper script

#### Bug Trophy

As for the number of bugs found by the initial fuzzing done on my small laptop there have been quite a few, all of which have been fixed, by me and other VLC developers.
Few of them:
* [6c947b775d4d6c6ed07ebde140bddd3a2007b41a][3]
* [a672ea060efbb5898e1d80327f5909a43e8b57d8][4]
* [b2fb79e3201c5ce77a176b52936835ce195aa986][5]
* [1de4047a25cd336d1539ea0867c29180928dd230][6]
* [7033852e1a8292734e1d5800bec864bb5fb24c30][7]
* [3266d738e906b38b53f7936f58565441d0652713][8]
* [4b76784615f74254a7b66e34ff78393b44af2ed5][9]
* [7eea089393d6fc1de3436b4d486a466d14566a04][10]

#### Continuous Fuzzing

One thing that's left to do is setting up of a continuous fuzzing server.
Since OSS-fuzz is not in the picture right now, that is something VLC devs
will have to do on their own infrastructure.


[1]: http://git.videolan.org/?p=vlc.git;a=commit;h=2bc9957c615d9505f89d88511a0d6ef3889b392b

[3]: https://github.com/videolan/vlc/commit/6c947b775d4d6c6ed07ebde140bddd3a2007b41a
[4]: https://github.com/videolan/vlc/commit/a672ea060efbb5898e1d80327f5909a43e8b57d8
[5]: https://github.com/videolan/vlc/commit/b2fb79e3201c5ce77a176b52936835ce195aa986
[6]: https://github.com/videolan/vlc/commit/1de4047a25cd336d1539ea0867c29180928dd230
[7]: https://github.com/videolan/vlc/commit/7033852e1a8292734e1d5800bec864bb5fb24c30
[8]: https://github.com/videolan/vlc/commit/3266d738e906b38b53f7936f58565441d0652713
[9]: https://github.com/videolan/vlc/commit/4b76784615f74254a7b66e34ff78393b44af2ed5
[10]: https://github.com/videolan/vlc/commit/7eea089393d6fc1de3436b4d486a466d14566a04
[11]: https://code.videolan.org/GSoC2017/shalzz/vlc/
[12]: https://code.videolan.org/GSoC2017/shalzz/vlc/tree/release
[13]: https://github.com/Shalzz/vlc
[14]: https://github.com/videolan/vlc/commit/74e7bd240d5b239d0eeb3b67a7511b8b83cb6694
[15]: https://github.com/videolan/vlc/commit/b83e9fe08d12ae798390bfa64c08096801fcd8c1
