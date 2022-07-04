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

# Definition and Terminology

An oblivious system separates information that can be used to identify a user from other information that is needed to operate
a services in order to protect the user's privacy. In order to still enable a service to verify that
a user is authorized to use a service, a separate trusted entity that knows and attests the user's identity
is introduced into the architecture.

# Archictural Models using Partitioning

In order to achieve privacy partitioning, more entities than the user (data owner) and service provider need
to be involved in the communication architecture. Usually at least on entity in introduced that knows the user's identity and can
attest certain properties or rights that are connected with that identity (attester). However, this entity should not require
any knowledge about the service used or related user actions that could reveal privacy sensitive data about the user or user behavior.
Therefore another entities might be needed that has knowledge about the service that is requested by the user and its requirements but relies
on an anonymous identify provided by the attester (???).

The following section discusson currently on-going work in the IETF and how the privacy partitioning princplies is applied.

## OHAI

Oblivious HTTP introduces a Proxy Resource and a Request Resource. The Proxy Resource can identify the user based on its IP address, however, it can not the user request that is only decapsulated by the Request Resource which in turn only sees the IP address of the proxy.  

## ODoH

Oblivious DoH applies the same principle as Oblivious HTTP to DNS...

## MASQUE 

MASQUE provide a HTTP3 based generic proxying framework similar as many VPN services today. Such a proxy can be used reveal user identity/IP address from the server. However, the proxy itself still has full visibility. Only if the MASQUE framework is used with at least two encapsulated proxy, identify and user actions can be decouples for all entities involved.

## PrivacyPass

The privacypass protocol connect to and attester that forwards a request to a token issuer. This token can then be used by the client to accesss services anonymously. 

## PPM/PRIO

PPM doesn't necessarily separate identity from other user data, however, enables anonymous data collection (without the need of knowledge of identity) and ensure anonymity by only providing partial data to each so-called leader while still enabling analysis of aggregated data by the collector. 

# Impacts of Partitioning

Applying privacy partitioning to communication protocols lead to a substantial change in communication patterns. 
Instead of sending traffic directly to a service, essentially all user traffic is routed through a set of intermediaries.
Information has has been observed passively in the network or metadata that has been unintentionally revealed to the service provider
cannot be used anymore for e.g. existing security procedures such as access rate limiting or DDoS mitigation.

However, network management techniques deployed at present often rely on information that is exposed by most traffic but without any guarantees that the information is accurate. Privacy partitioning provides an opportunity for improvements in these management techniques by providing opportunities to actively exchange information with each entity in a privacy-preserving way and requesting a providing exactly the information needed for a specific task or function rather then relying on assumption that derived on a limited set of unintentionally revealed information which ca not be guaranteed to bee present and may disappear any time in future.

# Security Considerations

In these models, only the client knows the fully joined set of information. No server individually knows how to join all the data as long as servers do not collude the data. Therefore, for privacy partitioning to be applicable there must always be a non-collusion assumption between all involved entities.

Clients needs to be able to be explicit about what data they share (location, etc.) with which entity. All information shared need to be analysed based on its privacy-sensitivity and in relation to the entities/service it is shared with. In order to do this, clients need to be able to explicitly select different entities and hops over which to partition data. As a result, these entities need to actively/more explicitly collude in order to rejoin data but the client still needs to trust these entities to not collude.

TLS and ECH are good examples of using encryption to hide information from most parties, but in that case clients and servers (two entities) still can join all informations. VPNs hide identity from the server but the proxy has still the full information. Therefore privacy partitioning requires at least two additional entities to avoid revealing both (identity (who) and user actions (what)) from each involved party.

Timing side-channels, etc, are still possible. They can of course be mitigated in very expensive ways by e.g. fuzzing the data, but it's important to point out that this general architecture doesn't prevent these attacks. It does, however, move the threat model to this kind of analysis rather than just looking at direct metadata.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
