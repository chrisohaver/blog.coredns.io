+++
date = "2017-02-20T20:46:48Z"
slug = "How to add middleware to CoreDNS; a tutorial"
tags = ["middleware", "howto", "tutorial"]
title = "How to Add Middleware to CoreDNS"
+++

CoreDNS is a DNS server that chains middleware. Middleware is defined as a method `ServeDNS()` that
gets a request and either responds to the client or passes it on to the next middleware. If none of
the middleware handle the request a default response of SERVFAIL is returned.

This blog post details how to add middleware to CoreDNS. We're using the example of the *whoami*
middleware which is a CoreDNS middleware and loaded by default if nothing is specified.

> Note all the code examples here are in Go because CoreDNS is written in the Go language.

First question should be: "What should my middleware do?". In the case of the *whoami* middleware
its purpose is to echo back the client's IP (IPv4 or IPv6), the transport used ("tcp" or "udp") and
the request port number.

Next question is: "What is the name of this new middleware?" Try to find a short, descriptive name
for this. In this case, we already had a (good) name: *whoami*.

## Middleware

A middleware consists of a number of parts:

1. The registration of the middleware
2. The setup function that parses the *whoami* directive and
    possible options from the Corefile
3. The `ServeDNS()` and `Name()` methods

After we've defined the parts, we can:

4. Hook it up
5. Use it

# 1. Registration

Typically a middleware has a file called `setup.go` that handles the registration.
In there, the `init` function should look like:

~~~ go
func init() {
	caddy.RegisterPlugin("whoami", caddy.Plugin{
		ServerType: "dns",
		Action:     setupWhoami,
	})
}
~~~
First you see that we're using `caddy.RegisterPlugin` which makes perfectly sense, because
CoreDNS uses the [Caddy](https://caddyserver.com) server framework. The first argument is the name
of the directive as used in the Corefile.

The `ServerType` is "dns" because CoreDNS is a DNS server; Caddy supports HTTP servers as well. The
`Action` is the name of the setup function that takes care of the parsing of the Corefile. Its job
is to return a type that implements the `middleware.Handler` interface.

So whenever the Corefile parser sees "whoami", `setupWhoami` is called.

# 2. Setup function

Because the *whoami* directive does not allow for any options, the setup function is relatively
simple:
~~~ go
func setupWhoami(c *caddy.Controller) error {
	c.Next() // 'whoami'
	if c.NextArg() {
		return middleware.Error("whoami", c.ArgErr())
	}

	dnsserver.GetConfig(c).AddMiddleware(func(next middleware.Handler) middleware.Handler {
		return Whoami{}
	})

	return nil
}
~~~
We use the `*caddy.Controller` to receive tokens from the Corefile and act upon them. Here we only
check if there is nothing specified after the token `whoami`. If you need to do more there are
`c.Val()`, `c.Args()` and [friends](https://godoc.org/github.com/mholt/caddy/caddyfile#Dispenser).

The full `setup.go` for *whoami* is [here](https://github.com/miekg/coredns/blob/master/middleware/whoami/setup.go).

Note that you should test the parsing as well, see
[setup_test.go](https://github.com/miekg/coredns/blob/master/middleware/whoami/setup_test.go).

# 3. ServeDNS() and Name()

Let's start with the trivial `Name()` method which is used (mainly) to see if the metrics
middleware is loaded. CoreDNS does by this by name. The method just returns the string `whoami`.

~~~ go
// Name implements the Handler interface.
func (wh Whoami) Name() string { return "whoami" }
~~~

Next, the meat of the whole thing, the `ServeDNS` method. We will look at this method line by line.

~~~
// ServeDNS implements the middleware.Handler interface.
func (wh Whoami) ServeDNS(ctx context.Context, w dns.ResponseWriter, r *dns.Msg) (int, error) {
	state := request.Request{W: w, Req: r}
~~~
As seen the function gets a couple of parameters of which `w` is the client side: writing to `w`
returns a response to the client. As you will see below all interesting properties of the client
connection can be retrieved from the
[dns.ResponseWriter](https://godoc.org/github.com/miekg/dns#ResponseWriter). `r` is the incoming
query. The `ServeDNS` method returns an integer and/or an error. The values the integer can take are
the DNS RCODEs, dns.RcodeServerFailure, dns.RcodeNotImplemented, dns.RcodeSuccess, etc..
A successful return value indicates the middleware has written to the client.

`request.Request` is a helper struct that abstracts and caches some client-side properties, like
EDNS0 records and the DNSSEC OK bit.

Next we setup the reply message:
~~~ go
a := &dns.Msg{}
a.SetReply(r)
a.Compress = true
a.Authoritative = true
~~~

We create a new message and copy the relevant bits from the incoming reply to the one we're planning
to return. We fiddle with some message bits and turn on message compression.

Then we are going to inspect the incoming message via the `state` helper struct to see what we
should return.

~~~ go
ip := state.IP()
var rr dns.RR

switch state.Family() {
case 1:
    rr = &dns.A{}
    rr.(*dns.A).Hdr = dns.RR_Header{Name: state.QName(),
            Rrtype: dns.TypeA, Class: state.QClass()}
    rr.(*dns.A).A = net.ParseIP(ip).To4()
case 2:
    rr = &dns.AAAA{}
    rr.(*dns.AAAA).Hdr = dns.RR_Header{Name: state.QName(),
            Rrtype: dns.TypeAAAA, Class: state.QClass()}
    rr.(*dns.AAAA).AAAA = net.ParseIP(ip)
}
~~~
`IP()` returns the IP address of the client. `Family()` returns the IP version used. Depending on the
family we either create a A or AAAA record with the client's address. Note we don't specify a TTL
meaning it will be zero; indicating these records should not be cached.

Next we want to encode the client's source port and transport protocol used.
~~~go
srv := &dns.SRV{}
srv.Hdr = dns.RR_Header{Name: "_" + state.Proto() + "." + state.QName(),
            Rrtype: dns.TypeSRV, Class: state.QClass()}
port, _ := strconv.Atoi(state.Port())
srv.Port = uint16(port)
srv.Target = "."
~~~

SRV records are ideal for that. The domain names are either prefixed with `_tcp` or `_udp`, and the
SRV record's port number re-used to echo back the clients's port number, meaning we create
something like this: `_tcp.example.org.	0	IN	SRV	0 0 <portNr>`.

In the last bit of this method we create the full message and send it:

~~~go
a.Extra = []dns.RR{rr, srv}

state.SizeAndDo(a)
w.WriteMsg(a)

return 0, nil
~~~

First we add the two created resource records (`rr` and `srv`) to the additional section of the
answer message: `a.Extra`.
Then we call `state.SizeOnDo(a)` which will set the correct EDNS0 record on the message and handle
setting the truncation bit). Then,
finally, we call the `WriteMsg` method on `w` which writes the message back to the client.
We indicate a successful (even though it might have failed - we didn't check the return value of
`WriteMsg`) write by returning 0 and `nil`.

# 4. Hooking it up

Next we need to tell CoreDNS to compile and use this new middleware. Adding a middleware was
recently simplified and only consists of editing `middleware.cfg` and adding the line:

~~~
210:whoami:whoami
~~~
The initial number is used for sorting the middleware (more on that below), then the name of the
plugin as used in the registration and then the package inside the CoreDNS middleware directory.

Each middleware has a place in the list of all other ones. For instance the caching
or the metrics middleware needs to come early, so that it can "see" the query and
response and do cachy or metricsy things with it. A *whoami* middleware is not that special and can
be placed relatively late in the list (hence the high number).

Now do a `make` (or `go gen && go build`) to get a `coredns` binary that can be used with our shiny new middleware.
This binary should include `dns.whoami` when called with `-plugins`.

# 5. Using it

Write a Corefile:

~~~ txt
. {
    whoami
}
~~~
In words this says: be authoritative for the root `.` and below, meaning all possible queries will
hit this stanza. And for each request call *whoami*.

Start CoreDNS with: `coredns -conf Corefile -dns.port 1053`. Couple of notes here. CoreDNS
will look for a Corefile in the current directory so the `-conf Corefile` is only given here for
completeness. The `-dns.port` will start CoreDNS on port 1053, so we don't need to run as root.

CoreDNS will output the following:
~~~ txt
.:1053
2017/02/19 17:44:31 [INFO] CoreDNS-005
CoreDNS-005
~~~
The `.:1053` indicates it has parsed our Corefile and is listening on port 1053 for queries for
the root `.` zone and below.

So, lets send it a query with `dig`:

~~~ sh
% dig +nocmd @localhost mx example.org -p1053 +noall +additional
example.org.		    0	IN	A	127.0.0.1
_udp.example.org.		0	IN	SRV	0 0 58359 .
~~~
Beautiful. It is working. Checking the response, this request was sent with IPv4 over UDP
from port 58359.

Let's try using TCP:
~~~ sh
% dig +nocmd @localhost a example.org -p1053 +noall +additional +tcp
example.org.		0	IN	A	127.0.0.1
_tcp.example.org.	0	IN	SRV	0 0 33435 .
~~~
Yes, it correctly sees we've used TCP this time (and of course a different port).

There are already a lot of different middleware in CoreDNS. New, exciting ones, are always welcome. So if you
have an idea open an issue on the tracker.

This is an updated version of the [previous
blog](https://blog.coredns.io/2016/12/19/writing-middleware-for-coredns/) about writing CoreDNS
middleware.
