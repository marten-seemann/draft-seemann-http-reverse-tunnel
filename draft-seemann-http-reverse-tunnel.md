---
title: "Reverse Tunneling over HTTP"
abbrev: "HTTP Reverse Tunnel"
category: std

docname: draft-seemann-http-reverse-tunnel-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Applications"
workgroup: "HyperText Transfer Protocol"
keyword:
 - HTTP reverse tunnel

author:
 -
    fullname: Marten Seemann
    email: martenseemann@gmail.com

normative:
  QUIC: RFC9000
  HTTP-SEMANTICS: RFC9110
  HTTP1.1: RFC7231
  HTTP2: RFC9113
  HTTP3: RFC9114
  HPACK: RFC7541
  QPACK: RFC9204

informative:
  WEBTRANSPORT-HTTP3: I-D.ietf-webtrans-http3


--- abstract

This document specifies an extension to HTTP that allows the establishment of
HTTP request streams in the reverse direction, i.e. from the server to the
client. It works on HTTP/1.1, HTTP/2 as well as HTTP/3.

--- middle

# Introduction

HTTP defines how clients can issue HTTP requests to servers. This involves
establishing a TCP connection (for HTTP/1.1 ({{HTTP1.1}}) and HTTP/2
({{HTTP2}})) or a QUIC connection ({{HTTP3}}) to the target server, on which the
request is then sent.

This model assumes that the server is publicly reachable. Clients cannot reach
servers behind a firewall if the firewall blocks incoming connections.

Traditionally, servers use virtual private networks (VPNs) to bypass firewall
restrictions, allowing TCP and/or QUIC connections from a limited set of
endpoints.

This document defines a way for the (partial) role reversal of the HTTP client
and server. The server, acting as an HTTP client, first establishes an HTTP/1.1,
HTTP/2, or HTTP/3 connection to the proxy. This is possible, since the firewall
only blocks incoming, but not outgoing connections. By reversing roles, the
proxy can send HTTP requests to the server.

For HTTP/1.1, this approach suffers from head-of-line blocking of requests, even
with request pipelining. The throughput can be increased by opening multiple
connections to the proxy.

For HTTP/2, the extension defined in this document modifies the stream state
machine to allow the proxy to open HTTP request streams to the server.

For HTTP/3, this document specifies a way for the proxy to open bidirectional
streams to the server, using the same mechanism that WebTransport over HTTP/3
({{WEBTRANSPORT-HTTP3}}) uses."

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Protocol Definition

## HTTP/1.1

HTTP/1.1 uses the HTTP ugprade mechanism (see {{Section 7.4 of
HTTP-SEMANTICS}}).

This document defines the "reverse" upgrade token. The method of the request
SHALL be "GET". The server accepts the request with a 101 (Switching Protocols)
response. Accepting the upgrade reverses the roles of client and server.

~~~
GET /reverse-http HTTP/1.1
Host: example.com
Connection: upgrade
Upgrade: reverse

HTTP/1.1 101 Switching Protocols
Connection: upgrade
Upgrade: reverse
~~~
{: #fig-http11 title="Establishing a Reverse Tunnel over HTTP/1.1"}


## HTTP/2

### Negotiating Extension Use

To indicate support for Reverse Tunneling over HTTP/2, the server (acting as the
HTTP/2 client) sends the REVERSE_TUNNEL (see {{http2-settings}}) setting with a
value greater than 0. It SHOULD allow a sufficient number of incoming
bidrectional streams (see {{Section 5.1.2 of HTTP2}}).

A sender MUST NOT send a REVERSE_TUNNEL setting with a value of 0 after
previously sending a value greater than 0.

### Opening Streams

The protocol defined in this documents extens the HTTP/2 streams state machine
(see {{Section 5.1 of HTTP2}}). Once the Reverse Tunnel extension is enabled,
the proxy can open a stream by sending a HEADERS frame. This stream is treated
as an HTTP/2 request stream. The proxy SHALL send an HTTP/2 request, acting as
an HTTP/2 client.

### HPACK Considerations

Endpoints use the same HPACK ({{HPACK}}) context for reverse streams as they use
for streams in the regular direction.


## HTTP/3

### Negotiating Extension Use

To indicate support for Reverse Tunneling over HTTP/3, the server (acting as the
HTTP/3 client) sends the REVERSE_TUNNEL setting with a value greater than 0. It
SHOULD allow a sufficient number of incoming bidirectional streams at the QUIC
layer (see {{Section 4.6 of QUIC}}).

### Opening Streams

The protocol defined in this document extends HTTP/3 by defining rules for
server-initiated bidirectional streams. Once the Reverse Tunnel extension is
enabled, the proxy can open a bidirectional stream and SHALL send a special
signal value, encoded as a variable-length integer, as the first bytes of the
stream to indicate how the remaining bytes on the stream are used.

The signal value, 0xf2b8cb, reverses the direction of this stream: The stream is
now treated as an HTTP/3 request stream in the reverse direction. The proxy
SHALL send an HTTP/3 request, acting as an HTTP/3 client (see {{Section 6.1 of
HTTP3}}).

~~~
Bidirectional Stream {
    Signal Value (i) = 0xf2b8cb,
    Stream Body (..)
}
~~~
{: #fig-http3-signal title="Bidirectional Reverse Tunnel stream format"}

This document reserves the special signal value 0xf2b8cb as a
REVERSE_TUNNEL_STREAM frame type. While it is registered as an HTTP/3 frame type
to avoid collisions, REVERSE_TUNNEL_STREAM is not a proper HTTP/3 frame, as it
lacks length; it is an extension of HTTP/3 frame syntax that MUST be supported
by any peer negotiating this extension.


### QPACK Considerations {#qpack}

Endpoints use the same QPACK ({{QPACK}}) context for reverse streams as they use
for streams in the regular direction.


# Security Considerations

TODO Security


# IANA Considerations

## Upgrade Token Registration {#upgrade-token}

The following entry is added to the "Hypertext Transfer Protocol (HTTP) Upgrade
Token Registry" registry established by {{Section 16.7 of HTTP-SEMANTICS}}.

The "reverse-tunnel" label identifies HTTP/3 used as a protocol for the Reverse Tunnel:

Value:

: reverse-tunnel

Description:

: Reverse Tunneling over HTTP

Reference:

: This document

## HTTP/2 SETTINGS Parameter Registration {#http2-settings}

The following entry is added to the "HTTP/2 Settings" registry established by
[HTTP2]:

Setting Name:

: REVERSE_TUNNEL

Value:

: 0xf2b8cb

Default:

: 0

Specification:

: This document

## HTTP/3 SETTINGS Parameter Registration {#http3-settings}

The following entry is added to the "HTTP/3 Settings" registry established by
[HTTP3]:

Setting Name:

: REVERSE_TUNNEL

Value:

: 0xf2b8cc

Default:

: 0

Specification:

: This document


## Frame Type Registration

The following entry is added to the "HTTP/3 Frame Type" registry established by
[HTTP3]:

The `REVERSE_TUNNEL_STREAM` frame is reserved for the purpose of avoiding
collision with the Reverse Tunnel extensions:

Code:

: 0xf2b8cd

Frame Type:

: REVERSE_TUNNEL_STREAM

Specification:

: This document


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
