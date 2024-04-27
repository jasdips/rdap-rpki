%%%
Title = "An RDAP Extension for RPKI Data"
area = "Applications and Real-Time Area (ART)"
workgroup = "Registration Protocols Extensions (regext)"
abbrev = "rdap-rpki"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-jasdips-regext-rdap-rpki-00"
stream = "IETF"
status = "standard"
date = 2024-04-27T00:00:00Z

[[author]]
initials="J."
surname="Singh"
fullname="Jasdip Singh"
organization="ARIN"
[author.address]
email = "jasdips@arin.net"

%%%

.# Abstract

{mainmatter}

# Introduction

The network operators are increasingly deploying the Resource Public Key Infrastructure (RPKI, [@!RFC6480]) to secure
inter-domain routing ([@RFC4271]) on the internet. RPKI enables internet number resource holders to cryptographically
assert about their registered IP addresses and autonomous system numbers to help prevent route hijacks and leaks. To
that end, RPKI defines the following cryptographic profiles:
* Route Origin Authorization (ROA, [@!RFC6482] and [@?I-D.ietf-sidrops-rfc6482bis] (obsoletes [@!RFC6482])) where an IP
  prefix holder cryptographically asserts about the origin autonomous system (AS) for routing that IP prefix.
* Autonomous System Provider Authorization (ASPA, [@?I-D.ietf-sidrops-aspa-profile]) when an autonomous system number
  (ASN) holder cryptographically asserts about the provider AS for that ASN.

This extension maps the registration data in the Regional Internet Registries (RIRs), including at national and local
levels, for aforementioned RPKI profiles into RDAP. The intent is that such RDAP data can complement the existing RPKI
diagnostic tools when troubleshooting a route hijack or leak, by conveniently providing access to registration
information from an RIR's database beside what's inherently available from an RPKI profile object. There is metadata
that is often needed for troubleshooting that does not appear in, say, a ROA or a VRP (Verified ROA Payload); such as:
* Is it an auto-renewing ROA?
* And if so, when did the first version get published?
* And, was it created in conjunction with an Internet Routing Registry (IRR, [@RFC2622]) route object?

This specification next defines RDAP object classes, as well as lookup and search path segments, for ROA and ASPA
registration data.

## Requirements Language

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP
14 [@!RFC2119] [@!RFC8174] when, and only when, they appear in all
capitals, as shown here.

Indentation and whitespace in examples are provided only to illustrate
element relationships, and are not a REQUIRED feature of this
protocol.

"..." in examples is used as shorthand for elements defined outside of
this document.

# Route Origin Authorization Object Class

## Lookup

## Search

## Reverse Search

## Relationship with IP Network Object Class

# Autonomous System Provider Authorization Object Class

## Lookup

## Search

## Reverse Search

## Relationship with Autonomous System Number Object Class

# RDAP Conformance

# Security Considerations

# IANA Considerations

## RDAP Extensions Registry

## RDAP Reverse Search Registry

## RDAP Reverse Search Mapping Registry

# Acknowledgements

{backmatter}
