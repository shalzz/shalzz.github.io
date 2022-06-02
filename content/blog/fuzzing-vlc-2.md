+++
title="Fuzzing libVLC - De-multiplexer"
date=2017-06-24
updated=2017-07-05T00:00:00+05:30
[extra]
tags="libfuzzer, libvlc, vlc, fuzzing, gsoc-2017, gsoc"
+++

So, I have been working on writing a fuzz target for the demux API for the past
two weeks and I have reached a point where it has a good code coverage
and has already found some few shallow bugs. 

Here are some of the stacktraces libfuzzer spit out and all of them were fixed
rather promptly.

<!-- more -->

A Segfault on a null pointer, fixed by adding a [null check][1]

```
==25241==ERROR: AddressSanitizer: SEGV on unknown address 0xffffffffffffff68 (pc 0x7f56c19e8ab0 bp 0xffffffffffffff58 sp 0x7ffcdfe59e58 T0)
==25241==The signal is caused by a READ memory access.
    #0 0x7f56c19e8aaf  (/usr/lib/libpthread.so.0+0x9aaf)
    #1 0x7f56c275c5e1 in Lookup misc/variables.c:158
    #2 0x7f56c275c5e1 in AddCallback misc/variables.c:839
    #3 0x7f56c275c5e1 in var_AddCallback misc/variables.c:884
    #4 0x7f56b336a55e in blurayOpen access/bluray.c:674
    #5 0x7f56c26df59d in module_load modules/modules.c:183
    #6 0x7f56c26dfb43 in vlc_module_load modules/modules.c:275
    #7 0x7f56c2704dec in demux_NewAdvanced input/demux.c:259
    #8 0x7f56c2705236 in demux_New input/demux.c:151
    #9 0x40575b in LLVMFuzzerTestOneInput /home/shalzz/builds/vlc/test/fuzz/libvlc_demux_fuzzer.cpp:73
    #10 0x416714 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) FuzzerLoop.cpp:458
    #11 0x416983 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long) FuzzerLoop.cpp:397
    #12 0x41714d in fuzzer::Fuzzer::MutateAndTestOne() FuzzerLoop.cpp:596
    #13 0x417397 in fuzzer::Fuzzer::Loop() FuzzerLoop.cpp:624
    #14 0x410438 in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) FuzzerDriver.cpp:744
    #15 0x405c40 in main FuzzerMain.cpp:20
    #16 0x7f56c0089439 in __libc_start_main (/usr/lib/libc.so.6+0x20439)
    #17 0x404ee9 in _start (/home/shalzz/builds/vlc/test/fuzz/.libs/lt-libvlc_demux_fuzzer+0x404ee9)
```
 
 Abort due to character conversion without a conversion descriptor, [fixed][2].
```
==32703== ERROR: libFuzzer: deadly signal
    #0 0x7f46cd57fb68 in __sanitizer_print_stack_trace /build/gcc-multilib/src/gcc/libsanitizer/asan/asan_stack.cc:36
    #1 0x468961 in fuzzer::Fuzzer::CrashCallback() FuzzerLoop.cpp:195
    #2 0x46892d in fuzzer::Fuzzer::StaticCrashSignalCallback() FuzzerLoop.cpp:179
    #3 0x7f46cc83993f  (/usr/lib/libpthread.so.0+0x1193f)
    #4 0x7f46cbf8d66f in raise (/usr/lib/libc.so.6+0x3366f)
    #5 0x7f46cbf8ecff in __GI_abort (/usr/lib/libc.so.6+0x34cff)
    #6 0x7f46cce9996f in vlc_iconv extras/libc.c:391
    #7 0x7f46ccf3e083 in vlc_stream_ReadLine input/stream.c:321
    #8 0x7f4690b96342 in peek_Readline demux/subtitle_helper.h:38
    #9 0x7f4690b96b02 in Open demux/vobsub.c:128
    #10 0x7f46cce9b2dc in generic_start modules/modules.c:356
    #11 0x7f46cce9a508 in module_load modules/modules.c:183
    #12 0x7f46cce9abd6 in vlc_module_load modules/modules.c:279
    #13 0x7f46cce9b42d in module_need modules/modules.c:371
    #14 0x7f46cceef2bd in demux_NewAdvanced input/demux.c:259
    #15 0x7f46cceeeadd in demux_New input/demux.c:151
    #16 0x458a46 in LLVMFuzzerTestOneInput (/home/shalzz/builds/vlc-clean/test/fuzz/libvlc_demux_fuzzer+0x458a46)
    #17 0x469754 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) FuzzerLoop.cpp:458
    #18 0x4699c3 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long) FuzzerLoop.cpp:397
    #19 0x45f421 in fuzzer::RunOneTest(fuzzer::Fuzzer*, char const*, unsigned long) FuzzerDriver.cpp:268
    #20 0x463034 in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) FuzzerDriver.cpp:683
    #21 0x458c80 in main FuzzerMain.cpp:20
    #22 0x7f46cbf7a439 in __libc_start_main (/usr/lib/libc.so.6+0x20439)
    #23 0x458239 in _start (/home/shalzz/builds/vlc-clean/test/fuzz/libvlc_demux_fuzzer+0x458239)
```

A heap-use-after-free, fixed by [this][3] and [this][4].
```
==26135==ERROR: AddressSanitizer: heap-use-after-free on address 0x613000005a00 at pc 0x7f4549af4c64 bp 0x7ffcb951f110 sp 0x7ffcb951e8b8
    READ of size 4 at 0x613000005a00 thread T0
        #0 0x7f4549af4c63 in __interceptor_memcmp /build/gcc-multilib/src/gcc/libsanitizer/sanitizer_common/sanitizer_common_interceptors.inc:626
        #1 0x7f453871981c in Open demux/image.c:640
        #2 0x7f4548ffdb26 in generic_start modules/modules.c:356
        #3 0x7f4548ffca56 in module_load modules/modules.c:183
        #4 0x7f4548ffd2e5 in vlc_module_load modules/modules.c:279
        #5 0x7f4548ffdc77 in module_need modules/modules.c:371
        #6 0x7f4549076c84 in demux_NewAdvanced input/demux.c:259
        #7 0x7f4549075fc2 in demux_New input/demux.c:151
        #8 0x405ce5 in LLVMFuzzerTestOneInput /home/shalzz/builds/vlc/test/fuzz/libvlc_demux_fuzzer.cpp:68
        #9 0x4169d4 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) FuzzerLoop.cpp:458
        #10 0x416c43 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long) FuzzerLoop.cpp:397
        #11 0x41740d in fuzzer::Fuzzer::MutateAndTestOne() FuzzerLoop.cpp:596
        #12 0x417657 in fuzzer::Fuzzer::Loop() FuzzerLoop.cpp:624
        #13 0x4106f8 in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) FuzzerDriver.cpp:744
        #14 0x405f00 in main FuzzerMain.cpp:20
        #15 0x7f45464b0439 in __libc_start_main (/usr/lib/libc.so.6+0x20439)
        #16 0x404e49 in _start (/home/shalzz/builds/vlc/test/fuzz/.libs/lt-libvlc_demux_fuzzer+0x404e49)
     
    0x613000005a00 is located 128 bytes inside of 380-byte region [0x613000005980,0x613000005afc)
    freed by thread T0 here:
        #0 0x7f4549b234c8 in __interceptor_free /build/gcc-multilib/src/gcc/libsanitizer/asan/asan_malloc_linux.cc:45
        #1 0x7f4549193fc7 in block_generic_Release misc/block.c:99
        #2 0x7f4549193a3c in block_Release ../include/vlc_block.h:184
        #3 0x7f4549194de2 in block_TryRealloc misc/block.c:212
        #4 0x7f45490e89bb in vlc_stream_Peek input/stream.c:506
        #5 0x7f4538717037 in IsPnm demux/image.c:354
        #6 0x7f45387195bd in Open demux/image.c:634
        #7 0x7f4548ffdb26 in generic_start modules/modules.c:356
        #8 0x7f4548ffca56 in module_load modules/modules.c:183
        #9 0x7f4548ffd2e5 in vlc_module_load modules/modules.c:279
        #10 0x7f4548ffdc77 in module_need modules/modules.c:371
        #11 0x7f4549076c84 in demux_NewAdvanced input/demux.c:259
        #12 0x7f4549075fc2 in demux_New input/demux.c:151
        #13 0x405ce5 in LLVMFuzzerTestOneInput /home/shalzz/builds/vlc/test/fuzz/libvlc_demux_fuzzer.cpp:68
        #14 0x4169d4 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) FuzzerLoop.cpp:458
        #15 0x416c43 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long) FuzzerLoop.cpp:397
        #16 0x41740d in fuzzer::Fuzzer::MutateAndTestOne() FuzzerLoop.cpp:596
        #17 0x417657 in fuzzer::Fuzzer::Loop() FuzzerLoop.cpp:624
        #18 0x4106f8 in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) FuzzerDriver.cpp:744
        #19 0x405f00 in main FuzzerMain.cpp:20
        #20 0x7f45464b0439 in __libc_start_main (/usr/lib/libc.so.6+0x20439)
     
    previously allocated by thread T0 here:
        #0 0x7f4549b23860 in __interceptor_malloc /build/gcc-multilib/src/gcc/libsanitizer/asan/asan_malloc_linux.cc:62
        #1 0x7f4549194434 in block_Alloc misc/block.c:128
        #2 0x7f4549194c77 in block_TryRealloc misc/block.c:205
        #3 0x7f45490e89bb in vlc_stream_Peek input/stream.c:506
        #4 0x7f4539fc5e24 in DetectPVRHeadersAndHeaderSize demux/mpeg/ts.c:256
        #5 0x7f4539fc6272 in Open demux/mpeg/ts.c:357
        #6 0x7f4548ffdb26 in generic_start modules/modules.c:356
        #7 0x7f4548ffca56 in module_load modules/modules.c:183
        #8 0x7f4548ffd2e5 in vlc_module_load modules/modules.c:279
        #9 0x7f4548ffdc77 in module_need modules/modules.c:371
        #10 0x7f4549076c84 in demux_NewAdvanced input/demux.c:259
        #11 0x7f4549075fc2 in demux_New input/demux.c:151
        #12 0x405ce5 in LLVMFuzzerTestOneInput /home/shalzz/builds/vlc/test/fuzz/libvlc_demux_fuzzer.cpp:68
        #13 0x4169d4 in fuzzer::Fuzzer::ExecuteCallback(unsigned char const*, unsigned long) FuzzerLoop.cpp:458
        #14 0x416c43 in fuzzer::Fuzzer::RunOne(unsigned char const*, unsigned long) FuzzerLoop.cpp:397
        #15 0x41740d in fuzzer::Fuzzer::MutateAndTestOne() FuzzerLoop.cpp:596
        #16 0x417657 in fuzzer::Fuzzer::Loop() FuzzerLoop.cpp:624
        #17 0x4106f8 in fuzzer::FuzzerDriver(int*, char***, int (*)(unsigned char const*, unsigned long)) FuzzerDriver.cpp:744
        #18 0x405f00 in main FuzzerMain.cpp:20
        #19 0x7f45464b0439 in __libc_start_main (/usr/lib/libc.so.6+0x20439)
     
    SUMMARY: AddressSanitizer: heap-use-after-free /build/gcc-multilib/src/gcc/libsanitizer/sanitizer_common/sanitizer_common_interceptors.inc:626 in __interceptor_memcmp
    Shadow bytes around the buggy address:
      0x0c267fff8af0: fa fa fa fa fa fa fa fa 00 00 00 00 00 00 00 00
      0x0c267fff8b00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
      0x0c267fff8b10: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
      0x0c267fff8b20: 00 00 00 00 00 fa fa fa fa fa fa fa fa fa fa fa
      0x0c267fff8b30: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
    =>0x0c267fff8b40:[fd]fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
      0x0c267fff8b50: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
      0x0c267fff8b60: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
      0x0c267fff8b70: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
      0x0c267fff8b80: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
      0x0c267fff8b90: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
    Shadow byte legend (one shadow byte represents 8 application bytes):
      Addressable:           00
      Partially addressable: 01 02 03 04 05 06 07
      Heap left redzone:       fa
      Freed heap region:       fd
      Stack left redzone:      f1
      Stack mid redzone:       f2
      Stack right redzone:     f3
      Stack after return:      f5
      Stack use after scope:   f8
      Global redzone:          f9
      Global init order:       f6
      Poisoned by user:        f7
      Container overflow:      fc
      Array cookie:            ac
      Intra object redzone:    bb
      ASan internal:           fe
      Left alloca redzone:     ca
      Right alloca redzone:    cb
    ==26135==ABORTING
```

Unfortunately I cannot publish my source code yet or start integrating with oss-fuzz and cluster fuzz
by Google as VideoLAN/Labs is still undecided and would rather do this privately, mostly because they
don't want to be bound by Google's [bug disclosure][5] policy.

<del>I think they are just confused that they have to do a software release within the specified
deadline when it's just a bug disclosure, but whatever.</del>
(There's no longer any confusion.)

> Update: You can find all my bug fixes and patches [here][6]

[1]: https://github.com/videolan/vlc/commit/6c947b775d4d6c6ed07ebde140bddd3a2007b41a
[2]: https://github.com/videolan/vlc/commit/a672ea060efbb5898e1d80327f5909a43e8b57d8
[3]: https://github.com/videolan/vlc/commit/b2fb79e3201c5ce77a176b52936835ce195aa986
[4]: https://github.com/videolan/vlc/commit/1de4047a25cd336d1539ea0867c29180928dd230
[5]: https://googleprojectzero.blogspot.in/2015/02/feedback-and-data-driven-updates-to.html
[6]: https://github.com/videolan/vlc/commits/master?author=Shalzz
