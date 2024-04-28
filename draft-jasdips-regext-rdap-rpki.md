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
date = 2024-04-28T00:00:00Z

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

# Route Origin Authorization

## Object Class {#roa_object_class}

The Route Origin Authorization (ROA) object class can contain the following members:

* objectClassName -- the string "rpki roa"
* handle -- a string representing the RIR-unique identifier of the ROA registration
* name -- a string representing an identifier assigned to the ROA registration by the registration holder
* startAddress -- a string representing the starting IP address (a.k.a. CIDR prefix) of the CIDR address block, either
  IPv4 or IPv6
* prefixLength -- a number representing the prefix length (a.k.a. CIDR length) of the CIDR address block; up to 32 for
  IPv4 and up to 128 for IPv6
* ipVersion -- a string signifying the IP protocol version of the ROA: "v4" signifies an IPv4 ROA, and "v6" signifies
  an IPv6 ROA
* maxLength -- a number representing the maximum prefix length of the CIDR address block that the origin AS is
  authorized to advertise; up to 32 for IPv4 and up to 128 for IPv6
* originAutnum -- an unsigned 32-bit integer representing the origin autonomous system number [@!RFC5396]
* notValidBefore -- a string that contains the time and date representing the not-valid-before date of the end-entity
  certificate for the ROA
* notValidAfter -- a string that contains the time and date representing the not-valid-after date of the end-entity
  certificate for the ROA
* autoRenewed -- a boolean indicating if the ROA is auto-renewed or not
* status -- a string indicating the validation state of the ROA
* events -- see [@!RFC9083, section 4.5]
* links -- links ([@!RFC9083, section 4.2]) for "self", and "related" to IP network and IRR objects
* remarks -- see [@!RFC9083, section 4.3]

Here is an elided example of a ROA object in RDAP:

```
{
  "objectClassName": "rpki roa",
  "handle": "XXXX",
  "name": "ROA-1",
  "startAddress": "2001:db8::",
  "prefixLength": 48,
  "ipVersion": "v6",
  "maxLength": 64,
  "originAutnum": 65536,
  "notValidBefore": "2024-04-27T23:59:59Z",
  "notValidAfter": "2025-04-27T23:59:59Z"
  "autoRenewed": true,
  "status": [ "valid" ],
  "events":
  [
    {
      "eventAction": "registration",
      "eventDate": "2024-01-01T23:59:59Z"
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
  ]
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

Syntax: rpkiRoas?startAddress=<IP address>&&prefixLength=<CIDR length>

Syntax: rpkiRoas?autnum=<autonomous system number>

Searches for ROA information by name are specified using this form:

rpkiRoas?name=XXXX

XXXX is a search pattern representing the "name" property of a ROA, as described in (#roa_object_class). The following
URL would be used to find information for ROA names matching the "ROA-*" pattern:

```
https://example.net/rdap/rpkiRoas?name=ROA-*
```

Searches for ROA information by CIDR are specified using this form:

rpkiRoas?startAddress=XXXX&&prefixLength=YYYY

XXXX is an IP address representing the "startAddress" property of a ROA and YYYY is a CIDR length representing its
"prefixLength" property, as described in (#roa_object_class). The following URL would be used to find information for
the most-specific ROA matching the "2001:db8::/64" CIDR:

```
https://example.net/rdap/rpkiRoas?startAddress=2001%3Adb8%3A%3A&&prefixLength=64
```

Searches for ROA information by autonomous system number are specified using this form:

rpkiRoas?autnum=ZZZZ

ZZZZ is an autonomous system number representing the "autnum" property of a ROA, as described in (#roa_object_class).
The following URL would be used to find information for a ROA with origin autonomous system number 65536:

```
https://example.net/rdap/rpkiRoas?autnum=65536
```

## Reverse Search

## Relationship with IP Network Object Class

It would be useful to show all the ROAs associated with an IP network. To that end, this extension adds a new "rpkiRoas"
member to the IP Network object class ([@!RFC9083, section 5.4]):

* rpkiRoas -- an array of ROA objects ((#roa_object_class)) for the IP network

Here is an elided example for an IP network with ROAs:

```
{
  "objectClassName": "ip network",
  "handle": "ZZZZ-RIR",
  "startAddress": "2001:db8::",
  "endAddress": "2001:db8:ffff:ffff:ffff:ffff:ffff:ffff",
  "ipVersion": "v6",
  ...
  "rpkiRoas":
  [
    {
      "objectClassName": "rpki roa",
      "handle": "XXXX",
      "name": "ROA-1",
      "startAddress": "2001:db8::",
      "prefixLength": 48,
      "ipVersion": "v6",
      "maxLength": 64,
      "originAutnum": 65536,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z"
      "autoRenewed": true,
      "status": [ "valid" ],
      "events":
      [
        {
          "eventAction": "registration",
          "eventDate": "2024-01-01T23:59:59Z"
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
      ]
    },
    {
      "objectClassName": "rpki roa",
      "handle": "YYYY",
      "name": "ROA-2",
      "startAddress": "2001:db8:1::",
      "prefixLength": 48,
      "ipVersion": "v6",
      "maxLength": 64,
      "originAutnum": 65537,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z"
      "autoRenewed": true,
      "status": [ "not found" ],
      "events":
      [
        {
          "eventAction": "registration",
          "eventDate": "2024-01-01T23:59:59Z"
        },
        ...
      ],
      "links":
      [
        {
          "value": "https://example.net/rdap/rpkiRoa/YYYY",
          "rel": "self",
          "href": "https://example.net/rdap/rpkiRoa/YYYY",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpkiRoa/YYYY",
          "rel": "related",
          "href": "https://example.net/rdap/ip/2001:db8:1::/48",
          "type": "application/rdap+json"
        },
        ...
      ]
    },
    ...
  ]
}
```

# Autonomous System Provider Authorization

## Object Class {#aspa_object_class}

The Autonomous System Provider Authorization (ASPA) object class can contain the following members:

* objectClassName -- the string "rpki aspa"
* handle -- a string representing the RIR-unique identifier of the ASPA registration
* name -- a string representing an identifier assigned to the ASPA registration by the registration holder
* autnum -- an unsigned 32-bit integer representing the autonomous system number [@!RFC5396] of the registration holder
* providerAutnum -- an unsigned 32-bit integer representing the autonomous system number [@!RFC5396] of the AS that is
  authorized as a provider
* notValidBefore -- a string that contains the time and date representing the not-valid-before date of the end-entity
  certificate for the ASPA
* notValidAfter -- a string that contains the time and date representing the not-valid-after date of the end-entity
  certificate for the ASPA
* autoRenewed -- a boolean indicating if the ASPA is auto-renewed or not
* status -- a string indicating the validation state of the ASPA
* events -- see [@!RFC9083, section 4.5]
* links -- links ([@!RFC9083, section 4.2]) for "self", and "related" to autonomous system number and IRR objects
* remarks -- see [@!RFC9083, section 4.3]

Here is an elided example of an ASPA object in RDAP:

```
{
  "objectClassName": "rpki aspa",
  "handle": "YYYY",
  "name": "ASPA-1",
  "autnum": 65536,
  "providerAutnum": 65542,
  "notValidBefore": "2024-04-27T23:59:59Z",
  "notValidAfter": "2025-04-27T23:59:59Z"
  "autoRenewed": true,
  "status": [ "valid" ],
  "events":
  [
    {
      "eventAction": "registration",
      "eventDate": "2024-01-01T23:59:59Z"
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
  ]
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

Searches for ASPA information by name are specified using this form:

rpkiAspas?name=XXXX

XXXX is a search pattern representing the "name" property of an ASPA, as described in (#aspa_object_class). The
following URL would be used to find information for ASPA names matching the "ASPA-*" pattern:

```
https://example.net/rdap/rpkiAspas?name=ASPA-*
```

Searches for ASPA information by autonomous system number are specified using this form:

rpkiAspas?autnum=YYYY

YYYY is an autonomous system number representing the "autnum" property of an ASPA, as described in (#aspa_object_class).
The following URL would be used to find information for an ASPA with autonomous system number 65536:

```
https://example.net/rdap/rpkiAspas?autnum=65536
```

Searches for ASPA information by provider autonomous system number are specified using this form:

rpkiAspas?providerAutnum=ZZZZ

ZZZZ is an autonomous system number representing the "providerAutnum" property of an ASPA, as described in
(#aspa_object_class). The following URL would be used to find information for ASPAs with provider autonomous system
number 65542:

```
https://example.net/rdap/rpkiAspas?providerAutnum=65542
```

## Reverse Search

## Relationship with Autonomous System Number Object Class

It would be useful to show all the ASPAs associated with an autonomous system number. To that end, this extension adds a
new "rpkiAspas" member to the Autonomous System Number object class ([@!RFC9083, section 5.5]):

* rpkiAspas -- an array of ASPA objects ((#aspa_object_class)) for the autonomous system number

Here is an elided example for an autonomous system number with ASPAs:

```
{
  "objectClassName": "autnum",
  "handle": "ZZZZ-RIR",
  "startAutnum": 65536,
  "endAutnum": 65541,
  ...
  "rpkiAspas":
  [
    {
      "objectClassName": "rpki aspa",
      "handle": "XXXX",
      "name": "ASPA-1",
      "autnum": 65536,
      "providerAutnum": 65542,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z"
      "autoRenewed": true,
      "status": [ "valid" ],
      "events":
      [
        {
          "eventAction": "registration",
          "eventDate": "2024-01-01T23:59:59Z"
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
      ...
    },
    {
      "objectClassName": "rpki aspa",
      "handle": "YYYY",
      "name": "ASPA-2",
      "autnum": 65537,
      "providerAutnum": 65543,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z"
      "autoRenewed": true,
      "status": [ "not found" ],
      "events":
      [
        {
          "eventAction": "registration",
          "eventDate": "2024-01-01T23:59:59Z"
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
          "href": "https://example.net/rdap/autnum/65537",
          "type": "application/rdap+json"
        },
        ...
      ],
      ...
    },
    ...
  ]
}
```

# RDAP Conformance

# Security Considerations

# IANA Considerations

## RDAP Extensions Registry

IANA is requested to register the following values in the RDAP Extensions Registry  at
https://www.iana.org/assignments/rdap-extensions/:

Extension identifier: rpki1

Registry operator: Any

Published specification: [this document]

Contact: IETF <iesg@ietf.org>

Intended usage: This extension identifier is used for accessing the RPKI registration data through RDAP.

Extension identifier: rpkiRoa

Registry operator: Any

Published specification: [this document]

Contact: IETF <iesg@ietf.org>

Intended usage: This extension identifier is used for accessing the RPKI registration data through RDAP.

Extension identifier: rpkiRoas

Registry operator: Any

Published specification: [this document]

Contact: IETF <iesg@ietf.org>

Intended usage: This extension identifier is used for accessing the RPKI registration data through RDAP.

Extension identifier: rpkiRoaSearchResults

Registry operator: Any

Published specification: [this document]

Contact: IETF <iesg@ietf.org>

Intended usage: This extension identifier is used for accessing the RPKI registration data through RDAP.

Extension identifier: rpkiAspa

Registry operator: Any

Published specification: [this document]

Contact: IETF <iesg@ietf.org>

Intended usage: This extension identifier is used for accessing the RPKI registration data through RDAP.

Extension identifier: rpkiAspas

Registry operator: Any

Published specification: [this document]

Contact: IETF <iesg@ietf.org>

Intended usage: This extension identifier is used for accessing the RPKI registration data through RDAP.

Extension identifier: rpkiAspaSearchResults

Registry operator: Any

Published specification: [this document]

Contact: IETF <iesg@ietf.org>

Intended usage: This extension identifier is used for accessing the RPKI registration data through RDAP.

## RDAP Reverse Search Registry

## RDAP Reverse Search Mapping Registry

# Acknowledgements

Andy Newton helped clarify why the RPKI registration data in RDAP would be useful.

{backmatter}

