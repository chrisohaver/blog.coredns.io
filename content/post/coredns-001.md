+++
date = "2016-09-18T11:40:37+01:00"
slug = "CoreDNS-001 Release Notes"
tags = ["Release", "001", "Notes"]
title = "CoreDNS-001 Release"
+++

CoreDNS-001 has been [released](https://github.com/miekg/coredns/releases). This is the first
release! It provides a complete DNS server, that also does DNSSEC and is useful for service
discovery in cloud setups.

# What is CoreDNS?

[CoreDNS](https://coredns.io) is a DNS server that started its life as a fork of the [Caddy
web(!)server](https://caddyserver.com).

It chains [middleware](https://github.com/miekg/coredns/tree/master/middleware),
where each middleware implements some DNS feature. CoreDNS is a complete replacement
(with more features, and maybe less bugs) for [SkyDNS](https://github.com/skynetservices/skydns).

It is also useful as a normal DNS server, featuring DNSSEC, on-the-fly signing and zone transfers.

# What is New

CoreDNS is now a (the first!) server type plugin in Caddy - this means we can leverage a lot of code
from Caddy without having to fork (and maintain) it all. By doing so we were able to remove 9000
lines of code from CoreDNS.

## New Middlewares

* There is now a [specific
  middleware](https://github.com/miekg/coredns/tree/master/middleware/kubernetes) to deal with [Kubernetes](https://kubernetes.io).
* The `bind` [middleware](https://github.com/miekg/coredns/tree/master/middleware/bind)  allows you to bind to a specific IP address, instead of using the wildcard
  address.
* All other middlewares are reworked to fit in the new plugin framework from Caddy version 0.9 (and
  up).

# Contributors

The follow people help with getting this release done:

Cricket Liu, elcore, Félix Cantournet, Ilya Dmitrichenko, Joe Blow, Lee, Matt Layher,
Michael Richmond, Miek Gieben, pixelbender, Yong Tang.

# Documentation and Help

The forum on <https://forum.caddyserver.com> has section for
[CoreDNS](https://forum.caddyserver.com/c/coredns), where you can submit questions and get answers.
Or use [Twitter](https://twitter.com/corednsio).
