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
    organization: UCLouvain
    email: maxime.piraux@uclouvain.be

normative:
  RFC2119:
  QUIC-TRANSPORT: rfc9000
  QUIC-TLS: rfc9001

informative:
  MULTIPATH-QUIC: I-D.ietf-quic-multipath
  RFC8678:


--- abstract

This document specifies a QUIC Transport Parameter enabling a QUIC server
to advertise additional addresses that can be used for a QUIC connection.

--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overview

The additional_addresses transport parameter proposed in this document enables
a QUIC server to securely advertise additional addresses. These addresses can
be used by the client to migrate to a new server address at any time after
the handshake. When {{MULTIPATH-QUIC}} is used over a QUIC connection, the
client can use these addresses to establish new network paths.

When sending packets to a new server address, the client MUST validate the
address using Path Validation as described in {{Section 8.2 of QUIC-TRANSPORT}}.

# Additional Addresses Transport Parameter

The following transport parameters is defined:

additional_addresses (TBD - experiments use 0xadda):

: A list of server addresses that the client can migrate the connection to.
This transport parameter is only sent by a server.

~~~
Additional Addresses {
  Additional Address (..) ...,
}
~~~
{: #fig-additional-addresses title="Additional Addresses Format"}

~~~
Additional Address {
  Address Version (8),
  IP Address (..),
  IP Port (16),
}
~~~
{: #fig-additional-address title="Additional Address Format"}

Address Version:

: A 8-bit value identifying the Internet address version of this address. The
value 4 indicates IPv4 while 6 indicates IPv6.

IP Address:

: The address value. Its size depends on its version. IPv4 addresses are 32-bit
long while IPv6 addresses are 128-bit long.

IP Port:

: A 16-bit value representing the port to use with this IP Address.

# Security Considerations

This document specifies a mechanism allowing servers to influence the
IP addresses towards which clients send QUIC packets. In this case,
an attacker can cause a client to send packets to a victim. A countermeasure
similar to {{Section 21.5.3 of QUIC-TRANSPORT}} is to limit the packets that
can be sent to a non-validated additional addresses.

A client MUST NOT send non-probing frames to an additional address prior to
validating that address. The generic measures described in {{Section 21.5.6 of QUIC-TRANSPORT}}
also remain applicable for further mitigation.

# IANA Considerations

This document defines a new transport parameter for advertising additional addresses.
The draft defines a provisional identifier for experiments. IANA will allocate
the final identifier.

The following entry in {{transport-parameters}} should be added to
the "QUIC Transport Parameters" registry under the "QUIC Protocol" heading.

Value                        | Parameter Name.     | Specification
-----------------------------|---------------------|-----------------
TBD (experiments use 0xadda) | addition_addresses  | {{additional-addresses-transport-parameter}}
{: #transport-parameters title="Addition to QUIC Transport Parameters Entries"}


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
