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
  HTTP-SEMANTICS: RFC9110

informative:


--- abstract

TODO Abstract


--- middle

# Introduction

TODO Introduction


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



--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
