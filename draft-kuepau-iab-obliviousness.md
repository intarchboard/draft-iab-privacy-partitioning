---
title: "The Archiectural Principle of Obliviousness"
abbrev: "Obliviousness"
category: info

docname: draft-kuepau-iab-obliviousness-latest
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

This document defines the archiectural principle of obliviousness. In short an oblivious system
separates information that can be used to identify a user from other information that is needed to operate
a services in order to protect the user's privacy. In order to still enable a service to verify that
a user is authorized to use a service, a separate trusted entities that knows and attests the user's identifity
is introduced into the architecture.


--- middle

# Introduction

Recently various techniques to strengthen user security and privacy have been developed and deployed. 
Most techniques aim to protect communication from access and manipulation of unintended third parties, such as the wide-scale deployment of HTTPS to encrypt web content as well as deployment efforts for associated data like DNS requests. 
More recently various mechanisms to enhance user privacy during any such communications are in focus. 
These mechanism often rely on a relay services that intermediates the communication between two hosts.
While on a first glance it seems counterintuitive to involve yet another party in the communication in order to protect user privacy, the idea about these relays is that at the end each party can only access a limited set of information and therefore each parties knows less than before.


# Definition of Obliviousness

An oblivious system separates information that can be used to identify a user from other information that is needed to operate
a services in order to protect the user's privacy. In order to still enable a service to verify that
a user is authorized to use a service, a separate trusted entities that knows and attests the user's identifity
is introduced into the architecture.

# Impact of Obliviousness


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
