---
title: "Additional addresses for QUIC"
abbrev:
category: exp

docname: draft-piraux-quic-additional-addresses-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Transport"
workgroup: "QUIC"
keyword:
 - quic
 - address
venue:
  group: "QUIC"
  type: "Working Group"
  mail: "quic@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/quic/"
  github: "mpiraux/draft-piraux-quic-additional-addresses"
  latest: "https://mpiraux.github.io/draft-piraux-quic-additional-addresses/draft-piraux-quic-additional-addresses.html"

author:
 -
    fullname: Maxime Piraux
    organization: UCLouvain & WEL RI
    email: maxime.piraux@uclouvain.be
 -  fullname: Olivier Bonaventure
    organization: UCLouvain & WEL RI
    email: olivier.bonaventure@uclouvain.be

normative:
  RFC2119:
  QUIC-TRANSPORT: rfc9000

informative:
  MULTIPATH-QUIC: I-D.ietf-quic-multipath
  RFC8678:


--- abstract

This document specifies a QUIC frame enabling a QUIC server
to advertise additional addresses that can be used for a QUIC connection.

--- middle

# Introduction

The QUIC protocol specifies several techniques for network path migration.
The client can migrate from one of its local addresses to another at any time
after the handshake using connection migration. The server can transfer a
connection to one of its other addresses shortly after the handshake by using
the preferred_address transport parameter. However, it cannot advertise
additional addresses that a client may use.

This limitation impacts several scenarios. For instance, a multihomed server
that has access to several subnets cannot advertise all its addresses.
In entreprise deployments where provider-assigned IPv6 Addresses are used to
solve the multihoming problem {{RFC8678}}, announcing several server addresses
enables applications using QUIC to recover from provider failures.
Also, a dual-stack server cannot advertise its other address so that a client
losing the address family used to establish the connection can migrate to the
other address family.

This document proposes a QUIC frame and a QUIC transport parameter enabling
a QUIC server to advertise additional addresses that can be used for a QUIC
connection.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overview

The ADDITIONAL_ADDRESSES frame proposed in this document enables
a QUIC server to securely advertise additional addresses. The Additional
Addresses transport parameter enables a QUIC client to indicate support for
this frame.

These addresses can be used by the client to migrate to a new server address
at any time after the handshake. When {{MULTIPATH-QUIC}} is used over a QUIC
connection, the client can use these addresses to establish additional network
paths.

When sending packets to a new server address, the client validates the
address using Path Validation as described in {{Section 8.2 of QUIC-TRANSPORT}}.
When Preferred Adress and Additional Addresses are used together, the client
SHOULD NOT migrate to an additional address before acting on the preferred
address indicated by the server.

## Example of use

{{fig-example}} illustrates an example of use for Additional Addresses in a
QUIC deployment featuring a load balancer and a multihomed server
making use of the Preferred Address mechanism.

First, the client sends its Initial packet to the load balancer, which forwards
it to the first server IP. The client indicates support for this extension by
using the dedicated transport parameter. The server answers to the QUIC
connection opening and indicates its first IP as a preferred address and its
second one as an additional address using the dedicated frame. When the
handshake completes, the client validates the preferred address and migrates
to it. Later during the connection, the client can validate the path towards
the second server IP and can migrate to it.

~~~~
Client       Load-balancer @ IP lb        Server @ IP a   Server @ IP b
|                      |                      |               |
|       Initial[0]: CRYPTO(CH(Add.Addr))      |               |
|--------------------->|--------------------->|               |
                             ....
|      Handshake[0]: CRYPTO(EE(Pr.Addr=a),..) |               |
|<---------------------|<---------------------|               |
|         1-RTT[0]: ADDITIONAL_ADDRESSES([b]) |               |
|<---------------------|<---------------------|               |
                             ....
|       Handshake[0]: CRYPTO(Fin)             |               |
|--------------------->|--------------------->|               |
|    /* Migration to Preferred Address a */   |               |
|-------------------------------------------->|               |
                             ....
|                      .                      |
|                                             .               |
| 1-RTT[X]: PATH_CHALLENGE  /* Migration to Add. Address b */ |
|------------------------------------------------------------>|
|                                   1-RTT[Y]: PATH_RESPONSE   |
|<------------------------------------------------------------|
|                                                             |
~~~~
{: #fig-example title="A server reached through a load-balancer uses Add. Addresses"}

# Additional Addresses Transport Parameter

The following transport parameter is defined:

additional_addresses (TBD - experiments use 0x925addaXX):

: Indicates the support of the ADDITIONAL_ADDRESSES frame as defined in the -XX draft version of this document. This transport parameter has a zero-length value. It MUST NOT be sent by a server.

# ADDITIONAL_ADDRESSES Frames

The server uses an ADDITIONAL_ADDRESSES frame (type=TBD - experiments use 0x925addaXX)
to advertise the additional addresses that a client can use to reach it.
This frame MUST NOT be sent by a client and can only appear in 1-RTT packets.

~~~
Additional Addresses {
  Type (i) = TBD,
  Sequence Number (i),
  Additional Addresses Count (i),
  Additional Address (..) ...,
}
~~~
{: #fig-additional-addresses title="ADDITIONAL_ADDRESSES Frame Format"}

Sequence Number:

: A variable-length integer indicating the sequence of the frame. The number is
monotonically increasing within a QUIC connection and is chosen by the sender.
It helps the receiver to order ADDITIONAL_ADDRESSES frames by recency. A
receiver SHOULD ignore frames with a Sequence Number lower or equal to the
highest Sequence Number received.

Additional Addresses Count:

: A variable-length integer indicating the number of additional addresses in
the frame.

~~~
Additional Address {
  Address Version (8),
  IP Address (..),
  IP Port (16),
}
~~~
{: #fig-additional-address title="Additional Address Format"}

Address Version:

: An 8-bit value identifying the Internet address version of this address. The
value 4 indicates IPv4 while 6 indicates IPv6.

IP Address:

: The address value. Its size depends on its version. IPv4 addresses are 32-bit
long while IPv6 addresses are 128-bit long.

IP Port:

: A 16-bit value representing the port to use with this IP Address.

The ADDITIONAL_ADDRESSES frame is ack-eliciting. When a packet containing an
ADDITIONAL_ADDRESSES frame is lost and its content is still relevant, the sender
MAY retransmit the frame as is. Otherwise, sending a new frame with a new
Sequence number is preferred.

The server can update the client on its additional addresses at any time by
sending an ADDITIONAL_ADDRESSES frame. When a client is using one of these
additional addresses and receives an ADDITIONAL_ADDRESSES frame not containing
this address, it SHOULD stop using it in favor of another address.

# Security Considerations

This document specifies a mechanism allowing servers to influence the
IP addresses towards which clients send QUIC packets. In this case,
a malicious server could cause a client to send packets to a victim. A
countermeasure similar to {{Section 21.5.3 of QUIC-TRANSPORT}} is to limit
the packets that are sent to a non-validated additional addresses.

Given that a server can provide additional addresses at any point in time, a
malicious server could overload a client and direct it against many addresses.
To alleviate this, a client can choose to limit the number of addresses it
keeps track of and the frequency at which it considers them.

A client MUST NOT send non-probing frames to an additional address prior to
validating that address. The generic measures described in {{Section 21.5.6 of QUIC-TRANSPORT}}
also remain applicable for further mitigation.

# IANA Considerations

This document defines a new transport parameter for indicating support for
additional addresses.
The draft defines provisional identifiers for experiments. IANA will allocate
the final identifiers.

The following entry in {{transport-parameters}} should be added to
the "QUIC Transport Parameters" registry under the "QUIC" heading.

Value                        | Parameter Name.     | Specification
-----------------------------|---------------------|-----------------
TBD (experiments use 0x925addaXX) | additional_addresses  | {{additional-addresses-transport-parameter}}
{: #transport-parameters title="Addition to QUIC Transport Parameters Entries"}

The last byte of the experimental transport parameter ID is used by
implementations to indicate the version of this document they support.
For instance, the value 0x925adda01 indicates the support of the -01 version
of this document.

The following entry in {{transport-parameters}} should be added to
the "QUIC Frame Types" registry under the "QUIC" heading.

Value                        | Frame Type Name     | Specification
-----------------------------|---------------------|-----------------
TBD (experiments use 0x925addaXX) | ADDITIONAL_ADDRESSES  | {{additional-addresses-transport-parameter}}
{: #quic-frames title="Addition to QUIC Frame Types Entries"}

The last byte of the experimental frame type is used by
implementations to indicate the version of this document they support.
For instance, the value 0x925adda01 indicates the support of the -01 version
of this document.

--- back

# Acknowledgments
{:numbered="false"}

We thank Quentin De Coninck and François Michel for their feedback and
comments on the first version of this document.

We thank Marcel Kempf, Moritz Buhl, Louis Navarre and François Michel for
joining the interop during the IETF 118 Hackathon.
