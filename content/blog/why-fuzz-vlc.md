+++
title="VLC Media Player and Fuzz testing"
date=2017-05-15
updated=2017-07-08T00:00:00+05:30
[extra]
tags="libfuzzer, libvlc, vlc, fuzzing, gsoc-2017, gsoc"
+++
Software bugs and vulnerabilities can be difficult to detect and slow to find
even when actively searched for by developers and users who usually look for
superficial functional and visual bugs.

In a large software especially those written in middle level languages like C/C++,
security bugs and vulnerabilities can often be used to comprise the whole system.
Mainly because memory management is left to the programmers of the individual software.

One alternative to human Q&A testing is to use automated software testing techniques
like Fuzzing where random, invalid or unexpected data is provided as input to a computer
program.

<!-- more -->

Fuzzing is often more cost-effective than systematic testing 
techniques[^fn-cite_report_random_testing].
High profile CVE's such as [Heartbleed][1] in April 2014
and [Shellshock][2] in September 2014
could have easily been found with fuzzing[^fn-cite_heartbleed] [^fn-cite_shellshock].

Media processing is always a complex task and usually contain lots of security and
stability issues, take [Stagefright][3] or [FFmpeg and a thousand fixes][9], for example.

**Fuzzing VLC**, the most popular desktop and mobile media player should now seem
like a no-brainer.

Indeed, this was one of the project ideas and my proposal to VideoLAN for
Fuzz testing VLC as part of <abbr title="Google Summer of Code">GSoC</abbr> 2017.
VideoLAN accepted my [proposal][8] and invited me to their office
in Paris for a "GSoC conference" to discuss and help setting up the project
and get me started.

[^fn-cite_report_random_testing]: ["A report on random testing"][5]

[^fn-cite_heartbleed]: ["How Heartbleed could've been found (in English)"][6]

[^fn-cite_shellshock]: ["Bash bug: the other two RCEs, or how we chipped away at the original fix (CVE-2014-6277 and '78)"][7]

[1]: https://en.wikipedia.org/wiki/Heartbleed
[2]: https://en.wikipedia.org/wiki/Shellshock_(software_bug)
[3]: https://en.wikipedia.org/wiki/Stagefright_%28bug%29
[4]: https://github.com/google/sanitizers
[5]: http://dl.acm.org/citation.cfm?id=802530
[6]: https://blog.hboeck.de/archives/868-How-Heartbleed-couldve-been-found.html
[7]: http://lcamtuf.blogspot.in/2014/10/bash-bug-how-we-finally-cracked.html
[8]: https://summerofcode.withgoogle.com/projects/#5893995166171136
[9]: https://security.googleblog.com/2014/01/ffmpeg-and-thousand-fixes.html
