+++
title="Protecting Your Privacy in a Post-Aadhar India"
date=2018-08-19
[extra]
tags="aadhar, india, privacy, bharat, dummy, number, test, test number, fake number, fake, generate, aadhar number, uidai, aadhar card, card, protection, circumvent, virtual, virtual number, protect, private, generator, VID, virtual id, id"
+++

With more and more online and offline services demanding Indian citizens to provide
their Aadhar number for availing their services without any legal basis and
necessity under the pretense of proving your identity, it does raise more than
a few privacy concerns.

<!-- more -->

The unique Aadhar number that is assigned to a person and is printed on the Aadhar Card
is a 12 digit number where the first 11 digits are random and the last number is
a checksum based on [Verhoeff]'s algorithm ([source][1]) that can be used to verify the
validity of the complete number.

So to get a valid Aadhar number all you need is 11 random digits and their 
[Verhoeff] checksum as the 12th digit.

One example number that passes the above scheme and hence is valid is 
`999999990019`. This is a test number recommend by UIDAI but many more
numbers can be generated using the same scheme.

So if a system just checks the validity of a Aadhar number using the above scheme
you can safely enter `999999990019` and continue using their services while protecting
you privacy. That's what I did for using Amazon Pay :)

### Virtual Number

If you still feel like you have to use a legitimate number but want to minimise infringement
of your right to privacy, consider giving out a "Virtual Number" that is temporary
number that points to your actual Aadhar number without actually revealing your
real Aadhar number. You can use the Virtual Number/ID (VID) generator at the 
offical UIDAI [website][2] or the android [mobile app][3].

Also next time someone asks for a photocopy of your Aadhar card, tell them to f*ck off.

[1]: https://www.quora.com/What-is-the-structure-of-ones-Aadhar-Card-UID-number
[2]: https://resident.uidai.gov.in/web/resident/vidgeneration
[3]: https://play.google.com/store/apps/details?id=in.gov.uidai.mAadhaarPlus
[Verhoeff]: https://en.wikipedia.org/wiki/Verhoeff_algorithm
