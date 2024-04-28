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

TODO

{mainmatter}

# Introduction

The network operators are increasingly deploying the Resource Public Key Infrastructure (RPKI, [@!RFC6480]) to secure
inter-domain routing ([@RFC4271]) on the internet. RPKI enables internet number resource holders to cryptographically
assert about their registered IP addresses and autonomous system numbers to help prevent route hijacks and leaks. To
that end, RPKI defines the following cryptographic profiles:

* Route Origin Authorization (ROA, [@!RFC6482] and [@!I-D.ietf-sidrops-rfc6482bis] (obsoletes [@!RFC6482])) where a
  Classless Inter-Domain Routing (CIDR, [@!RFC1519]) address block holder cryptographically asserts about the origin
  autonomous system (AS, [@RFC4271]) for routing that CIDR address block.
* Autonomous System Provider Authorization (ASPA, [@!I-D.ietf-sidrops-aspa-profile]) where an autonomous system number
  (ASN, [@!RFC5396]) holder cryptographically asserts about the provider AS for that ASN.

This RDAP extension maps the registration data from the Regional Internet Registries (RIRs), including at national and
local levels, for aforementioned RPKI profiles into RDAP. The intent is that such RDAP data can complement the existing
RPKI diagnostic tools when troubleshooting a route hijack or leak, by conveniently providing access to registration
information from an RIR's database beside what's inherently available from an RPKI profile object. There is metadata
that is often needed for troubleshooting that does not appear in, say, a ROA or a VRP (Verified ROA Payload); such as:

* Is it an auto-renewing ROA?
* And if so, when did the first version get published?
* And, was it created in conjunction with an Internet Routing Registry (IRR, [@RFC2622]) route object?

This specification next defines RDAP object classes, as well as lookup and search path segments, for the ROA and ASPA
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

The Route Origin Authorization (ROA) object class can contain the following members:

* objectClassName -- the string "rpki roa"
* handle -- a string representing the RIR-unique identifier of the ROA registration
* name -- a string representing an identifier assigned to the ROA registration by the registration holder
* cidr -- a string representing the CIDR address block (CIDR prefix/CIDR length) of the ROA, either IPv4 or IPv6
* startAddress -- a string representing the starting IP address (a.k.a. CIDR prefix) of the CIDR address block, either
  IPv4 or IPv6
* endAddress -- a string representing the ending IP address of the CIDR address block, either IPv4 or IPv6
* prefixLength -- a number representing the prefix length (a.k.a. CIDR length) of the CIDR address block; up to 32 for
  IPv4 and up to 128 for IPv6
* ipVersion -- a string signifying the IP protocol version of the ROA: "v4" signifies an IPv4 ROA, and "v6" signifies
  an IPv6 ROA
* maxLength -- a number representing the maximum prefix length of the CIDR address block that the origin AS is
  authorized to advertise; up to 32 for IPv4 and up to 128 for IPv6
* originAutnum -- an unsigned 32-bit integer representing the origin autonomous system number [@!RFC5396]
* autoRenewed -- a boolean indicating if the ROA is auto-renewed or not
* status -- a string indicating the validation state of the ROA
* events -- events ([@!RFC9083, section 4.5]) representing the not-valid-before and not-valid-after dates of the
  end-entity certificate for the ROA
* links -- links ([@!RFC9083, section 4.2]) for "self", and "related" to IP network and IRR objects
* remarks -- see [@!RFC9083, section 4.3]

Here is an elided example of a ROA object in RDAP:

```
{
  "objectClassName": "rpki roa",
  "handle": "XXXX",
  "name": "ROA-1",
  "cidr": "2001:db8::/48",
  "startAddress": "2001:db8::",
  "endAddress": "2001:db8:0:ffff:ffff:ffff:ffff:ffff",
  "prefixLength": 48,
  "ipVersion": "v6",
  "maxLength": 64,
  "originAutnum": 65536,
  "autoRenewed": true,
  "status": [ "valid" ],
  "events":
  [
    {
      "eventAction": "not valid before",
      "eventDate": "2024-04-27T23:59:59Z"
    },
    {
      "eventAction": "not valid after",
      "eventDate": "2025-04-27T23:59:59Z"
    },
    ...
  ],
  "links":
  [
    {
      "value": "https://example.net/rdap/rpkiRoa/XXXX",
      "rel": "self",
      "href": "https://example.net/rdap/rpkiRoa/XXXX",
      "type": "application/rdap+json"
    },
    {
      "value": "https://example.net/rdap/rpkiRoa/XXXX",
      "rel": "related",
      "href": "https://example.net/rdap/ip/2001:db8::/48",
      "type": "application/rdap+json"
    },
    ...
  ],
  "remarks":
  [
    {
      "description": [ "A ROA object in RDAP" ]
    }
  ],
}
```

## Lookup

The resource type path segment for exact match lookup of a ROA object is "rpkiRoa".

The following lookup path segment is defined for a ROA object:

Syntax: rpkiRoa/<handle>

For example:

```
https://example.net/rdap/rpkiRoa/XXXX
```

## Search

The resource type path segment for searching ROA objects is "rpkiRoas".

The following search path segments are defined for ROA objects:

Syntax: rpkiRoas?name=<name search pattern>

Syntax: rpkiRoas?cidr=<CIDR prefix/CIDR length>

Syntax: rpkiRoas?startAddress=<IP address>

Syntax: rpkiRoas?autnum=<autonomous system number>

## Reverse Search

## Relationship with IP Network Object Class

# Autonomous System Provider Authorization Object Class

The Autonomous System Provider Authorization (ASPA) object class can contain the following members:

* objectClassName -- the string "rpki aspa"
* handle -- a string representing the RIR-unique identifier of the ASPA registration
* name -- a string representing an identifier assigned to the ASPA registration by the registration holder
* autnum -- an unsigned 32-bit integer representing the autonomous system number [@!RFC5396] of the registration holder
* providerAutnum -- an unsigned 32-bit integer representing the autonomous system number [@!RFC5396] of the AS that is
  authorized as a provider
* autoRenewed -- a boolean indicating if the ASPA is auto-renewed or not
* status -- a string indicating the validation state of the ASPA
* events -- events ([@!RFC9083, section 4.5]) representing the not-valid-before and not-valid-after dates of the
  end-entity certificate for the ASPA
* links -- links ([@!RFC9083, section 4.2]) for "self", and "related" to autonomous system number and IRR objects
* remarks -- see [@!RFC9083, section 4.3]

Here is an elided example of an ASPA object in RDAP:

```
{
  "objectClassName": "rpki aspa",
  "handle": "YYYY",
  "name": "ASPA-1",
  "autnum": 65536,
  "providerAutnum": 65537,
  "autoRenewed": true,
  "status": [ "valid" ],
  "events":
  [
    {
      "eventAction": "not valid before",
      "eventDate": "2024-04-27T23:59:59Z"
    },
    {
      "eventAction": "not valid after",
      "eventDate": "2025-04-27T23:59:59Z"
    },
    ...
  ],
  "links":
  [
    {
      "value": "https://example.net/rdap/rpkiAspa/YYYY",
      "rel": "self",
      "href": "https://example.net/rdap/rpkiAspa/YYYY",
      "type": "application/rdap+json"
    },
    {
      "value": "https://example.net/rdap/rpkiAspa/YYYY",
      "rel": "related",
      "href": "https://example.net/rdap/autnum/65536",
      "type": "application/rdap+json"
    },
    ...
  ],
  "remarks":
  [
    {
      "description": [ "An ASPA object in RDAP" ]
    }
  ],
}
```

## Lookup

The resource type path segment for exact match lookup of an ASPA object is "rpkiAspa".

The following lookup path segment is defined for an ASPA object:

Syntax: rpkiAspa/<handle>

For example:

```
https://example.net/rdap/rpkiAspa/YYYY
```

## Search

The resource type path segment for searching ASPA objects is "rpkiAspas".

The following search path segments are defined for ASPA objects:

Syntax: rpkiAspas?name=<name search pattern>

Syntax: rpkiAspas?autnum=<autonomous system number>

Syntax: rpkiAspas?providerAutnum=<autonomous system number>

## Reverse Search

## Relationship with Autonomous System Number Object Class

# RDAP Conformance

# Security Considerations

# IANA Considerations

## RDAP Extensions Registry

## RDAP Reverse Search Registry

## RDAP Reverse Search Mapping Registry

# Acknowledgements

Andy Newton helped clarify why the RPKI registration data in RDAP would be useful.

{backmatter}

