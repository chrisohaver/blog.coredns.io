+++
date = "2017-02-22T21:26:11Z"
slug = "CoreDNS-006 Release Notes"
tags = ["Release", "006", "Notes"]
title = "CoreDNS-006 Release"
author = "miek"
+++

CoreDNS-006 has been [released](https://github.com/coredns/coredns/releases/tag/v006)!

CoreDNS is a DNS server that chains middleware, where each middleware implements a DNS feature.

# What is New

# Core

Move CoreDNS to <https://github.com/coredns/coredns> together with several other repos. This will be
the new home for CoreDNS development.

Fixed:

* Fix hot-reloading. This would fail with `[ERROR] SIGUSR1: listen tcp :53: bind: address already in
  use`.
* Allow removal of core middleware, see comments in
  [middleware.cfg](https://github.com/miekg/coredns/blob/master/middleware.cfg).

## Middleware improvements

### New

* *reverse* middleware: allows CoreDNS to respond dynamicly to an PTR request and the related
  A/AAAA request.

### Improvements/changes

* *proxy* a new `protocol`: `grpc`: speak DNS over gRPC. Server side impl. resides [in this out of
  tree middleware](https://github.com/coredns/coredns-grpc).
* *file* additional section processing for MX and SRV queries.
* *prometheus* fix hot reloading
* *trace* various improvements

# Contributors

The following people helped with getting this release done:

John Belamaric,
Miek Gieben,
Richard Hillmann,
Yong Tang,

If you want to help, please check out one of the [issues](https://github.com/coredns/coredns/issues/)
and start coding!

# Documentation and Help

The forum on <https://forum.caddyserver.com> has section for
[CoreDNS](https://forum.caddyserver.com/c/coredns), where you can submit questions and get answers.
Use [Twitter](https://twitter.com/corednsio), or file an issue.

