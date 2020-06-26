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
defined as having a fixed value of 1 in QUIC version 1 {{!QUIC}}.  This bit is
The purpose of this is to allow intermediaries and endpoints to cheaply and
efficiently distinguish between QUIC and other protocols; see {{?DEMUX=RFC7983}}
for a description of a scheme with which QUIC is designed to integrate.  As this
bit effectively identifies a packet as QUIC, it is sometimes referred to as the
"QUIC Bit".

Where endpoints and the intermediaries that support them do not depend on the
QUIC Bit having a fixed value, sending the same value in every packet is more of
liability than an asset.  If systems come to depend on a fixed value, then it
might become infeasible to define a version of QUIC that attributes semantics to
this bit.

In order to safeguard future use of this bit, this document defines a QUIC
transport parameter that indicates that an endpoint is willing to receive QUIC
packets containing any value for this bit.


# Conventions and Definitions

{::boilerplate bcp14}

This document uses terms and notational conventions from {{QUIC}}.


# The Grease QUIC Bit Transport Parameter

The grease_quic_bit transport parameter (0xTBD) can be sent by both client and
server.  The transport parameter is sent with an empty value; an endpoint that
understands the transport parameter MUST treat receipt of a non-empty value as a
connection error of type TRANSPORT_PARAMETER_ERROR.

Advertising the grease_quic_bit transport parameter indicates that packets sent
to this endpoint MAY include values other than 1 for the QUIC Bit.  The QUIC Bit
is defined as the second-to-most significant bit of the first byte of QUIC
packets (that is, the value 0x40).


## Clearing the QUIC Bit

Endpoints that receive the grease_quic_bit transport parameter from a peer MAY
set the QUIC Bit to any value in packets they sent to that peer.  Endpoints
SHOULD set the QUIC Bit to an unpredictable value.  This includes Initial and
Handshake packets sent after receiving transport parameters.

A client MAY also clear the QUIC Bit in Initial packets sent to establish a new
connection. A client can only clear the QUIC Bit if the packet includes a token
provided by the server in a NEW_TOKEN frame on a connection where the server
also included the grease_quic_bit transport parameter.  To allow for changes in
server configuration, clients SHOULD set the QUIC Bit if the token was provided
more than 7 days prior.


## Using the QUIC Bit

Other extensions to QUIC that define semantics for the QUIC Bit can also be
advertised in combination with the grease_quic_bit transport parameter.  In
order for a receipient to distinguish between a randomized value and a value
carrying information, extensions that use the QUIC Bit need a signal of mutual
support prior to acting on any semantic.

For example, an extension might define a transport parameter that is sent in
addition to the grease_quic_bit transport parameter.  Though the value of the
QUIC Bit in packets received by a peer might carry a semantic specific to that
extension, they might also be randomized according to the definition of the
grease_quic_bit extension.  Receiving the transport parameter for the extension
confirms that a peer supports the semantic defined in the extension.  Thus, to
avoid acting on a randomized signal, the extension can require that endpoints
set the QUIC Bit according to the rules of the extension, but defer acting on
the information conveyed until the transport parameter for the extension is
received.


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
: IETF QUIC Working Group (quic@ietf.org)

Notes:
: (none)


--- back

# Acknowledgments
{:numbered="false"}

TODO
