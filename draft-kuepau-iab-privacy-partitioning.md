---
title: "Partitioning as an Architecture for Privacy"
abbrev: "Partitioning for Privacy"
category: info
stream: IAB

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

This document describes the principle of privacy partitioning, which selectively spreads data and communication across
multiple parties as a means to improve the privacy by separating user identity from user data.
This document describes emerging patterns in protocols to partition what data and metadata is
revealed through protocol interactions, provides common terminology, and discusses how
to analyze such models.

--- middle

# Introduction

Protocols such as TLS and IPsec provide a secure (authenticated and encrypted) channel
between two endpoints over which endpoints transfer information. Encryption and authentication
of data in transit is necessary to protect information from being seen or modified by parties
other than the intended protocol participants. As such, this kind of security is necessary for ensuring that
information transferred over these channels remain private.

However, a secure channel between two endpoints is insufficient for privacy of the endpoints
themselves. In recent years, privacy requirements have expanded beyond the need to protect data in transit
between two endpoints. Some examples of this expansion include:

- A user accessing a service on a website might not consent to reveal their location,
but if that service is able to observe the client's IP address, it can learn inforamtion
about the user's location. This is problematic for privacy since the service can link
user data to the user's location.

- A user might want to be able to access content for which they are authorized,
such as a news article, without needing to have which specific articles they
read on their account being recorded. This is problematic for privacy since the service
can link user activity to the user's account.

- A client device that needs to upload metrics to an aggregation service might want to be
able to contribute data to the system without having their specific contributions being
attribued to them. This is problematic for privacy since the service can link client
contributions to the specific client.

The commonality in these examples is that clients want to interact with or use a service
without exposing too much user-specific or identifying information to that service. In particular,
separating the user-specific identity information from user-specific data is necessary for
privacy. Thus, order to protect user privacy, it is important to keep identity (who) and data
(what) separate.

This document defines "privacy partitioning" as the general technique used to separate the data
and metadata visible to various parties in network communication, with the aim of improving
user privacy. Partitioning is a spectrum and not a panacea. It is difficult to guarantee there
is no link between user-specific identity and user-specific data. However, applied properly,
privacy partitioning helps ensure that user privacy violations becomes more technically difficult
to achieve over time.

Several IETF working groups are working on protocols or systems that adhere to the principle
of privacy partitioning, including OHAI, MASQUE, Privacy Pass, and PPM. This document summarizes
work in those groups and describes a framework for reasoning about the resulting privacy posture of different
endpoints in practice.

# Privacy Partitioning

For the purposes of user privacy, this document focuses on user-specific information. This
might include any identifying information that is specific to a user, such as their email
address or IP address, or data about the user, such as their date of birth. Informally,
the goal of privacy partitioning is to ensure that each party in a system beyond the user
themselves only has access to one type of user-specific information.

This is a simple application of the principle of least privilege, wherein every party in
a system only has access to the minimum amount of information needed to fulfill their
function. Privacy partitioning advocates for this minimization by ensuring that protocols,
applications, and systems only reveal user-specific information to parties that need access
to the  information for their intended purpose.

Put simply, privacy partitioning aims to separate *who* someone is from *what* they do. In the
rest of this section, we describe how the privacy partitioning can be used to achieve this goal.

## Privacy Contexts

Each piece of user-specific information exists within some context, where a context
is abstractly defined as a set of data and metadata and the entities that share access
to that metadata. In order to prevent correlation of user-specific information across
contexts, partitions need to ensure that any single entity (other than the client itself)
does not participate in more than one context where the information is visible.

{{?RFC6973}} discusses the importance of identifiers in reducing correlation:

"Correlation is the combination of various pieces of information related to an individual
or that obtain that characteristic when combined... Correlation is closely related to
identification.  Internet protocols can facilitate correlation by allowing individuals'
activities to be tracked and combined over time."

Context separation is foundational to privacy partitioning and reducing correlation.
As an example, consider an unencrypted HTTP session over TCP, wherein the context includes both the
content of the transaction as well as any metadata from the transport and IP headers; and the
participants include the client, routers, other network middleboxes, intermediaries, and server.

~~~ aasvg
+-------------------------------------------------------------------+
| Context A                                                         |
|  +--------+                +-----------+              +--------+  |
|  |        |------HTTP------|           |--------------|        |  |
|  | Client |                | Middlebox |              | Server |  |
|  |        |------TCP-------|           |--------------|        |  |
|  +--------+      flow      +-----------+              +--------+  |
|                                                                   |
+-------------------------------------------------------------------+
~~~
{: #diagram-middlebox title="Diagram of a basic unencrypted client-to-server connection with middleboxes"}

Adding TLS encryption to the HTTP session is a simple partitioning technique that splits the
previous context into two separate contexts: the content of the transaction is now only visible
to the client, TLS-terminating intermediaries, and server; while the metadata in transport and
IP headers remain in the original context. In this scenario, without any further partitioning,
the entities that participate in both contexts can allow the data in both contexts to be correlated.

~~~ aasvg
+-------------------------------------------------------------------+
| Context A                                                         |
|  +--------+                                           +--------+  |
|  |        |                                           |        |  |
|  | Client |-------------------HTTPS-------------------| Server |  |
|  |        |                                           |        |  |
|  +--------+                                           +--------+  |
|                                                                   |
+-------------------------------------------------------------------+
| Context B                                                         |
|  +--------+                +-----------+              +--------+  |
|  |        |                |           |              |        |  |
|  | Client |-------TCP------| Middlebox |--------------| Server |  |
|  |        |       flow     |           |              |        |  |
|  +--------+                +-----------+              +--------+  |
|                                                                   |
+-------------------------------------------------------------------+
~~~
{: #diagram-https title="Diagram of how adding encryption splits the context into two"}

Another way to create a partition is to simply use separate connections. For example, to
split two separate HTTP requests from one another, a client could issue the requests on
separate TCP connections, each on a different network, and at different times; and avoid
including obvious identifiers like HTTP cookies across the requests.

~~~ aasvg
+-------------------------------------------------------------------+
| Context A                                                         |
|  +--------+                +-----------+              +--------+  |
|  |        | IP A           |           |              |        |  |
|  | Client |-------TCP------| Middlebox |--------------| Server |  |
|  |        |       flow A   |     A     |              |        |  |
|  +--------+                +-----------+              +--------+  |
|                                                                   |
+-------------------------------------------------------------------+
| Context B                                                         |
|  +--------+                +-----------+              +--------+  |
|  |        | IP B           |           |              |        |  |
|  | Client |-------TCP------| Middlebox |--------------| Server |  |
|  |        |       flow B   |     B     |              |        |  |
|  +--------+                +-----------+              +--------+  |
|                                                                   |
+-------------------------------------------------------------------+
~~~
{: #diagram-dualconnect title="Diagram of making separate connections to generate separate contexts"}

## Context Separation

In order to define an analyze how various partitioning techniques work, the boundaries of what is
being partitioned need to be established. This is the role of context separation. In particular,
in order to prevent correlation of user-specific information across contexts, partitions need
to ensure that any single entity (other than the client itself) does not participate in contexts
where both identities are visible.

Context separation can be achieved in different ways, e.g. over time, across network paths, based
on (en)coding, etc. The privacy-oriented protocols described in this document generally involve
more complex partitioning, but the techniques to partition communication contexts still employ the
same techniques:

1. Encryption allows partitioning of contexts within a given network path.
1. Using separate connections across time or space allow partitioning of contexts for different
application transactions.

These techniques are frequently used in conjunction for context separation. For example,
encrypting an HTTP exchange might prevent a network middlebox that sees a client IP address
from seeing the user account identity, but it doesn't prevent the TLS-terminating server
from observing both identities and correlating them. As such, preventing correlation
requires separating contexts, such as by using proxying to conceal a client IP address
that would otherwise be used as an identifier.

## Mitigating Collusion

Partitioning aims to ensure that no single entity can gather information beyond the context
that a user or client intends. It can make trivial correlation of data and identities difficult for
a single entity. However, designing protocols to use partitioning cannot (alone) prevent
tracking if entities across the different contexts collude and share data. Thus, partitioning is not a
panacea, but rather a tool, and a necessary condition for meaningful privacy improvements.

Other techniques that can be used to mitigate collusion include:

- Policy and contractual agreements between entities involved in partitioning, to disallow
logging or sharing of data, or to require auditing.

- Protocol requirements to make collusion or data sharing more difficult.

- Adding more partitions and contexts, to make it increasingly difficult to collude with
enough parties to recover identities.

# A Survey of Protocols using Partitioning

The following section discusses currently on-going work in the IETF
that is applying privacy partitioning.

## CONNECT Proxying and MASQUE

HTTP "forward proxies", when using encryption, provide privacy partitioning by separating
a connection into multiple segments. When connections over the proxy themselves are encrypted,
the proxy cannot see the end-to-end content. HTTP has historically supported forward proxying
for TCP-like streams via the CONNECT method. More recently, the MASQUE working group has developed
protocols to similarly proxy UDP {{?CONNECT-UDP=RFC9297}} and IP packets
{{?CONNECT-IP=I-D.ietf-masque-connect-ip}} based on tunneling.

Use of a single proxy partitions communication into a Client-to-Proxy context (the transport
metadata between the client and the proxy, and the request to the proxy to open a connection
to the target), a Proxy-to-Target context (the transport metadata between the proxy and target),
and a Client-to-Target context (the end-to-end data, which generally would be a TLS-encrypted
connection). However, this alone does not achieve the goals of privacy partitioning fully,
since the proxy is in a privileged position where it participates in a context that sees
the client identity and the target information. If the target is sufficiently generic, this
scenario is acceptable, but if the act of communicating with the target is sensitive, then
the proxy can learn information about the client.

~~~ aasvg
+-------------------------------------------------------------------+
| Client-to-Target Context                                          |
|  +--------+                +-----------+              +--------+  |
|  |        |                |           |              |        |  |
|  | Client |----Proxied-----|   Proxy   |--------------| Server |  |
|  |        |      flow      |           |              |        |  |
|  +--------+                +-----------+              +--------+  |
|                                                                   |
+-------------------------------------------------------------------+
| Client-to-Proxy Context                                           |
|  +--------+                +-----------+                          |
|  |        |                |           |                          |
|  | Client |---Transport----|   Proxy   |                          |
|  |        |     flow       |           |                          |
|  +--------+                +-----------+                          |
|                                                                   |
+-------------------------------------------------------------------+
| Proxy-to-Target Context                                           |
|                            +-----------+              +--------+  |
|                            |           |              |        |  |
|                            |   Proxy   |--Transport---| Server |  |
|                            |           |    flow      |        |  |
|                            +-----------+              +--------+  |
|                                                                   |
+-------------------------------------------------------------------+
~~~
{: #diagram-1hop title="Diagram of one-hop proxy contexts"}


Using two or more proxies provides better privacy partitioning. Now, each proxy sees the
Client metadata, but not the Target; the Target, but not the Client metadata; or neither.

~~~ aasvg
+-------------------------------------------------------------------+
| Client-to-Target Context                                          |
|  +--------+                           +-------+       +--------+  |
|  |        |                           |       |       |        |  |
|  | Client |----------Proxied----------| Proxy |-------| Server |  |
|  |        |           flow            |   B   |       |        |  |
|  +--------+                           +-------+       +--------+  |
|                                                                   |
+-------------------------------------------------------------------+
| Client-to-Proxy B Context                                         |
|  +--------+         +-------+         +-------+                   |
|  |        |         |       |         |       |                   |
|  | Client |---------| Proxy |---------| Proxy |                   |
|  |        |         |   A   |         |   B   |                   |
|  +--------+         +-------+         +-------+                   |
|                                                                   |
+-------------------------------------------------------------------+
| Client-to-Proxy A Context                                         |
|  +--------+         +-------+                                     |
|  |        |         |       |                                     |
|  | Client |---------| Proxy |                                     |
|  |        |         |   A   |                                     |
|  +--------+         +-------+                                     |
|                                                                   |
+-------------------------------------------------------------------+
| Proxy A-to-Proxy B Context                                        |
|                     +-------+         +-------+                   |
|                     |       |         |       |                   |
|                     | Proxy |---------| Proxy |                   |
|                     |   A   |         |   B   |                   |
|                     +-------+         +-------+                   |
|                                                                   |
+-------------------------------------------------------------------+
| Proxy B-to-Target Context                                         |
|                                       +-------+       +--------+  |
|                                       |       |       |        |  |
|                                       | Proxy |-------| Server |  |
|                                       |   B   |       |        |  |
|                                       +-------+       +--------+  |
|                                                                   |
+-------------------------------------------------------------------+
~~~
{: #diagram-2hop title="Diagram of two-hop proxy contexts"}

Forward proxying, such as the protocols developed in MASQUE, uses both encryption (via TLS) and
separation of connections (proxy hops) to achieve privacy partitioning.

## Oblivious HTTP and DNS

Oblivious HTTP {{?OHTTP=I-D.ietf-ohai-ohttp}}, developed in the OHAI working group, adds per-message
encryption to HTTP exchanges through a relay system. Clients send requests through an Oblivious Relay,
which cannot read message contents, to an Oblivious Gateway, which can decrypt the messages but
cannot communicate directly with the client or observe client metadata like IP address.
Oblivious HTTP relies on Hybrid Public Key Encryption {{?HPKE=RFC9180}} to perform encryption.

Oblivious HTTP uses both encryption and separation of connections to achieve privacy partitioning.
The end-to-end messages are encrypted between the Client and Gateway (forming a Client-to-Gateway context),
and the connections are separated into a Client-to-Relay context and a Relay-to-Gateway context. It is also important
to note that the Relay-to-Gateway connection can be a single connection, even if the Relay has many
separate Clients. This provides better anonymity by making the pseudonym presented by the Relay to
be shared across many Clients.

~~~ aasvg
+-------------------------------------------------------------------+
| Client-to-Target Context                                          |
|  +--------+                           +---------+     +--------+  |
|  |        |                           |         |     |        |  |
|  | Client |---------------------------| Gateway |-----| Target |  |
|  |        |                           |         |     |        |  |
|  +--------+                           +---------+     +--------+  |
|                                                                   |
+-------------------------------------------------------------------+
| Client-to-Gateway Context                                         |
|  +--------+         +-------+         +---------+                 |
|  |        |         |       |         |         |                 |
|  | Client |---------| Relay |---------| Gateway |                 |
|  |        |         |       |         |         |                 |
|  +--------+         +-------+         +---------+                 |
|                                                                   |
+-------------------------------------------------------------------+
| Client-to-Relay Context                                           |
|  +--------+         +-------+                                     |
|  |        |         |       |                                     |
|  | Client |---------| Relay |                                     |
|  |        |         |       |                                     |
|  +--------+         +-------+                                     |
|                                                                   |
+-------------------------------------------------------------------+
~~~
{: #diagram-ohttp title="Diagram of Oblivious HTTP contexts"}

Oblivious DNS over HTTPS {{?ODOH=RFC9230}} applies the same principle as Oblivious HTTP, but operates on
DNS messages only. As a precursor to the more generalized Oblivious HTTP, it relies on the same
HPKE cryptographic primatives, and can be analyzed in the same way.

## Privacy Pass

Privacy Pass is an architecture {{?PRIVACYPASS=I-D.ietf-privacypass-architecture}} and set of protocols
being developed in the Privacy Pass working group that allow clients to present proof of verification in
an anonymous and unlinkable fashion, via tokens. These tokens originally were designed as a way to prove
that a client had solved a CAPTCHA, but can be applied to other fraud prevention and attestation
scenarios as well.

In Privacy Pass, privacy partitioning is achieved by encryption (creating tokens that are cryptographically
unlinkable) and separation of connections across two contexts: a "redemption context" between clients an origins
(servers that request and receive tokens), and an "issuance context" between clients, attestation servers, and
token issuance servers.

~~~ aasvg
+-------------------------------------------------------------------+
| Redemption Context                                                |
|  +--------+         +--------+                                    |
|  |        |         |        |                                    |
|  | Origin |---------| Client |                                    |
|  |        |         |        |                                    |
|  +--------+         +--------+                                    |
|                                                                   |
+-------------------------------------------------------------------+
| Issuance Context                                                  |
|                     +--------+      +----------+      +--------+  |
|                     |        |      |          |      |        |  |
|                     | Client |------| Attester |------| Issuer |  |
|                     |        |      |          |      |        |  |
|                     +--------+      +----------+      +--------+  |
|                                                                   |
+-------------------------------------------------------------------+
~~~
{: #diagram-privacypass title="Diagram of contexts in Privacy Pass"}

## DAP (PPM)

Distributed Aggregation Protocol (DAP) is a protocol being developed in the PPM working group to allow
servers to collect aggregate metrics across a large population of clients, without learning individual
data points about clients.

TODO: Analysis of contexts

TODO: Diagram for DAP

# Analysis of Privacy Partioning

TODO: Provide a framework for analyzing the privacy properties of partitioned contexts

TODO: Discuss how privacy breaks down when assumptions are violated

# Impacts of Partitioning

Applying privacy partitioning to communication protocols lead to a substantial change in communication patterns.
Instead of sending traffic directly to a service, essentially all user traffic is routed through a set of intermediaries.
Information has has been observed passively in the network or metadata that has been unintentionally revealed to the service provider
cannot be used anymore for e.g. existing security procedures such as access rate limiting or DDoS mitigation.

However, network management techniques deployed at present often rely on information that is exposed by
most traffic but without any guarantees that the information is accurate. Privacy partitioning provides
an opportunity for improvements in these management techniques by providing opportunities to actively
exchange information with each entity in a privacy-preserving way and requesting exactly the information
needed for a specific task or function rather then relying on assumption that are derived on a limited
set of unintentionally revealed information which cannot be guaranteed to be present and may disappear
any time in future.

# Security Considerations

In these models, only the client knows the fully joined set of information. No server individually
knows how to join all the data as long as servers do not collude the data. Therefore, for privacy
partitioning to be applicable there must always be a non-collusion assumption between all involved entities.

Clients needs to be able to be explicit about what data they share (location, etc.) with which entity.
All information shared need to be analysed based on its privacy-sensitivity and in relation to the
entities/service it is shared with. In order to do this, clients need to be able to explicitly select
different entities and hops over which to partition data. As a result, the client still needs to trust
these entities to not collude but these entities also need to actively/more explicitly collude in order
to rejoin data.

TLS and Encrypted Client Hello (ECH) are good examples of using encryption to hide information from some entities,
but where clients and servers (two entities) still can correlate all information. VPNs hide identity from end
servers, but the VPN server has still can see the identity of both the client and server. Therefore, privacy
partitioning requires at least two additional entities to avoid revealing both (identity (who) and user actions
(what)) from each involved party.

Timing side-channels, etc, are still possible. They can of course be mitigated in very expensive ways by, e.g., fuzzing the data,
but it's important to point out that this general architecture doesn't prevent these attacks. It does, however, move the
threat model to this kind of analysis rather than just looking at direct metadata.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
