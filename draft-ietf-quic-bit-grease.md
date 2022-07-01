---
title: "Greasing the QUIC Bit"
docname: draft-ietf-quic-bit-grease-latest
category: std

ipr: trust200902
area: TSV
workgroup: quic
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: M. Thomson
    name: Martin Thomson
    organization: Mozilla
    email: mt@lowentropy.net

normative:

informative:


--- abstract

This document describes a method for negotiating the ability to send an
arbitrary value for the second-to-most significant bit in QUIC packets.


--- middle

# Introduction

QUIC {{!QUIC=RFC9000}} intentionally describes a very narrow set of fields that
are visible to entities other than endpoints.  Beyond those characteristics that
are defined as invariant {{?QUIC-INVARIANTS=RFC8999}}, very little about the
"wire image" {{?RFC8546}} of QUIC is visible.

The second-to-most significant bit of the first byte in every QUIC packet is
defined as having a fixed value in QUIC version 1 {{!QUIC}}.  The purpose of
having a fixed value is to allow endpoints to efficiently distinguish QUIC from
other protocols; see {{?DEMUX=I-D.ietf-avtcore-rfc7983bis}} for a description of
a system that might use this property.  As this bit can identify a packet as
QUIC, it is sometimes referred to as the "QUIC Bit".

Where endpoints and the intermediaries that support them do not depend on the
QUIC Bit having a fixed value, sending the same value in every packet is more of
liability than an asset.  If systems come to depend on a fixed value, then it
might become infeasible to define a version of QUIC that attributes semantics to
this bit.

In order to safeguard future use of this bit, this document defines a QUIC
transport parameter that indicates that an endpoint is willing to receive QUIC
packets containing any value for this bit.  By sending different values for this
bit, the hope is that the value will remain available for future use
{{?USE-IT=RFC9170}}.


# Conventions and Definitions

{::boilerplate bcp14}

This document uses terms and notational conventions from {{QUIC}}.


# The Grease QUIC Bit Transport Parameter

The grease_quic_bit transport parameter (0x2ab2) can be sent by both client and
server.  The transport parameter is sent with an empty value; an endpoint that
understands this transport parameter MUST treat receipt of a non-empty value of
the transport parameter as a connection error of type TRANSPORT_PARAMETER_ERROR.

An endpoint that advertises the grease_quic_bit transport parameter MUST accept
packets with the QUIC Bit set to a value of 0.  The QUIC Bit is defined as the
second-to-most significant bit of the first byte of QUIC packets (that is, the
value 0x40).


## Clearing the QUIC Bit

Endpoints that receive the grease_quic_bit transport parameter from a peer
SHOULD set the QUIC Bit to an unpredictable value unless another extension
assigns specific meaning to the value of the bit.

Endpoints can set the QUIC Bit to 0 on all packets that are sent after receiving
and processing transport parameters. This could include Initial, Handshake, and
Retry packets.

A client MAY also set the QUIC Bit to 0 in Initial, Handshake, or 0-RTT packets
that are sent prior to receiving transport parameters from the server.  However,
a client MUST NOT set the QUIC Bit to 0 unless the Initial packets it sends
include a token provided by the server in a NEW_TOKEN frame ({{Section 19.7 of
QUIC}}), received less than 604800 seconds (7 days) prior on a connection where
the server also included the grease_quic_bit transport parameter.

{:aside}
> This 7 day limit allows for changes in server configuration.  If server
> configuration changes and a client does not set the QUIC Bit, then it is
> possible that a server will drop packets, resulting in connection failures.

A server MUST set the QUIC bit to 0 only after processing transport parameters
from a client.  A server MUST NOT remember that a client negotiated the
extension in a previous connection and set the QUIC Bit to 0 based on that
information.

An endpoint MUST NOT set the QUIC Bit to 0 without knowing whether the peer
supports the extension.  As Stateless Reset packets ({{Section 10.3 of QUIC}})
are only used after a loss of connection state, endpoints are unlikely to be
able to set the QUIC Bit to 0 on Stateless Reset packets.


## Using the QUIC Bit

The purpose of this extension is to allow for the use of the QUIC Bit by later
extensions.

Extensions to QUIC that define semantics for the QUIC Bit can be negotiated at
the same time as the grease_quic_bit transport parameter.  In this case, a
recipient needs to be able to distinguish a randomized value from a value
carrying information according to the extension.  Extensions that use the QUIC
Bit MUST negotiate their use prior to acting on any semantic.

For example, an extension might define a transport parameter that is sent in
addition to the grease_quic_bit transport parameter.  Though the value of the
QUIC Bit in packets received by a peer might be set according to rules defined
by the extension, they might also be randomized as specified in this document.

Receiving a transport parameter for an extension that uses the QUIC Bit could be
used to confirm that a peer supports the semantic defined in the extension.  To
avoid acting on a randomized signal, the extension can require that endpoints
set the QUIC Bit according to the rules of the extension, but defer acting on
the information conveyed until the transport parameter for the extension is
received.

Extensions that define semantics for the QUIC Bit can be negotiated without
using the grease_quic_bit transport parameter.  However, including both
extensions allows for the QUIC Bit to be greased even if the alternative use is
not supported.


# Security Considerations

This document introduces no new security considerations for endpoints or
entities that can rely on endpoint cooperation.  However, this change makes the
task of identifying QUIC more difficult without cooperation of endpoints.  This
sometimes works counter to the security goals of network operators who rely on
network classification to identify threats; see
{{?MANAGEABILITY=I-D.ietf-quic-manageability}} for a more comprehensive
treatment of this topic.


# IANA Considerations

This document registers the grease_quic_bit transport parameter in the "QUIC
Transport Parameters" registry established in {{Section 22.3 of QUIC}}.  The
following fields are registered:

Value:
: 0x2ab2

Parameter Name:
: grease_quic_bit

Status:
: Permanent

Specification:
: This document.

Date:
: Date of registration.

Contact:
: QUIC Working Group (quic@ietf.org)

Change Controller:
: IETF (iesg@ietf.org)

Notes:
: (none)
