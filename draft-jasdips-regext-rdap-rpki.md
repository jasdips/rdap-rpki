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
date = 2024-10-04T00:00:00Z

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
the Internet Number Registry System (INRS) through RDAP. The Internet Number Registry System (INRS) is composed of
Regional Internet Registries (RIRs), National Internet Registries (NIRs), and Local Internet Registries (LIRs).

{mainmatter}

# Introduction

The network operators are increasingly deploying the Resource Public Key Infrastructure (RPKI, [@!RFC6480]) to secure
inter-domain routing ([@RFC4271]) on the internet. RPKI enables Internet Number Resource (INR) holders to
cryptographically assert about their registered IP addresses and autonomous system numbers to prevent route hijacks and
leaks. To that end, RPKI defines the following cryptographic profiles:

* Route Origin Authorization (ROA, [@!RFC9582]) where a Classless Inter-Domain Routing (CIDR, [@!RFC1519]) address block
  holder cryptographically asserts about the origin autonomous system (AS, [@RFC4271]) for routing that CIDR address
  block.
* Autonomous System Provider Authorization (ASPA, [@!I-D.ietf-sidrops-aspa-profile]) where an autonomous system number
  (ASN, [@!RFC5396]) holder cryptographically asserts about the provider AS for that ASN.
* X.509 Resource Certificate ([@!RFC6487]) where the issuer grants the subject a right-of-use for the listed IP
  addresses and/or autonomous system numbers.

This document defines a new RDAP extension, "rpki1", for accessing the RPKI registration data within the Internet Number
Registry System (INRS) for aforementioned RPKI profiles through RDAP. The Internet Number Registry System (INRS) is
composed of Regional Internet Registries (RIRs), National Internet Registries (NIRs), and Local Internet Registries
(LIRs).

The motivation here is that such RDAP data could complement the existing RPKI diagnostic tools when troubleshooting a
route hijack or leak, by conveniently providing access to registration information from a registry's database beside
what is inherently available from an RPKI profile object. There is registration metadata that is often needed for
troubleshooting that does not appear in, say, a ROA or a VRP (Verified ROA Payload); such as:

* Is it an auto-renewing ROA or not?
* When did the initial version of a ROA get published?
* Was a ROA created in conjunction with an Internet Routing Registry (IRR, [@RFC2622]) route?
* Which IRR route is related with a ROA?
* Which IP network is associated with a ROA?

Furthermore, correlating registered RPKI data with registered IP networks and autonomous system numbers would also give
access to the latter's contact information through RDAP entity objects, which should aid troubleshooting.

In addition to troubleshooting, serving RPKI meta-data over RDAP offers a convenience to network operators
through a simple lookup mechanism. As is demonstrated in [@RDAP-GUIDE], constructing custom RDAP scripts is
relatively easy and beneficial to network operators for the purposes of reporting. Though not RDAP-based, systems such
as [@JDR] and [@CLOUDFLARE] have shown the utility of an approach that allows users to explore the RPKI hierarchy in a
visual fashion, without interacting with the signed objects directly.

For these purposes, this specification defines RDAP object classes, as well as lookup and search path segments, for the
ROA, ASPA, and X.509 resource certificate registration data.

## Requirements Language

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",
"NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [@!BCP14] when, and only
when, they appear in all capitals, as shown here.

Indentation and whitespace in examples are provided only to illustrate element relationships, and are not a REQUIRED
feature of this protocol.

"..." in examples is used as shorthand for elements defined outside of this document.

# Common Data Members {#common_data_members}

The RDAP object classes for RPKI ((#roa_object_class), (#aspa_object_class), (#x509_resource_cert_object_class)) can
contain one or more of the following common members:

* "handle" -- a string representing the registry unique identifier of an RPKI object registration
* "name" -- a string representing the identifier assigned to an RPKI object registration by the registration holder
* "notValidBefore" -- a string that contains the time and date in Zulu (Z) format with UTC offset of 00:00
  ([@!RFC3339]), representing the not-valid-before date of an X.509 resource certificate for an RPKI object
  ([@!RFC6487, section 4])
* "notValidAfter" -- a string that contains the time and date in Zulu (Z) format with UTC offset of 00:00 ([@!RFC3339]),
  representing the not-valid-after date of an X.509 resource certificate for an RPKI object ([@!RFC6487, section 4])
* "autoRenewed" -- a boolean indicating if a registered RPKI object is auto-renewed or not
* "publicationUri" -- a URI string pointing to the location of an RPKI object within an RPKI repository;
  the URI scheme is "rsync", per [@!RFC6487, section 4]
* "entities" -- an array of entity objects ([@!RFC9083, section 5.1]), including the organization (entity) registered as
  the authoritative source for an RPKI object
* "rpkiType" -- a string literal representing various combinations of an RPKI repository and a Certification Authority
  (CA), with the following possible values:
    * "hosted" -- both the repository and CA are operated by a registry for an organization with allocated resources
    * "delegated" -- both the repository and CA are operated by an organization with resources allocated by a registry
    * "hybrid" -- the repository is operated by a registry for an organization with allocated resources whereas the CA
      is operated by the organization itself

# Route Origin Authorization {#roa}

## Object Class {#roa_object_class}

The Route Origin Authorization (ROA) object class can contain the following members:

* "objectClassName" -- the string "rpki1_roa"
* "handle" -- see (#common_data_members)
* "name" -- see (#common_data_members)
* "roaIpAddresses" -- an array of objects representing CIDR address blocks within a ROA; such an object can contain the
  following members:
    * "startAddress" -- a string representing the start IP address (a.k.a. CIDR prefix) of a CIDR address block,
      either IPv4 or IPv6 ([@!RFC9582, section 4])
    * "prefixLength" -- a number representing the prefix length (a.k.a. CIDR length) of a CIDR address block; up to 32
      for IPv4 and up to 128 for IPv6 ([@!RFC9582, section 4])
    * "ipVersion" -- a string signifying the IP protocol version of a CIDR address block: "v4" for IPv4 and "v6" for
      IPv6 ([@!RFC9582, section 4])
    * "maxLength" -- a number representing the maximum prefix length of a CIDR address block that the origin AS is
      authorized to advertise; up to 32 for IPv4 and up to 128 for IPv6 ([@!RFC9582, section 4])
* "originAutnum" -- an unsigned 32-bit integer representing the origin autonomous system number ([@!RFC9582, section 4])
* "notValidBefore" -- see (#common_data_members)
* "notValidAfter" -- see (#common_data_members)
* "autoRenewed" -- see (#common_data_members)
* "publicationUri" -- see (#common_data_members)
* "entities" -- see (#common_data_members)
* "rpkiType" -- see (#common_data_members)
* "events" -- see [@!RFC9083, section 4.5]
* "links" -- "self" link, and "related" links for IP network and IRR (when defined) objects ([@!RFC9083, section 4.2])
* "remarks" -- see [@!RFC9083, section 4.3]

Here is an elided example of a ROA object:

```
{
  "objectClassName": "rpki1_roa",
  "handle": "XXXX",
  "name": "ROA-1",
  "roaIpAddresses":
  [
    {
      "startAddress": "2001:db8::",
      "prefixLength": 48,
      "ipVersion": "v6",
      "maxLength": 64
    },
    ...
  ],
  "originAutnum": 65536,
  "notValidBefore": "2024-04-27T23:59:59Z",
  "notValidAfter": "2025-04-27T23:59:59Z",
  "autoRenewed": true,
  "publicationUri": "rsync://example.net/path/to/XXXX.roa",
  "entities":
  [
    {
      "objectClassName": "entity",
      "handle": "XYZ-RIR",
      ...
    },
    ...
  ],
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

The resource type path segment for exact or closest match lookup of a ROA object is "rpki1/roa".

The following lookup path segments are defined for a ROA object:

Syntax: rpki1/roa/<handle>

Syntax: rpki1/roa/<IP address>

Syntax: rpki1/roa/<CIDR prefix>/<CIDR length>

A lookup query for ROA information by handle is specified using this form:

rpki1/roa/XXXX

XXXX is a string representing the "handle" property of a ROA, as described in (#roa_object_class). The following URL
would be used to find information for a ROA that exactly matches the "8a848ab0729f0f4f0173ba2013bc5eb3" handle:

```
https://example.net/rdap/rpki1/roa/8a848ab0729f0f4f0173ba2013bc5eb3
```

A lookup query for ROA information by IP address is specified using this form:

rpki1/roa/YYYY

YYYY is a string representing an IPv4 or IPv6 address. The following URL would be used to find information for a ROA
that completely encompasses the "192.0.2.0" IP address:

```
https://example.net/rdap/rpki1/roa/192.0.2.0
```

A lookup query for ROA information by CIDR is specified using this form:

rpki1/roa/YYYY/ZZZZ

YYYY is an IP address representing the "startAddress" property of a CIDR address block within a ROA and ZZZZ is a CIDR
length representing its "prefixLength" property, as described in (#roa_object_class). The following URL would be used to
find information for the most-specific ROA matching the "2001:db8::/64" CIDR:

```
https://example.net/rdap/rpki1/roa/2001%3Adb8%3A%3A/64
```

In the "links" array of a ROA object, the context URI ("value" member) of each link should be the lookup URL by its
handle, and if that's not available, then the lookup URL by one of its IP addresses.

## Search

The resource type path segment for searching ROA objects is "rpki1/roas".

The following search path segments are defined for ROA objects:

Syntax: rpki1/roas?name=<name search pattern>

Syntax: rpki1/roas?originAutnum=<autonomous system number>

Searches for ROA information by name are specified using this form:

rpki1/roas?name=XXXX

XXXX is a search pattern per [@!RFC9082, section 4.1], representing the "name" property of a ROA, as described in
(#roa_object_class). The following URL would be used to find information for ROA names matching the "ROA-*" pattern:

```
https://example.net/rdap/rpki1/roas?name=ROA-*
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
      "roaIpAddresses":
      [
        {
          "startAddress": "2001:db8::",
          "prefixLength": 48,
          "ipVersion": "v6",
          "maxLength": 64
        },
        ...
      ],
      "originAutnum": 65536,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": true,
      "publicationUri": "rsync://example.net/path/to/XXXX.roa",
      "entities":
      [
        {
          "objectClassName": "entity",
          "handle": "XYZ-RIR",
          ...
        },
        ...
      ],
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

(#reverse_search_registry) and (#reverse_search_mapping_registry) include registration of entries for IP network
searches in the RDAP Reverse Search and RDAP Reverse Search Mapping IANA registries when the related resource type is
"rpki1_roa".

## Relationship with IP Network Object Class

It would be useful to show all the ROAs associated with an IP network object. To that end, this extension adds a new
"rpki1_roas" member to the IP Network object class ([@!RFC9083, section 5.4]):

* "rpki1_roas" -- an array of ROA objects ((#roa_object_class)) associated with an IP network object; if the array is
  too large, the server MAY truncate it, per [@!RFC9083, section 9]

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
      "roaIpAddresses":
      [
        {
          "startAddress": "2001:db8::",
          "prefixLength": 48,
          "ipVersion": "v6",
          "maxLength": 64
        },
        ...
      ],
      "originAutnum": 65536,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": true,
      "publicationUri": "rsync://example.net/path/to/XXXX.roa",
      "entities":
      [
        {
          "objectClassName": "entity",
          "handle": "XYZ-RIR",
          ...
        },
        ...
      ],
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
      "roaIpAddresses":
      [
        {
          "startAddress": "2001:db8:1::",
          "prefixLength": 48,
          "ipVersion": "v6",
          "maxLength": 64
        },
        ...
      ],
      "originAutnum": 65537,
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": false,
      "publicationUri": "rsync://example.net/path/to/YYYY.roa",
      "entities":
      [
        {
          "objectClassName": "entity",
          "handle": "XYZ-RIR",
          ...
        },
        ...
      ],
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

The Autonomous System Provider Authorization (ASPA) object class can contain the following members:

* "objectClassName" -- the string "rpki1_aspa"
* "handle" -- see (#common_data_members)
* "name" -- see (#common_data_members)
* "autnum" -- an unsigned 32-bit integer representing an autonomous system number of the registration holder
  ([@!I-D.ietf-sidrops-aspa-profile, section 3])
* "providerAutnums" -- an array of unsigned 32-bit integers, each representing the autonomous system number of an AS
  that is authorized as a provider ([@!I-D.ietf-sidrops-aspa-profile, section 3])
* "notValidBefore" -- see (#common_data_members)
* "notValidAfter" -- see (#common_data_members)
* "autoRenewed" -- see (#common_data_members)
* "publicationUri" -- see (#common_data_members)
* "entities" -- see (#common_data_members)
* "rpkiType" -- see (#common_data_members)
* "events" -- see [@!RFC9083, section 4.5]
* "links" -- "self" link, and "related" links for autonomous system number and IRR (when defined) objects
  ([@!RFC9083, section 4.2])
* "remarks" -- see [@!RFC9083, section 4.3]

Here is an elided example of an ASPA object:

```
{
  "objectClassName": "rpki1_aspa",
  "handle": "XXXX",
  "name": "ASPA-1",
  "autnum": 65536,
  "providerAutnums":
  [
    65542,
    ...
  ],
  "notValidBefore": "2024-04-27T23:59:59Z",
  "notValidAfter": "2025-04-27T23:59:59Z",
  "autoRenewed": true,
  "publicationUri": "rsync://example.net/path/to/XXXX.aspa",
  "entities":
  [
    {
      "objectClassName": "entity",
      "handle": "XYZ-RIR",
      ...
    },
    ...
  ],
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

The following lookup path segments are defined for an ASPA object:

Syntax: rpki1/aspa/<handle>

Syntax: rpki1/aspa/<autonomous system number>

A lookup query for ASPA information by handle is specified using this form:

rpki1/aspa/XXXX

XXXX is a string representing the "handle" property of an ASPA, as described in (#aspa_object_class). The following URL
would be used to find information for an ASPA that exactly matches the "47ab80ed8693f25d0187d93a07db4484" handle:

```
https://example.net/rdap/rpki1/aspa/47ab80ed8693f25d0187d93a07db4484
```

A lookup query for ASPA information by autonomous system number is specified using this form:

rpki1/aspa/YYYY

YYYY is an autonomous system number representing the "autnum" property of an ASPA, as described in (#aspa_object_class).
The following URL would be used to find information for an ASPA with autonomous system number 65536:

```
https://example.net/rdap/rpki1/aspa/65536
```

In the "links" array of an ASPA object, the context URI ("value" member) of each link should be the lookup URL by its
handle, and if that's not available, then the lookup URL by its autonomous system number.

## Search

The resource type path segment for searching ASPA objects is "rpki1/aspas".

The following search path segments are defined for ASPA objects:

Syntax: rpki1/aspas?name=<name search pattern>

Syntax: rpki1/aspas?providerAutnum=<autonomous system number>

Searches for ASPA information by name are specified using this form:

rpki1/aspas?name=XXXX

XXXX is a search pattern per [@!RFC9082, section 4.1], representing the "name" property of an ASPA, as described in
(#aspa_object_class). The following URL would be used to find information for ASPA names matching the "ASPA-*" pattern:

```
https://example.net/rdap/rpki1/aspas?name=ASPA-*
```

Searches for ASPA information by provider autonomous system number are specified using this form:

rpki1/aspas?providerAutnum=YYYY

YYYY is an autonomous system number within the "providerAutnums" property of an ASPA, as described in
(#aspa_object_class). The following URL would be used to find information for ASPAs with provider autonomous system
number 65542:

```
https://example.net/rdap/rpki1/aspas?providerAutnum=65542
```

### Search Results

The ASPA search results are returned in the "rpki1_aspaSearchResults" member, which is an array of ASPA objects
((#aspa_object_class)).

Here is an elided example of the search results when finding information for ASPAs with provider autonomous system
number 65542:

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
      "providerAutnums":
      [
        65542,
        ...
      ],
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": true,
      "publicationUri": "rsync://example.net/path/to/XXXX.aspa",
      "entities":
      [
        {
          "objectClassName": "entity",
          "handle": "XYZ-RIR",
          ...
        },
        ...
      ],
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

(#reverse_search_registry) and (#reverse_search_mapping_registry) include registration of entries for autonomous system
number searches in the RDAP Reverse Search and RDAP Reverse Search Mapping IANA registries when the related resource
type is "rpki1_aspa".

## Relationship with Autonomous System Number Object Class

It would be useful to show all the ASPAs associated with an autonomous system number object. To that end, this extension
adds a new "rpki1_aspas" member to the Autonomous System Number object class ([@!RFC9083, section 5.5]):

* "rpki1_aspas" -- an array of ASPA objects ((#aspa_object_class)) for the autonomous system number range in the
  autonomous system number object; if the array is too large, the server MAY truncate it, per [@!RFC9083, section 9]

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
      "providerAutnums":
      [
        65542,
        ...
      ],
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": true,
      "publicationUri": "rsync://example.net/path/to/XXXX.aspa",
      "entities":
      [
        {
          "objectClassName": "entity",
          "handle": "XYZ-RIR",
          ...
        },
        ...
      ],
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
      "providerAutnums":
      [
        65543,
        ...
      ],
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "autoRenewed": false,
      "publicationUri": "rsync://example.net/path/to/YYYY.aspa",
      "entities":
      [
        {
          "objectClassName": "entity",
          "handle": "XYZ-RIR",
          ...
        },
        ...
      ],
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

# X.509 Resource Certificate {#x509_resource_cert}

## Object Class {#x509_resource_cert_object_class}

The X.509 resource certificate object class can contain the following members:

* "objectClassName" -- the string "rpki1_x509_resource_cert"
* "handle" -- see (#common_data_members)
* "serialNumber" -- a string representing the unique identifier for the certificate ([@!RFC6487, section 4.2])
* "issuer" -- a string representing the CA that issued the certificate ([@!RFC6487, section 4.4])
* "signatureAlgorithm" -- a string representing the algorithm used by the CA to sign the certificate
  ([@!RFC6487, section 4.3])
* "subject" -- a string representing the identity of the subject the certificate is issued to ([@!RFC6487, section 4.5])
* "subjectPublicKeyInfo" -- an object representing the subject's public key information ([@!RFC6487, section 4.7]), with
  the following members:
    * "publicKeyAlgorithm" -- a string representing the algorithm for the public key
    * "publicKey" -- a string representation of the public key
* "subjectKeyIdentifier" -- a string, typically Base64-encoded, representing the unique identifier for the public key
  ([@!RFC6487, section 4.8.2])
* "ips" -- an array of strings, each representing an IPv4 or IPv6 CIDR address block with the
  "<CIDR prefix>/<CIDR length>" format ([@!RFC6487, section 4.8.10])
* "autnums" -- an array of unsigned 32-bit integers, each representing an autonomous system number
  ([@!RFC6487, section 4.8.11])
* "notValidBefore" -- see (#common_data_members)
* "notValidAfter" -- see (#common_data_members)
* "autoRenewed" -- see (#common_data_members)
* "publicationUri" -- see (#common_data_members)
* "entities" -- see (#common_data_members)
* "rpkiType" -- see (#common_data_members)
* "events" -- see [@!RFC9083, section 4.5]
* "links" -- "self" link, and "related" links for IP network and/or autonomous system number objects
  ([@!RFC9083, section 4.2])
* "remarks" -- see [@!RFC9083, section 4.3]

Here is an elided example of an X.509 resource certificate object -- specifically, a BGPSec router certificate
([@!RFC8209]) where an ASN(s) holder cryptographically asserts that a router holding the corresponding private key is
authorized to emit secure route advertisements on behalf of the AS(es) specified in the certificate:

```
{
  "objectClassName": "rpki1_x509_resource_cert",
  "handle": "ABCD",
  "serialNumber": "1234",
  "issuer": "CN=ISP-CA",
  "signatureAlgorithm": "ecdsa-with-SHA256",
  "subject": "CN=BGPSEC-ROUTER",
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
  "entities":
  [
    {
      "objectClassName": "entity",
      "handle": "XYZ-RIR",
      ...
    },
    ...
  ],
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
      "value": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
      "rel": "self",
      "href": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
      "type": "application/rdap+json"
    },
    {
      "value": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
      "rel": "related",
      "href": "https://example.net/rdap/autnum/65536",
      "type": "application/rdap+json"
    },
    {
      "value": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
      "rel": "related",
      "href": "https://example.net/rdap/autnum/65537",
      "type": "application/rdap+json"
    },
    ...
  ],
  "remarks":
  [
    {
      "description": [ "An X.509 resource certificate object in RDAP" ]
    }
  ]
}
```

## Lookup

The resource type path segment for exact match lookup of an X.509 resource certificate object is
"rpki1/x509_resource_cert".

The following lookup path segment is defined for an X.509 resource certificate object:

Syntax: rpki1/x509_resource_cert/<handle>

For example:

```
https://example.net/rdap/rpki1/x509_resource_cert/ABCD
```

## Search

The resource type path segment for searching X.509 resource certificate objects is "rpki1/x509_resource_certs".

The following search path segments are defined for X.509 resource certificate objects:

Syntax: rpki1/x509_resource_certs?handle=<handle search pattern>

Syntax: rpki1/x509_resource_certs?issuer=<issuer search pattern>

Syntax: rpki1/x509_resource_certs?subject=<subject search pattern>

Syntax: rpki1/x509_resource_certs?subjectKeyIdentifier=<subject key identifier>

Syntax: rpki1/x509_resource_certs?ip=<IP address>

Syntax: rpki1/x509_resource_certs?cidr=<CIDR>

Syntax: rpki1/x509_resource_certs?autnum=<autonomous system number>

Searches for X.509 resource certificate information by handle are specified using this form:

rpki1/x509_resource_certs?handle=XXXX

XXXX is a search pattern per [@!RFC9082, section 4.1], representing the "handle" property of an X.509 resource
certificate object, as described in (#x509_resource_cert_object_class). The following URL would be used to find
information for X.509 resource certificate objects with handle matching the "ABC*" pattern:

```
https://example.net/rdap/rpki1/x509_resource_certs?handle=ABC*
```

Searches for X.509 resource certificate information by certificate issuer are specified using this form:

rpki1/x509_resource_certs?issuer=YYYY

YYYY is a search pattern per [@!RFC9082, section 4.1], representing the "issuer" property of an X.509 resource
certificate object, as described in (#x509_resource_cert_object_class). The following URL would be used to find
information for X.509 resource certificate objects with issuer matching the "CN=ISP-*" pattern:

```
https://example.net/rdap/rpki1/x509_resource_certs?issuer=CN%3DISP-*
```

Searches for X.509 resource certificate information by certificate subject are specified using this form:

rpki1/x509_resource_certs?subject=ZZZZ

ZZZZ is a search pattern per [@!RFC9082, section 4.1], representing the "subject" property of an X.509 resource
Certificate object, as described in (#x509_resource_cert_object_class). The following URL would be used to find
information for X.509 resource certificate objects with subject matching the "CN=BGPSEC-ROUTE*" pattern:

```
https://example.net/rdap/rpki1/x509_resource_certs?subject=CN%3DBGPSEC-ROUTE*
```

Searches for X.509 resource certificate information by subject key identifier are specified using this form:

rpki1/x509_resource_certs?subjectKeyIdentifier=BBBB

BBBB is a string representing the "subjectKeyIdentifier" property of an X.509 resource certificate object, as described
in (#x509_resource_cert_object_class). The following URL would be used to find an X.509 resource certificate object with
subject key identifier matching the "hOcGgxqXDa7mYv78fR+sGBKMtWJqItSLfaIYJDKYi8A=" string:

```
https://example.net/rdap/rpki1/x509_resource_certs?subjectKeyIdentifier=hOcGgxqXDa7mYv78fR+sGBKMtWJqItSLfaIYJDKYi8A=
```

Searches for X.509 resource certificate information by an IP address are specified using this form:

rpki1/x509_resource_certs?ip=CCCC

CCCC is a string representing an IPv4 or IPv6 address. The following URL would be used to find information for X.509
resource certificate objects with the "ips" member encompassing the "192.0.2.0" IP address:

```
https://example.net/rdap/rpki1/x509_resource_certs?ip=192.0.2.0
```

Searches for X.509 resource certificate information by a CIDR are specified using this form:

rpki1/x509_resource_certs?cidr=CCCC/DDDD

"CCCC/DDDD" is a string representing an IPv4 or IPv6 CIDR, with CCCC as the CIDR prefix and DDDD as the CIDR length. The
following URL would be used to find information for X.509 resource certificate objects with the "ips" member
encompassing the "2001:db8::/64" CIDR:

```
https://example.net/rdap/rpki1/x509_resource_certs?cidr=2001%3Adb8%3A%3A/64
```

Searches for X.509 resource certificate information by an autonomous system number are specified using this form:

rpki1/x509_resource_certs?autnum=EEEE

EEEE is an autonomous system number within the "autnums" property of an X.509 resource certificate object, as described
in (#x509_resource_cert_object_class). The following URL would be used to find information for X.509 resource
certificate objects with the "autnums" member including autonomous system number 65536:

```
https://example.net/rdap/rpki1/x509_resource_certs?autnum=65536
```

### Search Results

The X.509 resource certificate search results are returned in the "rpki1_x509ResourceCertSearchResults" member, which is
an array of X.509 resource certificate objects ((#x509_resource_cert_object_class)).

Here is an elided example of the search results when finding information for X.509 resource certificate objects with
issuer matching the "CN=ISP-*" pattern:

```
{
  "rdapConformance":
  [
    "rdap_level_0",
    "rpki1",
    ...
  ],
  ...
  "rpki1_x509ResourceCertSearchResults":
  [
    {
      "objectClassName": "rpki1_x509_resource_cert",
      "handle": "ABCD",
      "serialNumber": "1234",
      "issuer": "CN=ISP-CA",
      "signatureAlgorithm": "ecdsa-with-SHA256",
      "subject": "CN=BGPSEC-ROUTER",
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
      "entities":
      [
        {
          "objectClassName": "entity",
          "handle": "XYZ-RIR",
          ...
        },
        ...
      ],
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
          "value": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
          "rel": "self",
          "href": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
          "rel": "related",
          "href": "https://example.net/rdap/autnum/65536",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
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

Per [@!RFC9536, section 2], if a server receives a reverse search query with a searchable resource type of "ips"
([@!I-D.ietf-regext-rdap-rir-search, section 5]), a related resource type of "rpki1_x509_resource_cert", and an X.509
Resource Certificate property of "handle", then the reverse search will be performed on the IP network objects from its
data store.

Similarly, if a server receives a reverse search query with a searchable resource type of "autnums", a related resource
type of "rpki1_x509_resource_cert", and an X.509 Resource Certificate property of "handle", then the reverse search will
be performed on the autonomous system number objects.

(#reverse_search_registry) and (#reverse_search_mapping_registry) include registration of entries for IP network and
autonomous system number searches in the RDAP Reverse Search and RDAP Reverse Search Mapping IANA registries when the
related resource type is "rpki1_x509_resource_cert".

## Relationship with Other Object Classes

It would be useful to show all the X.509 resource certificates associated with an object of another RDAP class; in
particular, with an IP network object, an autonomous system number object, or an entity (organization) object. To that
end, this extension adds a new "rpki1_x509_resource_certs" member to the IP Network ([@!RFC9083, section 5.4]),
Autonomous System Number ([@!RFC9083, section 5.5]), and Entity ([@!RFC9083, section 5.1]) object classes:

* "rpki1_x509_resource_certs" -- an array of X.509 resource certificate objects ((#x509_resource_cert_object_class)) for
  the IP address range in an IP network object, the autonomous system number range in an autonomous system number
  object, or an entity (organization) object; if the array is too large, the server MAY truncate it, per
  [@!RFC9083, section 9]

Here is an elided example for an entity (organization) object with an X.509 resource certificate -- specifically, a CA
certificate that a registry issues to an organization for its allocated IP addresses and/or autonomous system numbers,
authorizing the organization CA to issue end-entity certificates:

```
{
  "objectClassName" : "entity",
  "handle":"XXXX",
  ...
  "rpki1_x509_resource_certs":
  [
    {
      "objectClassName": "rpki1_x509_resource_cert",
      "handle": "ABCD",
      "serialNumber": "1234",
      "issuer": "CN=RIR-CA",
      "signatureAlgorithm": "ecdsa-with-SHA256",
      "subject": "CN=XXXX-CA",
      "subjectPublicKeyInfo":
      {
        "publicKeyAlgorithm": "id-ecPublicKey",
        "publicKey": "..."
      },
      "subjectKeyIdentifier": "hOcGgxqXDa7mYv78fR+sGBKMtWJqItSLfaIYJDKYi8A=",
      "ips":
      [
        "192.0.2.0/24",
        "2001:db8::/48"
      ],
      "autnums":
      [
        65536,
        65537
      ],
      "notValidBefore": "2024-04-27T23:59:59Z",
      "notValidAfter": "2025-04-27T23:59:59Z",
      "publicationUri": "rsync://example.net/path/to/ABCD.cer",
      "entities":
      [
        {
          "objectClassName": "entity",
          "handle": "XYZ-RIR",
          ...
        },
        ...
      ],
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
          "value": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
          "rel": "self",
          "href": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
          "rel": "related",
          "href": "https://example.net/rdap/ip/192.0.2.0/24",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
          "rel": "related",
          "href": "https://example.net/rdap/ip/2001:db8::/48",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
          "rel": "related",
          "href": "https://example.net/rdap/autnum/65536",
          "type": "application/rdap+json"
        },
        {
          "value": "https://example.net/rdap/rpki1/x509_resource_cert/ABCD",
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

A server that supports the functionality specified in this document MUST include the "rpki1" string literal in the
"rdapConformance" array of its responses.

# Security Considerations

The RDAP extension in this document MUST NOT be used to directly influence internet routing. Neither RDAP nor this
extension define the necessary security properties or distribution mechanisms required to securely add, remove, or
modify internet routes.

This document does not introduce any new security considerations past those already discussed in the RDAP protocol
specifications ([@RFC7481], [@RFC9560]).

# IANA Considerations

## RDAP Extensions Registry

IANA is requested to register the following values in the RDAP Extensions Registry at
https://www.iana.org/assignments/rdap-extensions/:

* Extension identifier: rpki1
* Registry operator: Any
* Published specification: This document.
* Contact: IETF <iesg@ietf.org>
* Intended usage: This extension identifier is used for accessing the RPKI registration data through RDAP.

## RDAP Reverse Search Registry {#reverse_search_registry}

IANA is requested to register the following entries in the RDAP Reverse Search Registry at
https://www.iana.org/assignments/rdap-reverse-search/:

IP network search by the origin autonomous system number of a ROA:

* Searchable Resource Type: ips
* Related Resource Type: rpki1_roa
* Property: originAutnum
* Description: The server supports the IP network search by the origin autonomous system number of an associated RPKI
  ROA.
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

IP network search by the start IP address of a CIDR address block of a ROA:

* Searchable Resource Type: ips
* Related Resource Type: rpki1_roa
* Property: startAddress
* Description: The server supports the IP network search by the start IP address (a.k.a. CIDR prefix) of a CIDR address
  block of an associated RPKI ROA.
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

Autonomous system number search by the autonomous system number of an ASPA:

* Searchable Resource Type: autnums
* Related Resource Type: rpki1_aspa
* Property: autnum
* Description: The server supports the autonomous system number search by the autonomous system number of an associated
  RPKI ASPA.
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

Autonomous system number search by a provider autonomous system number of an ASPA:

* Searchable Resource Type: autnums
* Related Resource Type: rpki1_aspa
* Property: providerAutnum
* Description: The server supports the autonomous system number search by a provider autonomous system number of an
  associated RPKI ASPA.
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

IP network search by the handle of an X.509 resource certificate:

* Searchable Resource Type: ips
* Related Resource Type: rpki1_x509_resource_cert
* Property: handle
* Description: The server supports the IP network search by the handle of an associated RPKI X.509 resource certificate.
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

Autonomous system number search by the handle of an X.509 resource certificate:

* Searchable Resource Type: autnums
* Related Resource Type: rpki1_x509_resource_cert
* Property: handle
* Description: The server supports the autonomous system number search by the handle of an associated RPKI X.509
  resource certificate.
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

## RDAP Reverse Search Mapping Registry {#reverse_search_mapping_registry}

IANA is requested to register the following entries in the RDAP Reverse Search Mapping Registry at
https://www.iana.org/assignments/rdap-reverse-search-mapping/:

IP network search by the origin autonomous system number of a ROA:

* Searchable Resource Type: ips
* Related Resource Type: rpki1_roa
* Property: originAutnum
* Property Path: $.originAutnum
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

IP network search by the start IP address of a CIDR address block of a ROA:

* Searchable Resource Type: ips
* Related Resource Type: rpki1_roa
* Property: startAddress
* Property Path: $.roaIpAddresses[*].startAddress
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

Autonomous system number search by the autonomous system number of an ASPA:

* Searchable Resource Type: autnums
* Related Resource Type: rpki1_aspa
* Property: autnum
* Property Path: $.autnum
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

Autonomous system number search by a provider autonomous system number of an ASPA:

* Searchable Resource Type: autnums
* Related Resource Type: rpki1_aspa
* Property: providerAutnum
* Property Path: $.providerAutnums[*]
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

IP network search by the handle of an X.509 resource certificate:

* Searchable Resource Type: ips
* Related Resource Type: rpki1_x509_resource_cert
* Property: handle
* Property Path: $.handle
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

Autonomous system number search by the handle of an X.509 resource certificate:

* Searchable Resource Type: autnums
* Related Resource Type: rpki1_x509_resource_cert
* Property: handle
* Property Path: $.handle
* Registrant Name: IETF
* Registrant Contact Information: iesg@ietf.org
* Reference: This document.

# Acknowledgements

Job Snijders, Ties de Kock, Mark Kosters, Tim Bruijnzeels, and Bart Bakker provided valuable feedback for this document.

{backmatter}

<reference anchor='RDAP-GUIDE' target='https://rdap.rcode3.com/misc/uses.html'>
    <front>
        <title>RDAP Guide</title>
        <author>
            <organization>Newton, A.</organization>
        </author>
        <date year='2024'/>
    </front>
</reference>

<reference anchor='JDR' target='https://blog.nlnetlabs.nl/introducing-jdr/'>
    <front>
        <title>JDR</title>
        <author>
            <organization>NLNet Labs</organization>
        </author>
        <date year='2020'/>
    </front>
</reference>

<reference anchor='CLOUDFLARE' target='https://rpki.cloudflare.com/'>
    <front>
        <title>RPKI Portal</title>
        <author>
            <organization>Cloudflare</organization>
        </author>
        <date year='2022'/>
    </front>
</reference>
