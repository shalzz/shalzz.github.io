+++
title="Continuous Marketing - Exploiting Insecure REST API's"
date=2018-07-08
[extra]
tags="api, marketing, rest api, python, sendgrid, sendgrid API, sendgrid python, programmatic, continuous, continuous marketing, exploit, insecure api, email, email promotion, marketing campaign, email campaign, automation, marketing automation"
+++

After I finished refactoring my android app to use a 3rd-party public API for the
user data,
I needed a way to quickly promote my application to users.

The API I used is inherently insecure with virtually no standard REST API security
in place and designed with a top-down approach to suit their own mobile app.
Their whole premise being to secure the mobile app with 2FA and having an API
that's quite literally a bunch of wrapper functions around database queries 
exposed as endpoints.

<!-- more -->

All their CRUD operations are implemented as GET methods (so much for being idempotent)
and their idea of security is just using a users mobile number to get their
unique user ID that they call "encrypturl". 2FA is just two separate endpoints
added to send and confirm the OTP send to the user's mobile, which again just
advance the mobile flow and are completely orthogonal to the API design.

So I exploited their API yet again to get the email ids of every user they have
in their system by trying every 10 digit combination with the 10th digit always
being either 7, 8 or 9 (majority of Indian mobile phone numbers) until I 
get a hit and receive the users unique id required to access the rest of the API
endpoints.

Using the user-id I get their email-ID from my own API that I built around their
API to have better object models. 
Once I have the email-id I proceed to send them an email promoting my application
designed using the Campaign and Template design editor of [Sendgrid] with
their awesome developer API and [python][2] library that made programmatically sending
emails a lot more easier.

So with the brute force method I found a new email-id every 5~6 seconds that 
translates into a new promotional email sent with every new email discovered.
And my "Continuous Marketing" is still running with over 15000 emails sent already.

If you as well every find and are willing to exploit an insecure API for at least
marketing gains perhaps this python script will make that a lot more easy for you.

{{ gist(url="https://gist.github.com/shalzz/ff60ae09162112db067b9463c76469e0") }}

[Sendgrid]: https://sendgrid.com
[2]: https://github.com/sendgrid/sendgrid-python
