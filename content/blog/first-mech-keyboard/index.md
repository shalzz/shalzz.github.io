+++
title="My First Mechanical Keyboard - A Review of TADA68"
date=2018-06-03
[extra]
image="tada68-alu-top.jpg"
card="summary_large_image"
tags="mechanical, mechanical keyboard, cherry mx, gateron brown, tada68, india, in india, review, tada68 review, kbdfans review, kbdfans, cherry mx brown, manufacturing, manufacturing cherry mx in india, low-profile, aluminium case, low profile, aluminium, case, low, profile, keyboard, custom, duty, custom duty, import"
+++

<figure>
{{ resize_image(path="tada68-alu-top.jpg") }}
<figcaption> TADA68 with the aluminium case</figcaption>
</figure>

## Why TADA68? 

After first coming across the Keywalker IFD68 on [Massdrop] for just $99
I almost bought it. But after trying to find some reviews online I also
found the [TADA68/SABER68][TADA68] designed by originatives, which is a very similar keyboard
with the exact same 68 key layout and even the type and color of keycaps.

But the few differences that tipped the favor towards me buying the [TADA68]
were; first the Keywalker being a Bluetooth keyboard with an included battery
the shipping increased to be just too expensive, second the Keywalker uses a proprietary
firmware with a flashing tool to program the keyboard only available for windows
platform.

<!-- more -->

With the [TADA68] even though not having Bluetooth might seem a con it saves me
from the few quirks that Bluetooth generally has. Mainly depleting my laptop
battery faster than normal due to having the Bluetooth being on all the time and
possible input latency inherent in a shared medium communication channel. 
Besides I told myself if I ever really wanted Bluetooth in the future I could
always just hack together a adafruit feather 32u4 [bluefruit] which already has existing
support in [QMK] firmware.

Which leads me to definitely the most important factor
in my decision, the [TADA68] runs the [QMK] firmware which is an Open Source firmware
(forked from TMK) for keyboards with a myriad of additional features and
has a [flashing process][2] that is as simple as it gets with support for all major
platforms.
Also definitely a bonus, the [TADA68] already has a wide variety of aluminium
and wooden cases available in the market unlike the Keywalker IFD68.

Another major factor was the shipping time. With Massdrop the *expected* shipping
was after three months! For the TADA68 I found a well reputed vendor,
mentioned by various people on the mechanical keyboards subreddit, [kbdfans.cn][KBD]
(also sells on aliexpress) who already had the keyboard in stock.

With all these factors it was a no-brainer to buy the TADA68 and so I did!

<figure>
<div class="image-row">
    {{ resize_image(path="tada68-box.jpg",class="image-column") }} {{ resize_image(path="tada68-plastic-top.jpg",class="image-column") }}
</div>
<figcaption>TADA68 in its original box and on the right with its stock plastic case</figcaption>
</figure>

## The Keyboard and The Switches

I bought the [TADA68] keyboard with Gateron brown switches. I had originally planned
to buy a keyboard with Cherry MX brown switches having tested and liked the switch
on a 9 switch tester board with 9 different Cherry MX switches that I had bought
earlier from aliexpress. But kbdfans did not have the board with Cherry switches,
only Gateron switches which are Cherry MX clones by a Chinese company, which I had
never tried before.

<figure>
    {{ resize_image(path="tada68-switch.jpg") }}
<figcaption>Gateron Brown switches with white leds.</figcaption>
</figure>
</br>

Cherry and Gateron switches variants are mainly similar with both companies Blue,
Red, Brown switches having the same travel distance, actuation force, being linear
or tactile and so on. The things that are on spec are same but of course there are
subtle difference between the switches of the two companies due to manufacturing
differences and other advancement and changes they both have made over the years.

Having searched online for comparisons between the Cherry and Gateron Brown
switches mostly were positive reviews for the Gateron Brown with many saying it
does not have the scratchy feeling/sound that you get with the Cherry switches
and also it has a somewhat pronounced tactile feedback. So I took a risk in
good faith.
And indeed I feel the same about the Gateron Brown's not having that scratchy
sound that I hear with the Cherry Brown switches, kind of a plastic scratching with plastic
sound on every key press. But the tactile feedback did feel quite same to me on both.

The Gateron Brown's 45g actuation force does feel a little light when coming from
my laptop's membrane switches with what feels like 100g of force,
but they feel good and relaxed after getting some used to. I can also see why
MX Clears with 65g of actuation force with the same tactile feedback as Browns would also
appeal to some people as it is a nice balance between being easy to press
and not having spurious key presses while just resting you fingers (Something
I haven't experience with Browns till now).

Along with the keyboard I also bought the [low-profile silver aluminium case][3]
sold by kbdfans, least it got sold out later. And I am definitely glad I did.
An aluminium case gives any keyboard the rigidity and durability that instantly makes
it a premium product. I particularly like a low-profile case than a high-profile one
giving others a glimpse of the excellent technology and engineering that goes into the switches
and the keyboard as a whole, adding to the mechanical keyboard aura.

{{ resize_image(path="tada68-alu-profile.jpg") }}
{{ resize_image(path="tada68-alu-profile-angled.jpg") }}

Switching out the plastic case with the aluminium one I found out that two screws
were "missing" in the sense that they were not screwed into the cases but stuck between
the PCB and the steel plate. Which can definitely happen while screwing if you
do not have a magnetic screw driver. The fact that the aluminium case came with
aluminium screws is certainly not very help since aluminium is a nonferrous metal
and is not attracted to magnets. So I had to fish out the screws stuck between
the plates since they were indeed made of a ferrous metal and used them to replace
the case.

<figure>
    {{ resize_image(path="tada68-alucase-bottomfeet.jpg") }}
<figcaption>The bottom of the aluminium case with the anodised feet.</figcaption>
</figure>
<br/>

Buying a mechanical keyboard right now is just perfect as I have just finished
setting up [vim and tmux][vim-blog], requiring extensive use of a
keyboard.

So far it has been only a day and I am really enjoying my new keyboard and 
definitely have had an increase in my productivity, supported
by the fact how long this blog post has already become which I am of course
writing using my new keyboard.

## The KBDfans store

I have to mention how awesome my experience has been ordering from [kbdfans.cn][KBD].
Even though it is a Chinese company the website is very well made
and has I think only an English version which is very well written. There 
are a few details that you may find missing in the product descriptions but they are
generally very active on their FB messenger chat and answer any questions that you may
have.

But perhaps the most amazing thing about them was that after I told them
I am an international customer they happily offered to under declare the product
value for custom duty. Which is something I wasn't even aware 
the sellers could do! After briefly going through the custom duty laws of my
country and coming across the clause "... satisfaction of the Customs authorities with the truth and
accuracy of the Declared Value." ([Rule 4.1 of the Customs Valuation][1]) I thought
lets try not be too suspicious and told them to declare a value of around $49.9.
But they anyways declared the value as low as they usually do, I guess, to that of
$15! And guess what? It got through!

So I had to pay ~50% duty (which is the general duty rate on anything for
"personal use") on $15 and an unsurprisingly surprise ₹500 "Disbursement fee"
by DHL which I didn't see (or notice?) the last time I had imported something
and was completely ignorant about imports and custom duty.

Anyhow in the end the total amount I had to pay for "duty" was 10% of the actual
amount I paid which is definitely great considering how expensive mechanical keyboards
are and how there are really no other alternatives (atleast in India) that would
benefit from trade restrictions. Which leads to my next sections and a general
rant about the socio-economic state of my country. (You have been warned!)

## Manufacturing in India

There are no Cherry MX clones being designed and made in India even after the 
original Cherry MX [patent][12] has been expired for years.

There is an Indian manufacture of the original Cherry MX switches which is a 
Joint Venture between Cherry (ZF Electronics Corporation) and TVS Group, India
called [ZF Electronics TVS (India)][6]. Taking a look at their website, it
already [hacked][4] by someone 6 months ago and not yet fixed. There also seems
to be a user flow where you can apparently add the [items][7] to a cart and "checkout",
which is a completely bogus flow since their is no price information.
So of course don't do that unless you want to be spammed and possible be harassed
at your home address.

Assuming the company is still operational with perhaps a lacking IT department, they
are mainly a manufacturing company so it would make sense they only sell in wholesale
and bulk. Their alternate legitimate looking product page for [MX switches][5]
has an option to run a product query and they also seem to be active on
[IndiaMart][9]. So it should be possible for a consumer/e-commerce company
to procure the switches in bulk and offer them to consumers.

Unless of course they keep telling themselves that there is not enough demand
in India for "things like these".

The only e-commerce site I could find that sells Cherry MX switches is
[thingbits.net][10] since all their product pages have titles "... in India"
If you look at their price you'll see it is way to expensive since they import
all their products and sell it at double the price including the custom duty.

Buying just 68 switches can cost you ₹6,338.96 or $93, at this price just 
[buying][11] and importing with the ~50% import duty on 68 switches you can 
save more than 1/4th of the thingsbit price. Their business model of product 
delivery in 3-4 days instead of the 18-28 days (usually faster for the Indian 
subcontinent) with free shipping on [aliexpress](https://aliexpress.com), 
with 25% premium doesn't make a lot of sense.
Especially since aliexpress has the buyer protection and return/refund policies like
any other major e-commerce site.

I have sent them an email urging to source locally and pointed to the suppliers
so that they can get them cheap and pass on the benefit to their customers. 
But I haven't heard anything from them yet.

 With the government placing restrictions on trade in order to support
local manufacturers and now the "Make in India" movement, their is still no focus
on the bigger picture of encouraging the development of intellectual property 
used in consumer products creating the need for manufacturing.
For example Tata Hitachi and Tata Marcopolo are one of many of the existing Indian breed
industrial companies manufacturing locally but still have JV with Japanese and German
companied providing the patents, technology and products that are actually
manufactured.

For now all we can do is import until there is a boom of the manufacturing industry
in India and the people get over the illusion that skipping over the manufacturing
sector and directly into the service sector by a country as a whole is a good sign of
development.

[vim-blog]: @/blog/vim-and-tmux/index.md
[bluefruit]: https://www.adafruit.com/product/2829
[Massdrop]: https://www.massdrop.com/buy/keywalker-68-bluetooth-mechanical-keyboard#description?referer=8MUP7X
[TADA68]: https://kbdfans.cn/products/tada68-mechanical-keyboard-gateron-swtich-65-layout-dye-sub-keycaps-cherry-profils?spr_ref=5b12bce
[KBD]: https://kbdfans.myshopify.com?spr_ref=5b12bce
[QMK]: https://github.com/qmk/qmk_firmware
[1]: http://www.customsandforeigntrade.com/Customs%20Valuation.pdf
[2]: https://github.com/ravicious/TADA68
[3]: https://kbdfans.cn/products/kbdfans-tada68-aluminum-case?variant=2403415687181&spr_ref=5b12bce
[4]: http://zftvs.co.in/category/uncategorized/
[5]: http://zftvs.co.in/ZFTVS1/MX.php
[6]: http://zftvs.co.in/ZFTVS1/default.php
[7]: http://zftvs.co.in/product/xs-trackball-keyboard-g84-5400/
[8]: http://zftvs.co.in/product/mx-series/
[9]: https://www.indiamart.com/proddetail/key-switch-mx-series-8369799588.html
[10]: https://www.thingbits.net/products/cherry-mx-switch
[11]: https://www.aliexpress.com/item/Original-Cherry-mx-switch-3-pin-mechanical-keyboard-brown-blue-red-white-clear-silver-slilent-black/32839844484.html?spm=2114.search0104.3.3.53614dcejrotv0&ws_ab_test=searchweb0_0,searchweb201602_2_10152_10151_10065_10344_10068_10342_10343_10059_10340_10341_308_10696_100031_10084_10083_10103_10618_10624_10307_10623_10622_10621_10620,searchweb201603_25,ppcSwitch_5&algo_expid=9cf6af5d-4175-436a-8131-544b32d0aa64-0&algo_pvid=9cf6af5d-4175-436a-8131-544b32d0aa64&priceBeautifyAB=0
[12]: https://patents.google.com/patent/US4467160
