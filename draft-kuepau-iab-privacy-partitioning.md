---
title: "Partitioning as an Architecture for Privacy"
abbrev: "Partitioning for Privacy"
category: info

docname: draft-kuepau-iab-privacy-partitioning-latest
v: 3
keyword: Internet-Draft

author:
 -
    fullname: Mirja Kühlewind
    organization: Ericsson Research
    email: mirja.kühlewind@ericsson.com
 -
    fullname: Tommy Pauly
    organization: Apple
    email: tpauly@apple.com

normative:

informative:


--- abstract

There is a growing trend in Internet protocols to partition data and communication across
multiple parties as a means to improve the privacy of user identity, data, and metadata.
This document describes emerging patterns in protocols to partition what information is
revealed through protocol interactions, provides common terminology, and discusses how
to analyze such models.

--- middle

# Introduction

The focus of many common security protocols, such as TLS or IPsec, is to prevent data
from being modified or seen by parties other than the protocol participants. Encrypting
and authenticating communication (in HTTP, in DNS, and more) provides user privacy
benefits by ensuring that information about user identity and activity cannot be
read by passively observing traffic.

More recently, however, another aspect of privacy has come into focus: preventing
protocol participants from being exposed to unnecessary data or metadata. Some examples
of this include:

- A user accessing a service on a website might not want to reveal their location,
but if that service is able to observe the client's IP address, it can infer many
location details.

- A user might want to be able to access content for which they are authorized,
such as a news article, without needing to have which specific articles they
read on their account being recorded.

- A client device that needs to upload metrics to an aggregated
service might want to be able to contribute into the aggregated metrics system without
having specific activities being tracked and identified.

These are all problems that involve needing to separate out which information
is seen at different steps of protocol interaction, or between different
participants in a protocol.

Several working groups in the IETF are working on solutions in this space, including
OHAI, MASQUE, Privacy Pass, and PPM. One commonality between these is that they
usually involve at least three parties: a client and at least two parties acting
as server or intermediaries. While at first glance, the involvement of more parties
can seem counterintuitive for providing privacy, the ability to separate what data
is shared between these parties limits the amount of information about a single
client that can be concentrated by a server.

These works mainly focus one communication protocol support for privacy partitioning.
A remaining challenging also lives in identity management itself. Today, mainly services
implement their own identify management (requiring separate account of each service) or
rely on a small number of large identify providers. The use of a third party identity
provider would allow for privacy partitioning by having the identify provider attesting service
access rights or other relevant user characteristics in an anonymous way. However, today use of
the centralized system rather lead to an even larger centralization of data as user activity is
not sufficiently revealed from the identify management system, and identity is not anonymized by
the management system.

# Definition and Terminology

An oblivious system separates information that can be used to identify a user from other information that is needed to operate
a services in order to protect the user's privacy. In order to still enable a service to verify that
a user is authorized to use a service, a separate trusted entities that knows and attests the user's identity
is introduced into the architecture.

# Archictural Models that use Partitioning

# Impacts of Partitioning

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
