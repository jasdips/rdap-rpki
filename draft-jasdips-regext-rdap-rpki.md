%%%
Title = "An RDAP Extension for RPKI Registration Data"
area = "Applications and Real-Time Area (ART)"
workgroup = "Registration Protocols Extensions (regext)"
abbrev = "rdap-rpki"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-jasdips-regext-rdap-rpki-00"
stream = "IETF"
status = "standard"
date = 2024-07-05T00:00:00Z

[[author]]
initials="J."
surname="Singh"
fullname="Jasdip Singh"
organization="ARIN"
[author.address]
email = "jasdips@arin.net"

[[author]]
initials="A."
surname="Newton"
fullname="Andy Newton"
organization="ICANN"
[author.address]
email = "andy@hxr.us"

%%%

.# Abstract

The Resource Public Key Infrastructure (RPKI) is used to secure inter-domain routing on the internet. This document
defines a new Registration Data Access Protocol (RDAP) extension, "rpki1", for accessing the RPKI registration data in
the Internet Number Registries (INRs) through RDAP. An Internet Number Registry (INR) could be a Regional Internet
Registry (RIR), a National Internet Registry (NIR), or a Local Internet Registry (LIR).

{mainmatter}

# Introduction

The network operators are increasingly deploying the Resource Public Key Infrastructure (RPKI, [@!RFC6480]) to secure
inter-domain routing ([@RFC4271]) on the internet. RPKI enables internet number resource holders to cryptographically
assert about their registered IP addresses and autonomous system numbers to prevent route hijacks and leaks. To that
end, RPKI defines the following cryptographic profiles:

* Route Origin Authorization (ROA, [@!RFC9582]) where a Classless Inter-Domain Routing (CIDR, [@!RFC1519]) address block
  holder cryptographically asserts about the origin autonomous system (AS, [@RFC4271]) for routing that CIDR address
  block.
* Autonomous System Provider Authorization (ASPA, [@!I-D.ietf-sidrops-aspa-profile]) where an autonomous system number
  (ASN, [@!RFC5396]) holder cryptographically asserts about the provider AS for that ASN.
* BGPSec Router Certificate ([@!RFC8209]) where an ASN(s) holder cryptographically asserts that a router holding the
  corresponding private key is authorized to emit secure route advertisements on behalf of the AS(es) specified in the
  certificate.

This document defines a new RDAP extension, "rpki1", for accessing the RPKI registration data in the Internet Number
Registries (INRs) for aforementioned RPKI profiles through RDAP. An Internet Number Registry (INR) could be a Regional
Internet Registry (RIR), a National Internet Registry (NIR), or a Local Internet Registry (LIR).

The motivation here is that such RDAP data could complement the existing RPKI diagnostic tools when troubleshooting a
route hijack or leak, by conveniently providing access to registration information from an INR's database beside what's
inherently available from an RPKI profile object. There is registration metadata that is often needed for
troubleshooting that does not appear in, say, a ROA or a VRP (Verified ROA Payload); such as:

* Is it an auto-renewing ROA or not?
* When did the initial version of a ROA get published?
* Was a ROA created in conjunction with an Internet Routing Registry (IRR, [@RFC2622]) route?
* Which IRR route is related with a ROA?
* Which IP network is associated with a ROA?

Furthermore, correlating registered RPKI data with registered IP networks and autonomous system numbers would also give
access to the latter's contact information through RDAP entity objects, which should aid troubleshooting.

Beside the troubleshooting context, the ability to conveniently look up and search registered RPKI data through RDAP
would inform users irrespective of their RPKI expertise level.

This specification next defines RDAP object classes, as well as lookup and search path segments, for the ROA, ASPA, and
BGPSec Router Certificate registration data.

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

# Common Data Members {#common_data_members}

An RDAP object class for RPKI in (#roa_object_class), (#aspa_object_class), and (#bgpsec_router_cert_object_class) can
contain one or more of the following common members:

* handle -- a string representing the INR-unique identifier of the RPKI object registration
* name -- a string representing an identifier assigned to the RPKI object registration by the registration holder
* notValidBefore -- a string that contains the time and date in Zulu (Z) format with UTC offset of 00:00 ([@!RFC3339]),
  representing the not-valid-before date of the end-entity certificate for the RPKI object ([@!RFC6487, section 4])
* notValidAfter -- a string that contains the time and date in Zulu (Z) format with UTC offset of 00:00 ([@!RFC3339]),
  representing the not-valid-after date of the end-entity certificate for the RPKI object ([@!RFC6487, section 4])
* autoRenewed -- a boolean indicating if the registered RPKI object is auto-renewed or not
* publicationUri -- a URI string pointing to the location of the RPKI object within the RPKI repository; the URI scheme
  is "rsync", per [@!RFC6487, section 4]
* source -- a string representing the INR-unique identifier (handle) of the organization (entity) which is the
  authoritative source for the RPKI object; it could be an INR or a downstream organization
* rpkiType -- a string literal representing the type of the RPKI repository, with the following possible values:
    * "hosted" -- An INR fully hosts the RPKI repository for a downstream organization
    * "delegated" -- The downstream organization fully hosts its RPKI repository
    * "hybrid" -- The downstream organization runs the Certificate Authority (CA) for its RPKI repository whereas the
      INR hosts that organization's RPKI objects

# Route Origin Authorization {#roa}

## Object Class {#roa_object_class}

The Route Origin Authorization (ROA) object class can contain the following data members:

* objectClassName -- the string "rpki1_roa"
* handle -- see (#common_data_members)
* name -- see (#common_data_members)
* startAddress -- a string representing the starting IP address (a.k.a. CIDR prefix) of the CIDR address block, either
  IPv4 or IPv6 ([@!RFC9582, section 4])
* prefixLength -- a number representing the prefix length (a.k.a. CIDR length) of the CIDR address block; up to 32 for
  IPv4 and up to 128 for IPv6 ([@!RFC9582, section 4])
* ipVersion -- a string signifying the IP protocol version of the ROA: "v4" signifies an IPv4 ROA, and "v6" signifies
  an IPv6 ROA ([@!RFC9582, section 4])
* maxLength -- a number representing the maximum prefix length of the CIDR address block that the origin AS is
  authorized to advertise; up to 32 for IPv4 and up to 128 for IPv6 ([@!RFC9582, section 4])
* originAutnum -- an unsigned 32-bit integer representing the origin autonomous system number ([@!RFC9582, section 4])
* notValidBefore -- see (#common_data_members)
* notValidAfter -- see (#common_data_members)
* autoRenewed -- see (#common_data_members)
* publicationUri -- see (#common_data_members)
* source -- see (#common_data_members)
* rpkiType -- see (#common_data_members)
* events -- see [@!RFC9083, section 4.5]
* links -- links ([@!RFC9083, section 4.2]) for "self", and "related" to IP network and IRR (when defined) objects
* remarks -- see [@!RFC9083, section 4.3]

Here is an elided example of a ROA object in RDAP:

```
{
  "objectClassName": "rpki1_roa",
  "handle": "XXXX",
  "name": "ROA-1",
  "startAddress": "2001:db8::",
  "prefixLength": 48,
  "ipVersion": "v6",
  "maxLength": 64,
  "originAutnum": 65536,
  "notValidBefore": "2024-04-27T23:59:59Z",
  "notValidAfter": "2025-04-27T23:59:59Z",
  "autoRenewed": true,
  "publicationUri": "rsync://example.net/path/to/XXXX.roa",
  "source": "XYZ-RIR",
  "rpkiType": "hosted",
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
      "value": "https://example.net/rdap/rpki1/roa/XXXX",
      "rel": "self",
      "href": "https://example.net/rdap/rpki1/roa/XXXX",
      "type": "application/rdap+json"
    },
    {
      "value": "https://example.net/rdap/rpki1/roa/XXXX",
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

The resource type path segment for exact match lookup of a ROA object is "rpki1/roa".

The following lookup path segment is defined for a ROA object:

Syntax: rpki1/roa/<handle>

For example:

```
https://example.net/rdap/rpki1/roa/XXXX
```

## Search

The resource type path segment for searching ROA objects is "rpki1/roas".

The following search path segments are defined for ROA objects:

Syntax: rpki1/roas?name=<name search pattern>

Syntax: rpki1/roas?startAddress=<IP address>&&prefixLength=<CIDR length>

Syntax: rpki1/roas?originAutnum=<autonomous system number>

Searches for ROA information by name are specified using this form:

rpki1/roas?name=XXXX

XXXX is a search pattern per [@!RFC9082, section 4.1], representing the "name" property of a ROA, as described in
(#roa_object_class). The following URL would be used to find information for ROA names matching the "ROA-*" pattern:

```
https://example.net/rdap/rpki1/roas?name=ROA-*
```

Searches for ROA information by CIDR are specified using this form:

rpki1/roas?startAddress=YYYY&&prefixLength=ZZZZ

YYYY is an IP address representing the "startAddress" property of a ROA and ZZZZ is a CIDR length representing its
"prefixLength" property, as described in (#roa_object_class). The following URL would be used to find information for
the most-specific ROA matching the "2001:db8::/64" CIDR:

```
https://example.net/rdap/rpki1/roas?startAddress=2001%3Adb8%3A%3A&&prefixLength=64
```

Searches for ROA information by origin autonomous system number are specified using this form:

rpki1/roas?originAutnum=BBBB

BBBB is an autonomous system number representing the "originAutnum" property of a ROA, as described in
(#roa_object_class). The following URL would be used to find information for ROAs with origin autonomous system number
65536:

```
https://example.net/rdap/rpki1/roas?originAutnum=65536
```

### Search Results

The ROA search results are returned in the "rpki1_roaSearchResults" member, which is an array of ROA objects
((#roa_object_class)).

Here is an elided example of the search results when finding information for ROAs with origin autonomous system number
65536:

```
{
  "rdapConformance":
  [
    "rdap_level_0",
    "rpki1",
    ...
  ],
  ...
  "rpki1_roaSearchResults":
  [
    {
      "objectClassName": "rpki1_roa",
      "handle": "XXXX",
      "name": "ROA-1",
      "startAddress": "2001:db8::",
      "prefixLength": 48,
      "ipVersion": "v6",
      "maxLength": 64,
      "originAutnum": 65536,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": true,
      "publicationUri": "rsync://example.net/path/to/XXXX.roa",
      "source": "XYZ-RIR",
      "rpkiType": "hosted",
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
          "value": "https://example.net/rdap/rpki1/roa/XXXX",
          "rel": "self",
          "href": "https://example.net/rdap/rpki1/roa/XXXX",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/roa/XXXX",
          "rel": "related",
          "href": "https://example.net/rdap/ip/2001:db8::/48",
          "type": "application/rdap+json"
        },
        ...
      ]
    },
    ...
  ]
}
```

## Reverse Search

Per [@!RFC9536, section 2], if a server receives a reverse search query with a searchable resource type of "ips"
([@!I-D.ietf-regext-rdap-rir-search, section 5]), a related resource type of "rpki1_roa", and a ROA property of
"originAutnum" or "startAddress", then the reverse search will be performed on the IP network objects from its data
store.

(#reverse_search_registry) and (#reverse_search_mapping_registry) include requests to register new entries for IP
network searches in the RDAP Reverse Search and RDAP Reverse Search Mapping IANA registries when the related resource
type is "rpki1_roa".

## Relationship with IP Network Object Class

It would be useful to show all the ROAs associated with an IP network object. To that end, this extension adds a new
"rpki1_roas" member to the IP Network object class ([@!RFC9083, section 5.4]):

* rpki1_roas -- an array of ROA objects ((#roa_object_class)) for the IP network; if the array is too large, the server
  MAY truncate it, per [@!RFC9083, section 9]

Here is an elided example for an IP network object with ROAs:

```
{
  "objectClassName": "ip network",
  "handle": "ZZZZ-RIR",
  "startAddress": "2001:db8::",
  "endAddress": "2001:db8:ffff:ffff:ffff:ffff:ffff:ffff",
  "ipVersion": "v6",
  ...
  "rpki1_roas":
  [
    {
      "objectClassName": "rpki1_roa",
      "handle": "XXXX",
      "name": "ROA-1",
      "startAddress": "2001:db8::",
      "prefixLength": 48,
      "ipVersion": "v6",
      "maxLength": 64,
      "originAutnum": 65536,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": true,
      "publicationUri": "rsync://example.net/path/to/XXXX.roa",
      "source": "XYZ-RIR",
      "rpkiType": "hosted",
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
          "value": "https://example.net/rdap/rpki1/roa/XXXX",
          "rel": "self",
          "href": "https://example.net/rdap/rpki1/roa/XXXX",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/roa/XXXX",
          "rel": "related",
          "href": "https://example.net/rdap/ip/2001:db8::/48",
          "type": "application/rdap+json"
        },
        ...
      ]
    },
    {
      "objectClassName": "rpki1_roa",
      "handle": "YYYY",
      "name": "ROA-2",
      "startAddress": "2001:db8:1::",
      "prefixLength": 48,
      "ipVersion": "v6",
      "maxLength": 64,
      "originAutnum": 65537,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": false,
      "publicationUri": "rsync://example.net/path/to/YYYY.roa",
      "source": "XYZ-RIR",
      "rpkiType": "hosted",
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
          "value": "https://example.net/rdap/rpki1/roa/YYYY",
          "rel": "self",
          "href": "https://example.net/rdap/rpki1/roa/YYYY",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/roa/YYYY",
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

# Autonomous System Provider Authorization {#aspa}

## Object Class {#aspa_object_class}

The Autonomous System Provider Authorization (ASPA) object class can contain the following data members:

* objectClassName -- the string "rpki1_aspa"
* handle -- see (#common_data_members)
* name -- see (#common_data_members)
* autnum -- an unsigned 32-bit integer representing the autonomous system number of the registration holder
  ([@!I-D.ietf-sidrops-aspa-profile, section 3])
* providerAutnum -- an unsigned 32-bit integer representing the autonomous system number of the AS that is authorized
  as a provider ([@!I-D.ietf-sidrops-aspa-profile, section 3])
* notValidBefore -- see (#common_data_members)
* notValidAfter -- see (#common_data_members)
* autoRenewed -- see (#common_data_members)
* publicationUri -- see (#common_data_members)
* source -- see (#common_data_members)
* rpkiType -- see (#common_data_members)
* events -- see [@!RFC9083, section 4.5]
* links -- links ([@!RFC9083, section 4.2]) for "self", and "related" to autonomous system number and IRR (when defined)
  objects
* remarks -- see [@!RFC9083, section 4.3]

Here is an elided example of an ASPA object in RDAP:

```
{
  "objectClassName": "rpki1_aspa",
  "handle": "XXXX",
  "name": "ASPA-1",
  "autnum": 65536,
  "providerAutnum": 65542,
  "notValidBefore": "2024-04-27T23:59:59Z",
  "notValidAfter": "2025-04-27T23:59:59Z",
  "autoRenewed": true,
  "publicationUri": "rsync://example.net/path/to/XXXX.aspa",
  "source": "XYZ-RIR",
  "rpkiType": "hosted",
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
      "value": "https://example.net/rdap/rpki1/aspa/XXXX",
      "rel": "self",
      "href": "https://example.net/rdap/rpki1/aspa/XXXX",
      "type": "application/rdap+json"
    },
    {
      "value": "https://example.net/rdap/rpki1/aspa/XXXX",
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

The resource type path segment for exact match lookup of an ASPA object is "rpki1/aspa".

The following lookup path segment is defined for an ASPA object:

Syntax: rpki1/aspa/<handle>

For example:

```
https://example.net/rdap/rpki1/aspa/YYYY
```

## Search

The resource type path segment for searching ASPA objects is "rpki1/aspas".

The following search path segments are defined for ASPA objects:

Syntax: rpki1/aspas?name=<name search pattern>

Syntax: rpki1/aspas?autnum=<autonomous system number>

Syntax: rpki1/aspas?providerAutnum=<autonomous system number>

Searches for ASPA information by name are specified using this form:

rpki1/aspas?name=XXXX

XXXX is a search pattern per [@!RFC9082, section 4.1], representing the "name" property of an ASPA, as described in
(#aspa_object_class). The following URL would be used to find information for ASPA names matching the "ASPA-*" pattern:

```
https://example.net/rdap/rpki1/aspas?name=ASPA-*
```

Searches for ASPA information by autonomous system number are specified using this form:

rpki1/aspas?autnum=YYYY

YYYY is an autonomous system number representing the "autnum" property of an ASPA, as described in (#aspa_object_class).
The following URL would be used to find information for ASPAs with autonomous system number 65536:

```
https://example.net/rdap/rpki1/aspas?autnum=65536
```

Searches for ASPA information by provider autonomous system number are specified using this form:

rpki1/aspas?providerAutnum=ZZZZ

ZZZZ is an autonomous system number representing the "providerAutnum" property of an ASPA, as described in
(#aspa_object_class). The following URL would be used to find information for ASPAs with provider autonomous system
number 65542:

```
https://example.net/rdap/rpki1/aspas?providerAutnum=65542
```

### Search Results

The ASPA search results are returned in the "rpki1_aspaSearchResults" member, which is an array of ASPA objects
((#aspa_object_class)).

Here is an elided example of the search results when finding information for ASPAs with autonomous system number 65536:

```
{
  "rdapConformance":
  [
    "rdap_level_0",
    "rpki1",
    ...
  ],
  ...
  "rpki1_aspaSearchResults":
  [
    {
      "objectClassName": "rpki1_aspa",
      "handle": "XXXX",
      "name": "ASPA-1",
      "autnum": 65536,
      "providerAutnum": 65542,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": true,
      "publicationUri": "rsync://example.net/path/to/XXXX.aspa",
      "source": "XYZ-RIR",
      "rpkiType": "hosted",
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
          "value": "https://example.net/rdap/rpki1/aspa/XXXX",
          "rel": "self",
          "href": "https://example.net/rdap/rpki1/aspa/XXXX",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/aspa/XXXX",
          "rel": "related",
          "href": "https://example.net/rdap/autnum/65536",
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

## Reverse Search

Per [@!RFC9536, section 2], if a server receives a reverse search query with a searchable resource type of "autnums"
([@!I-D.ietf-regext-rdap-rir-search, section 5]), a related resource type of "rpki1_aspa", and an ASPA property of
"autnum" or "providerAutnum", then the reverse search will be performed on the autonomous system number objects from its
data store.

(#reverse_search_registry) and (#reverse_search_mapping_registry) include requests to register new entries for
autonomous system number searches in the RDAP Reverse Search and RDAP Reverse Search Mapping IANA registries when the
related resource type is "rpki1_aspa".

## Relationship with Autonomous System Number Object Class

It would be useful to show all the ASPAs associated with an autonomous system number object. To that end, this extension
adds a new "rpki1_aspas" member to the Autonomous System Number object class ([@!RFC9083, section 5.5]):

* rpki1_aspas -- an array of ASPA objects ((#aspa_object_class)) for the autonomous system number; if the array is too
  large, the server MAY truncate it, per [@!RFC9083, section 9]

Here is an elided example for an autonomous system number object with ASPAs:

```
{
  "objectClassName": "autnum",
  "handle": "ZZZZ-RIR",
  "startAutnum": 65536,
  "endAutnum": 65541,
  ...
  "rpki1_aspas":
  [
    {
      "objectClassName": "rpki1_aspa",
      "handle": "XXXX",
      "name": "ASPA-1",
      "autnum": 65536,
      "providerAutnum": 65542,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": true,
      "publicationUri": "rsync://example.net/path/to/XXXX.aspa",
      "source": "XYZ-RIR",
      "rpkiType": "hosted",
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
          "value": "https://example.net/rdap/rpki1/aspa/XXXX",
          "rel": "self",
          "href": "https://example.net/rdap/rpki1/aspa/XXXX",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/aspa/XXXX",
          "rel": "related",
          "href": "https://example.net/rdap/autnum/65536",
          "type": "application/rdap+json"
        },
        ...
      ],
      ...
    },
    {
      "objectClassName": "rpki1_aspa",
      "handle": "YYYY",
      "name": "ASPA-2",
      "autnum": 65537,
      "providerAutnum": 65543,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": false,
      "publicationUri": "rsync://example.net/path/to/YYYY.aspa",
      "source": "XYZ-RIR",
      "rpkiType": "hosted",
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
          "value": "https://example.net/rdap/rpki1/aspa/YYYY",
          "rel": "self",
          "href": "https://example.net/rdap/rpki1/aspa/YYYY",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/aspa/YYYY",
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

# BGPSec Router Certificate {#bgpsec_router_cert}

## Object Class {#bgpsec_router_cert_object_class}

The BGPSec Router Certificate object class can contain the following data members:

* objectClassName -- the string "rpki1_bgpsec_router_cert"
* handle -- see (#common_data_members)
* serialNumber -- a string representing the unique identifier for the certificate ([@!RFC6487, section 4])
* issuer -- a string representing the Certificate Authority (CA) that issued the certificate ([@!RFC6487, section 4])
* signatureAlgorithm -- a string representing the algorithm used by the CA to sign the certificate
  ([@!RFC6487, section 4])
* subject -- a string representing the identity of the router ([@!RFC8209, section 3.1.1])
* subjectPublicKeyInfo -- an object representing the subject's public key information ([@!RFC8208, section 3.1]), with
  the following members:
    * publicKeyAlgorithm -- a string representing the algorithm for the public key
    * publicKey -- a string representation of the public key
* subjectKeyIdentifier -- a string, typically Base64-encoded, representing the unique identifier for the public key
  ([@!RFC6487, section 4])
* autnums -- an array of unsigned 32-bit integers, each representing the autonomous system number that the router emits
  secure route advertisements on behalf of ([@!RFC8209, section 3.1.3.5])
* notValidBefore -- see (#common_data_members)
* notValidAfter -- see (#common_data_members)
* autoRenewed -- see (#common_data_members)
* publicationUri -- see (#common_data_members)
* source -- see (#common_data_members)
* rpkiType -- see (#common_data_members)
* events -- see [@!RFC9083, section 4.5]
* links -- links ([@!RFC9083, section 4.2]) for "self", and "related" to one or more autonomous system number objects
* remarks -- see [@!RFC9083, section 4.3]

Here is an elided example of a BGPSec Router Certificate object in RDAP:

```
{
  "objectClassName": "rpki1_bgpsec_router_cert",
  "handle": "ABCD",
  "serialNumber": "1234",
  "issuer": "CN=ISP-CA",
  "signatureAlgorithm": "ecdsa-with-SHA256",
  "subject": "CN=ROUTER-ISP-ASNS",
  "subjectPublicKeyInfo":
  {
    "publicKeyAlgorithm": "id-ecPublicKey",
    "publicKey": "..."
  },
  "subjectKeyIdentifier": "hOcGgxqXDa7mYv78fR+sGBKMtWJqItSLfaIYJDKYi8A=",
  "autnums":
  [
    65536,
    65537
  ],
  "notValidBefore": "2024-04-27T23:59:59Z",
  "notValidAfter": "2025-04-27T23:59:59Z",
  "publicationUri": "rsync://example.net/path/to/ABCD.cer",
  "source": "XYZ-RIR",
  "rpkiType": "hosted",
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
      "value": "https://example.net/rdap/rpki1/bgpsec_router_cert/ABCD",
      "rel": "self",
      "href": "https://example.net/rdap/rpki1/bgpsec_router_cert/ABCD",
      "type": "application/rdap+json"
    },
    {
      "value": "https://example.net/rdap/rpki1/bgpsec_router_cert/ABCD",
      "rel": "related",
      "href": "https://example.net/rdap/autnum/65536",
      "type": "application/rdap+json"
    },
    {
      "value": "https://example.net/rdap/rpki1/bgpsec_router_cert/ABCD",
      "rel": "related",
      "href": "https://example.net/rdap/autnum/65537",
      "type": "application/rdap+json"
    },
    ...
  ],
  "remarks":
  [
    {
      "description": [ "A BGPSec Router Certificate object in RDAP" ]
    }
  ]
}
```

## Lookup

The resource type path segment for exact match lookup of a BGPSec Router Certificate object is
"rpki1/bgpsec_router_cert".

The following lookup path segment is defined for a BGPSec Router Certificate object:

Syntax: rpki1/bgpsec_router_cert/<handle>

For example:

```
https://example.net/rdap/rpki1/bgpsec_router_cert/ABCD
```

## Search

The resource type path segment for searching BGPSec Router Certificate objects is "rpki1/bgpsec_router_certs".

The following search path segments are defined for BGPSec Router Certificate objects:

Syntax: rpki1/bgpsec_router_certs?handle=<handle search pattern>

Syntax: rpki1/bgpsec_router_certs?issuer=<issuer search pattern>

Syntax: rpki1/bgpsec_router_certs?subject=<subject search pattern>

Syntax: rpki1/bgpsec_router_certs?subjectKeyIdentifier=<subject key identifier>

Syntax: rpki1/bgpsec_router_certs?autnum=<autonomous system number>

Searches for BGPSec router certificate information by handle are specified using this form:

rpki1/bgpsec_router_certs?handle=XXXX

XXXX is a search pattern per [@!RFC9082, section 4.1], representing the "handle" property of a BGPSec Router Certificate
object, as described in (#bgpsec_router_cert_object_class). The following URL would be used to find information for
BGPSec Router Certificate objects with handle matching the "ABC*" pattern:

```
https://example.net/rdap/rpki1/bgpsec_router_certs?handle=ABC*
```

Searches for BGPSec router certificate information by certificate issuer are specified using this form:

rpki1/bgpsec_router_certs?issuer=YYYY

YYYY is a search pattern per [@!RFC9082, section 4.1], representing the "issuer" property of a BGPSec Router Certificate
object, as described in (#bgpsec_router_cert_object_class). The following URL would be used to find information for
BGPSec Router Certificate objects with issuer matching the "CN=ISP-C*" pattern:

```
https://example.net/rdap/rpki1/bgpsec_router_certs?issuer=CN%3DISP-C*
```

Searches for BGPSec router certificate information by certificate subject are specified using this form:

rpki1/bgpsec_router_certs?subject=ZZZZ

ZZZZ is a search pattern per [@!RFC9082, section 4.1], representing the "subject" property of a BGPSec Router
Certificate object, as described in (#bgpsec_router_cert_object_class). The following URL would be used to find
information for BGPSec Router Certificate objects with subject matching the "CN=ROUTER-ASN-655*" pattern:

```
https://example.net/rdap/rpki1/bgpsec_router_certs?subject=CN%3DROUTER-ASN-6553*
```

Searches for BGPSec router certificate information by subject key identifier are specified using this form:

rpki1/bgpsec_router_certs?subjectKeyIdentifier=BBBB

BBBB is a string representing the "subjectKeyIdentifier" property of a BGPSec Router Certificate object, as described in
(#bgpsec_router_cert_object_class). The following URL would be used to find a BGPSec Router Certificate object with
subject key identifier matching the "hOcGgxqXDa7mYv78fR+sGBKMtWJqItSLfaIYJDKYi8A=" string:

```
https://example.net/rdap/rpki1/bgpsec_router_certs?subjectKeyIdentifier=hOcGgxqXDa7mYv78fR+sGBKMtWJqItSLfaIYJDKYi8A=
```

Searches for BGPSec router certificate information by autonomous system number are specified using this form:

rpki1/bgpsec_router_certs?autnum=CCCC

CCCC is an autonomous system number representing one of the elements of the "autnums" array of a BGPSec Router
Certificate object, as described in (#bgpsec_router_cert_object_class). The following URL would be used to find a BGPSec
Router Certificate object with autonomous system number 65536:

```
https://example.net/rdap/rpki1/bgpsec_router_certs?autnum=65536
```

### Search Results

The BGPSec Router Certificate search results are returned in the "rpki1_bgpsecRouterCertSearchResults" member, which is
an array of BGPSec Router Certificate objects ((#bgpsec_router_cert_object_class)).

Here is an elided example of the search results when finding information for BGPSec Router Certificate objects with
issuer matching the "CN=ISP-C*" pattern:

```
{
  "rdapConformance":
  [
    "rdap_level_0",
    "rpki1",
    ...
  ],
  ...
  "rpki1_bgpsecRouterCertSearchResults":
  [
    {
      "objectClassName": "rpki1_bgpsec_router_cert",
      "handle": "ABCD",
      "serialNumber": "1234",
      "issuer": "CN=ISP-CA",
      "signatureAlgorithm": "ecdsa-with-SHA256",
      "subject": "CN=ROUTER-ISP-ASNS",
      "subjectPublicKeyInfo":
      {
        "publicKeyAlgorithm": "id-ecPublicKey",
        "publicKey": "..."
      },
      "subjectKeyIdentifier": "hOcGgxqXDa7mYv78fR+sGBKMtWJqItSLfaIYJDKYi8A=",
      "autnums":
      [
        65536,
        65537
      ],
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "publicationUri": "rsync://example.net/path/to/ABCD.cer",
      "source": "XYZ-RIR",
      "rpkiType": "hosted",
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
          "value": "https://example.net/rdap/rpki1/bgpsec_router_cert/ABCD",
          "rel": "self",
          "href": "https://example.net/rdap/rpki1/bgpsec_router_cert/ABCD",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/bgpsec_router_cert/ABCD",
          "rel": "related",
          "href": "https://example.net/rdap/autnum/65536",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/bgpsec_router_cert/ABCD",
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

## Reverse Search

Per [@!RFC9536, section 2], if a server receives a reverse search query with a searchable resource type of "autnums"
([@!I-D.ietf-regext-rdap-rir-search, section 5]), a related resource type of "rpki1_bgpsec_router_cert", and a BGPSec
Router Certificate property of "handle", then the reverse search will be performed on the autonomous system number
objects from its data store.

(#reverse_search_registry) and (#reverse_search_mapping_registry) include requests to register new entries for
autonomous system number searches in the RDAP Reverse Search and RDAP Reverse Search Mapping IANA registries when the
related resource type is "rpki1_bgpsec_router_cert".

# RDAP Conformance

A server that supports the functionality specified in this document MUST include the "rpki1" string literal in the
"rdapConformance" array of its responses.

To be in compliance with this specification, a registry with RPKI data would need to implement at the least one of the
newly defined RDAP object classes ((#roa), (#aspa), (#bgpsec_router_cert)), and if possible, all.

# Security Considerations

The RDAP extension in this document MUST NOT be used to directly influence Internet routing. Neither RDAP nor this
extension define the necessary security properties or distribution mechanisms required to securely add, remove, or
modify Internet routes.

This document does not introduce any new security considerations past those already discussed in the RDAP protocol
specifications ([@RFC7481], [@RFC9560]).

# IANA Considerations

## RDAP Extensions Registry

IANA is requested to register the following values in the RDAP Extensions Registry at
https://www.iana.org/assignments/rdap-extensions/:

Extension identifier: rpki1

Registry operator: Any

Published specification: [this document]

Contact: IETF <iesg@ietf.org>

Intended usage: This extension identifier is used for accessing the RPKI registration data through RDAP.

## RDAP Reverse Search Registry {#reverse_search_registry}

IANA is requested to register the following entries in the RDAP Reverse Search Registry at
https://www.iana.org/assignments/rdap-reverse-search/:

Searchable Resource Type: ips

Related Resource Type: rpki1_roa

Property: originAutnum

Description: The server supports the IP search based on the origin autonomous system number of an associated RPKI ROA.

Registrant Name: IETF

Registrant Contact Information: iesg@ietf.org

Reference: [this document]

Searchable Resource Type: ips

Related Resource Type: rpki1_roa

Property: startAddress

Description: The server supports the IP search based on the starting IP address (a.k.a. CIDR prefix) of the CIDR address
block of an associated RPKI ROA.

Registrant Name: IETF

Registrant Contact Information: iesg@ietf.org

Reference: [this document]

Searchable Resource Type: autnums

Related Resource Type: rpki1_aspa

Property: autnum

Description: The server supports the autnum search based on the autonomous system number of an associated RPKI ASPA.

Registrant Name: IETF

Registrant Contact Information: iesg@ietf.org

Reference: [this document]

Searchable Resource Type: autnums

Related Resource Type: rpki1_aspa

Property: providerAutnum

Description: The server supports the autnum search based on the provider autonomous system number of an associated RPKI
ASPA.

Registrant Name: IETF

Registrant Contact Information: iesg@ietf.org

Reference: [this document]

Searchable Resource Type: autnums

Related Resource Type: rpki1_bgpsec_router_cert

Property: autnum

Description: The server supports the autnum search based on the handle of an associated RPKI BGPSec Router Certificate
object.

Registrant Name: IETF

Registrant Contact Information: iesg@ietf.org

Reference: [this document]

## RDAP Reverse Search Mapping Registry {#reverse_search_mapping_registry}

IANA is requested to register the following entries in the RDAP Reverse Search Mapping Registry at
https://www.iana.org/assignments/rdap-reverse-search-mapping/:

Searchable Resource Type: ips

Related Resource Type: rpki1_roa

Property: originAutnum

Property Path: $.originAutnum

Registrant Name: IETF

Registrant Contact Information: iesg@ietf.org

Reference: [this document]

Searchable Resource Type: ips

Related Resource Type: rpki1_roa

Property: startAddress

Property Path: $.startAddress

Registrant Name: IETF

Registrant Contact Information: iesg@ietf.org

Reference: [this document]

Searchable Resource Type: autnums

Related Resource Type: rpki1_aspa

Property: autnum

Property Path: $.autnum

Registrant Name: IETF

Registrant Contact Information: iesg@ietf.org

Reference: [this document]

Searchable Resource Type: autnums

Related Resource Type: rpki1_aspa

Property: providerAutnum

Property Path: $.providerAutnum

Registrant Name: IETF

Registrant Contact Information: iesg@ietf.org

Reference: [this document]

Searchable Resource Type: autnums

Related Resource Type: rpki1_bgpsec_router_cert

Property: handle

Property Path: $.handle

Registrant Name: IETF

Registrant Contact Information: iesg@ietf.org

Reference: [this document]

# Acknowledgements

Job Snijders suggested accessing the BGPSec Router Certificate registration data as well through this RDAP extension.
Ties de Kock also provided valuable feedback for this document.

{backmatter}

