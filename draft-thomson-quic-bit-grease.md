---
title: "Greasing the QUIC Bit"
docname: draft-thomson-quic-bit-grease-latest
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

QUIC {{!QUIC=I-D.ietf-quic-transport}} intentionally describes a very narrow set
of fields that are visible to entities other than endpoints.  Beyond those
characteristics that are defined as invariant
{{?QUIC-INVARIANTS=I-D.ietf-quic-invariants}}, very little about the "wire
image" {{?RFC8546}} of QUIC is visible.

The second-to-most significant bit of the first byte in every QUIC packet is
defined as having a fixed value in QUIC version 1 {{!QUIC}}.  The purpose of
having a fixed value is to allow intermediaries and endpoints to efficiently
distinguish between QUIC and other protocols; see {{?DEMUX=RFC7983}} for a
description of a scheme that QUIC can integrate with as a result.  As this bit
effectively identifies a packet as QUIC, it is sometimes referred to as the
"QUIC Bit".

Where endpoints and the intermediaries that support them do not depend on the
QUIC Bit having a fixed value, sending the same value in every packet is more of
liability than an asset.  If systems come to depend on a fixed value, then it
might become infeasible to define a version of QUIC that attributes semantics to
this bit.

In order to safeguard future use of this bit, this document defines a QUIC
transport parameter that indicates that an endpoint is willing to receive QUIC
packets containing any value for this bit.  By sending different values for this
bit, the hope is that the value will remain available for future use
{{?USE-IT=I-D.iab-use-it-or-lose-it}}.


# Conventions and Definitions

{::boilerplate bcp14}

This document uses terms and notational conventions from {{QUIC}}.


# The Grease QUIC Bit Transport Parameter

The grease_quic_bit transport parameter (0x2ab2) can be sent by both client and
server.  The transport parameter is sent with an empty value; an endpoint that
understands this transport parameter MUST treat receipt of a non-empty value as
a connection error of type TRANSPORT_PARAMETER_ERROR.

Advertising the grease_quic_bit transport parameter indicates that packets sent
to this endpoint MAY set a value of 0 for the QUIC Bit.  The QUIC Bit is defined
as the second-to-most significant bit of the first byte of QUIC packets (that
is, the value 0x40).

A server MUST respect the value it previously provided for the grease_quic_bit
transport parameter if it accepts 0-RTT.  A client MAY forget the value.  In all
other cases, only the presence or absence of the transport parameter in the
current handshake is used to determine what values can be sent in the QUIC Bit.


## Clearing the QUIC Bit

Endpoints that receive the grease_quic_bit transport parameter from a peer MAY
set the QUIC Bit to any value in packets they sent to that peer.  Endpoints
SHOULD set the QUIC Bit to an unpredictable value unless another extension
assigns specific meaning to the value of the bit.  All packets sent after
receiving and processing transport parameters are affected, including Retry,
Initial, and Handshake packets.

A client MAY also clear the QUIC Bit in Initial packets that are sent to
establish a new connection. A client can only clear the QUIC Bit if the packet
includes a token provided by the server in a NEW_TOKEN frame on a connection
where the server also included the grease_quic_bit transport parameter.  To
allow for changes in server configuration, clients SHOULD set the QUIC Bit if
the token was provided more than 7 days prior.


## Using the QUIC Bit

The purpose of this extension is to allow for the use of the QUIC Bit by later
extensions.

Extensions to QUIC that define semantics for the QUIC Bit can be negotiated at
the same time as the grease_quic_bit transport parameter.  In this case, a
recipient needs to be able to distinguish a randomized value from a value
carrying information according to the extension.  Extensions that use the QUIC
Bit MUST negotiate their use prior to acting on any semantic.  Endpoints MAY
send a signal prior to this negotiation completing, but any value carried by the
bit cannot be used until it is clear that the peer is using the extension.

For example, an extension might define a transport parameter that is sent in
addition to the grease_quic_bit transport parameter.  Though the value of the
QUIC Bit in packets received by a peer might be set according to rules defined
by the extension, they might also be randomized according to the definition of
the grease_quic_bit extension.  Receiving the transport parameter for the
extension could be used to confirm that a peer supports the semantic defined in
the extension.  To avoid acting on a randomized signal, the extension can
require that endpoints set the QUIC Bit according to the rules of the extension,
but defer acting on the information conveyed until the transport parameter for
the extension is received.

Extensions that define semantics for the QUIC Bit can be negotiated without
using the grease_quic_bit transport parameter.


# Security Considerations

This document introduces no new security considerations for endpoints or
entities that can rely on endpoint cooperation.  However, this change makes the
task of identifying QUIC more difficult without cooperation of endpoints.  This
sometimes works counter to the security goals of network operators who rely on
network classification to identify threats.


# IANA Considerations

This document registers the grease_quic_bit transport parameter in the "QUIC
Transport Parameters" registry established in Section 22.2 of {{QUIC}}.  The
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


--- back

# Acknowledgments
{:numbered="false"}

TODO
