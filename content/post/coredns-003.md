+++
date = "2016-11-11T16:38:32Z"
slug = "CoreDNS-003 Release Notes"
tags = ["Release", "003", "Notes"]
title = "CoreDNS-003 Release"
+++

CoreDNS-003 has been [released](https://github.com/miekg/coredns/releases)!

CoreDNS is a DNS server that chains middleware, where each middleware implements a DNS feature.

# What is New

## Core

Refused queries are properly logged and exported if metrics are enabled.

## Middleware improvements

* *proxy*: allow  `/etc/resolv.conf` to be used in the configuration.
* *metrics*: add tests and normalize some of the metrics. Removed the AXFR size metrics.
* *cache*: Added size and capacity of the cache (for both `denial` and `success` cache types).
  Don't cache meta data records and zone transfers.
* *dnssec*: metrics were unused, hooked them up: export size and capacity of the signature cache.
* *loadbalance*: balance MX records as well.
* *auto*: numerous bugfixes.
* *file*: fix data race in reload process and also reload a zone when it is `mv`ed (newly created) into place.
  Also rewrite the zone lookup algorithm and be more standards compliant, esp. in the area of DNSSEC, wildcards and empty-non-terminals; handle secure delegations.
* *kubernetes*: vender the k8s dependency and updates to be compatible with Kubernetes 1.4 and 1.5.
   Multiple cleanups and fixes. Kubernetes services can now be resolved.

# Contributors

The following people helped with getting this release done:

Ben Kochie,
Chris O'Haver,
John Belamaric,
Jonathan Dickinson,
Manuel Alejandro de Brito Fontes,
Michael Grosser,
Miek Gieben,
Yong Tang.

If you want to help, please check out one of the [issues](https://github.com/miekg/coredns/issues/)
and start coding!

# Documentation and Help

The forum on <https://forum.caddyserver.com> has section for
[CoreDNS](https://forum.caddyserver.com/c/coredns), where you can submit questions and get answers.
Use [Twitter](https://twitter.com/corednsio), or file an issue.
