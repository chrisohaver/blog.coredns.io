+++
date = "2017-2-22T21:26:11Z"
slug = "CoreDNS-007 Release Notes"
tags = ["Release", "007", "Notes"]
title = "CoreDNS-007 Release"
author = "miek"
+++

CoreDNS-007 has been [released](https://github.com/coredns/coredns/releases/tag/v007)!

CoreDNS is a DNS server that chains middleware, where each middleware implements a DNS feature.

CoreDNS is accepted as an inception project by the [CNCF](https://cncf.io)!

We moved repos: https://github.com/coredns is the main overarching repo. There is an automatic
redirect in place from the old repo.

And... We have a new logo! We're also discussion a website redesign for <https://coredns.io> and
this blog.

# Core

* `ServeDNS` is extended to take a context. This allows (for instance) tracing to start at an earlier entrypoint.
* gRPC and TLS are made first class citizens. See [the zone specification](https://github.com/coredns/coredns/blob/master/README.md#zone-specification) on how to use it. TL;DR using `grpc://` makes the server talk gRPC. The `tls` directive is used to specify TLS certifcates/
* Zipkin tracing can be enabled for all middleware.

# Middleware

* *rewrite* now allows you to add or modify EDNS0 local or NSID options. The framework is in place to add additional EDNS0 types in the future.
* *etcd* when no upstreams are defined won't default to using 8.8.8.8/8.8.4.4; it just does not resolve external names in that case.
* *erratic* now can also delay queries and send queries with Truncated set.
* *metrics* will happily start as many (different) listeners as you want (if you really need that).
* *startup* and *shutdown* allow for command to be run during startup or shutdown. These directly use the code from Caddy, see [Caddy's docs](https://caddyserver.com/docs/startup).
* *kubernetes* now implements a `fallthrough` option to pass queries that would result in NXDOMAIN to the next middleware, even if the query is in the cluster domain. This enables custom DNS entries in the cluster domain (as long as they do not overlap with a normal Kubernetes record). To facilitate this the middleware ordering is also altered to put *kubernetes* in the chain before other backends.
* *cache* will no longer cache RRSIGs that will expire while cached.

# Contributors

The following people helped with getting this release done:

TODO

If you want to help, please check out one of the [issues](https://github.com/coredns/coredns/issues/)
and start coding!

## Community

You find CoreDNS's community in the following places:

- Mailing list/group: <coredns-discuss@googlegroups.com>
- Slack: #coredns on <https://slack.cncf.io>
- Twitter: [@corednsio](https://twitter.com/corednsio)
- Github: <https://github.com/coredns/coredns>
