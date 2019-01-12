+++
title="Mobile and Desktop Wallets with gRPC"
date=2018-05-16
[extra]
tags="bitcoin, lnd, lightning network, lightning, network, wallets, gRPC, wallet, bitcoin wallet, lightning wallet"
+++

Does anyone else also feel that we need lightning [network] desktop and mobile app wallets that can connect to your remote lnd server with gRPC (using rpc username and password) instead of every app loading and syncing their own chain on device? Leading to multiple wallets and coin fragmentation among those wallets.

<!-- more -->

I know the neutrino light client mode exists but hardly any app at this point lets us use this mode with our own server. Especially with mobile wallets like Eclair that use a json api wrapper over the rpc protocol that just connects to only their own server. 

And when their [beta] android wallet does happen to work, I did not find any option to connect to my own server even if I am running the eclair server implementation of the lightning network.  If you are like me you might also be thinking this leads to some amount of centralization.

This is the same problem I felt with bitcoin and bitcoin-core where you had to have the full chain node to use a wallet, which lead to creation of cloud custodian wallets and client side encrypted cloud wallets.

With LND being the second layer over the bitcoin blockchain and needing to stay online as much as possible to be able to negotiate and terminate contracts if your channel peer is misbehaving or trying to steal you funds, it makes sense to have one heavily fortified full-node server running 24/7 and have the convenience and ease of use of desktop and mobile wallets pointing to your full-node.

What do you think?

> Originally posted on [yalls.org][1]

[1]: https://yalls.org/articles/a1264e7f-401d-4c46-97a7-7b14e6b54193:926B20A0-5375-4E8A-BC75-D060BD930D7C/e5d7864e-e316-4f4a-9db6-0b72410702c6
