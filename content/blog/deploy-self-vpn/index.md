+++
title="Why Deploy your Own VPN Server?"
date=2023-07-28
[extra]
tags="vpn, self-host, self deploy, deploy, selfvpn, self-deploy, algo, wireguard, ansible, google cloud shell, free vpn, free, zero-cost, virtual private network, gumroad"
+++

<figure>
{{ resize_image(path="cloud-shell.png") }}
<figcaption>SelfVPN script running on Google Cloud Shell</figcaption>
</figure>

### Background and Motivation 

VPNs have become a requirement in many countries and for many professionals across
countries to be able to do their work and maintain a level of safely and anonymity
that's increasing disparate with the modern internet.

However, VPN services provided by centralized VPN providers just shifts the
responsibility of maintaining user privacy and safely from decentralized
ISP's to centralized entities.

Many VPN providers still record logs of their users, either knowing or unknowing
for the purposes of having an analytical and billing system. As we have seen
time and time again, it is this metadata that many of these centralized entities
can be force to hand over and can successfully be used to reconstruct online
activity and user identity. More over they often provide a degraded user experience
by way of having shared "public" servers that many users are routed through to
the open internet simultaneously.

<!-- more -->

With the proliferation and scale of cloud computing providers that provide the
raw resources of a VPN server across geolocations, it no longer makes sense to
pay a specialized service provider for general resources with unit economics with
just a software slapped on top. Especially when the software that does most of the
heavy lifting is a Free and Open Source Software.

### Solution

While it does take some expertise to deploy your own VPN server solution on a
cloud provider, it is a one time cost that truly provides the benefits of a
VPN server and keeps you in control of your privacy and data across multiple
levels of the infrastructure.

Automation and abstraction continue to make it possible to deploy your own
server without increasing expertise and domain knowledge for a wider group of users.

[SelfVPN] is my attempt at providing a set of tools and guides that lets anyone
DIY their own VPN server while maintaining ownership and sovereignty of their
data and identity.

<video playsinline="" loop="" autoplay="" muted="" style="width:100%;height:auto;display:block;margin:0">
  <source src="freevpn-algo-x3-take8.mp4" type="video/mp4">
</video>
<br/>


[SelfVPN] deploys a [Wireguard][1] VPN server using the excellent [Algo][2] ansible
playbook by Trait of Bits consulting company.
It builds on their work to package the script in a cloud execution
environment with all the prerequisite dependencies and software pre-installed.
As well as bring all documentation together in a step by step manner.

Please give it a shot, and I appreciate any feedback I can get to make the setup
even easier to use.


[SelfVPN]: https://selfvpn.8bitlabs.tech/
[1]: https://www.wireguard.com/
[2]:  https://github.com/trailofbits/algo
