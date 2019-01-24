+++
title="Fuzzing libVLC - Improvements"
date=2017-07-08
[extra]
tags="libfuzzer, libvlc, vlc, fuzzing, gsoc-2017, gsoc"
+++

I have been making some few specific improvement to demux fuzz target
and also working on the decoder target side by side which isn't done yet.

Some general improvements that I have made for all fuzz targets is 
abstracting away and initializing the parent `libvlc_instance_t` object in
`LLVMFuzzerInitialize` which obviously has some performance improvements
as it isn't created and destroyed on every run of the `LLVMFuzzerTestOneInput`.

<!-- more -->

#### fuzzer_common.c
```c
#include <assert.h>

#include "fuzzer.h"
static libvlc_instance_t *p_libvlc;

extern int LLVMFuzzerInitialize(int *argc, char ***argv)
{
	setenv("VLC_PLUGIN_PATH", "../../modules", 1);
	p_libvlc = libvlc_new(0, NULL);
	assert(p_libvlc != NULL);
	return 0;
}

extern int LLVMFuzzerTestOneInput(const uint8_t *buf, size_t len) {
    return FuzzerTestOneInput(buf, len, p_libvlc);
}

void FuzzerCleanup(void)
{
    libvlc_release(p_libvlc);
}
```

Another benefit of this abstraction is that our fuzz targets are now completely
agnostic to the fuzzing engine we use which has been [libFuzzer][1] until now.
We can now use the exact same target with [AFL][2] by writing a driver,
since AFL expects a main function, here is an [example][3] driver.

### Dictionaries
At this point the demux fuzzer had found a few more bugs but it
had kind of reached a saturation point in terms of the line coverage it
had and it just wasn't increasing any more significantly with new sample input.

So I decided instead of adding sample files of every possible video 
file format with different types of chunks and atoms that VLC supports,
to create a keywords dictionary of magic values of the various file formats
and feed it to the demux target. Libfuzzer has in-built support for
dictionary files and can much more efficiently create structured inputs
with it.

libFuzzer and AFL have the same syntax of dictionary files and the AFL project
even has some excellent dictionaries for [JPEG][4], [PNG][6] and [GIF][5] file formats.
And you guessed it VLC can actually "Play" those files as well since `ffmpeg`
provides their codec as well.

I'm still working on the dictionary and I'll maybe post it once it's 
completed or if it's ever merged into the VLC codebase.

[1]: http://llvm.org/docs/LibFuzzer.html#startup-initialization
[2]: http://lcamtuf.coredump.cx/afl/
[3]: https://github.com/llvm-mirror/compiler-rt/blob/58d43607862096aeb32d72173911c9df244a30f1/lib/fuzzer/afl/afl_driver.cpp
[4]: https://github.com/mcarpenter/afl/blob/master/dictionaries/jpeg.dict
[5]: https://github.com/mcarpenter/afl/blob/master/dictionaries/gif.dict
[6]: https://github.com/mcarpenter/afl/blob/master/dictionaries/png.dict
