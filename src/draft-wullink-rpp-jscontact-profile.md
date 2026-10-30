%%%
title = "JSContact Profile for RESTful Provisioning Protocol (RPP)"
abbrev = "JSContact Profile for RPP"
area = "Internet"
workgroup = "Network Working Group"
submissiontype = "IETF"
keyword = [""]
TocDepth = 4
date = 2026-07-06

[seriesInfo]
name = "Internet-Draft"
value = "draft-wullink-rpp-jscontact-profile-01 (editorial)"
stream = "IETF"
status = "standard"

[[author]]
initials="M."
surname="Wullink"
fullname="Maarten Wullink"
abbrev = ""
organization = "SIDN Labs"
  [author.address]
  email = "maarten.wullink@sidn.nl"
  uri = "https://sidn.nl/"

[[author]]
initials="P."
surname="Kowalik"
fullname="Pawel Kowalik"
abbrev = ""
organization = "DENIC"
  [author.address]
  email = "pawel.kowalik@denic.de"
  uri = "https://denic.de/"

%%%

.# Abstract

This document defines a JSContact usage profile, conforming to the rules described in [@!RFC10050], for the RESTful Provisioning Protocol (RPP). The JSContact Profile for RPP specifies how the JSContact format defined in [@!RFC9982] can be used as a standardized representation of contact information within RPP operations, enabling interoperability and consistency across RPP implementations that manage contact data.

{mainmatter}

# Introduction

JSContact [@!RFC9982] is a flexible, JSON-based format for representing contact
information. Its flexibility allows the same data to be expressed in multiple
valid ways, which can hinder interoperability in a provisioning context where
data must be stored and interpreted consistently.

This document defines a JSContact usage profile for the RESTful Provisioning
Protocol (RPP), restricting the set of JSContact constructs that RPP clients and
servers are required to support when provisioning contact information.

## Relationship to the RDAP JSContact Profile

The Registration Data Access Protocol (RDAP) [@!RFC9083] provides read-only
access to registration data managed by RPP. Both protocols use JSContact for
contact representation, but with profiles suited to their respective roles.

The RDAP JSContact profile [@I-D.ietf-regext-rdap-jscontact] defines the constructs used when serving registration data to read-only consumers. The RPP profile, defined in this document, is a subset of these constructs. All JSContact constructs permitted by the RPP profile can be used for RDAP.

The converse does not hold. RDAP supports additional JSContact constructs that are useful in the read-only context of RDAP responses but that may hinder interoperability. These constructs are not present in RPP requests and responses.

# Terminology

In this document, the following terminology is used.

RESTful Provisioning Protocol or RPP - The protocol described in this document.

URL - A Uniform Resource Locator as defined in [@!RFC3986].

RPP client - An HTTP user agent performing an RPP request.

RPP server - An HTTP server responsible for processing requests and returning results in any supported media type.

Internationalized form - The representation of contact information in the Card members outside the `"localizations"` map.

Localized form - A representation of contact information placed in the `"localizations"` map, keyed by a language tag.

# Conventions Used in This Document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT","SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [@!RFC2119].

# Using JSContact in RPP

## JSContact Profile for RPP

Since JSContact is a generalized representation of contact data, many of its
capabilities are not optimized for interoperable use in RPP payloads. This
document therefore defines a usage profile for JSContact that simplifies
implementation for both RPP clients and RPP servers.

The JSContact profile for RPP is compliant with the rules described in
[@!RFC10050]. The profile properties are formally
listed in (#profile-properties). All types mentioned in this section refer to
the JSContact specification [@!RFC9982].

### Version {#version}

The Card `"version"` member value MUST be `"2.0"`.

### Kind {#kind}

If present, the Card `"kind"` member value MUST be `"individual"` (default) or
`"org"` to represent an individual or an organization, respectively.

### Language {#language}

The Card `"language"` member SHOULD be set when localizations are specified
(i.e., when the `"localizations"` member is present and non-empty).

### Name {#name}

Other than the optional `"@type"` member, the Name type MUST include only the `"full"` member.

### Organizations {#organizations}

Other than the optional `"@type"` member, the Organization type MUST include only the `"name"` member.

### Addresses {#addresses}

Other than the optional `"@type"` member, the Address type MUST include only the `"components"` and `"countryCode"` members.

Other than the optional `"@type"` member, the AddressComponent type MUST include only the `"kind"` and `"value"` members. The `"kind"` member value MUST be one of `"name"`, `"locality"`, `"region"`, or `"postcode"`.

The Address type MUST include the `"countryCode"` member. The `"components"` member MUST contain exactly one `"locality"` component, at most one `"region"` component, and at most one `"postcode"` component.

The `"name"` components represent street lines. The server MAY constraint the maximum number of `"name"` components supported. Their order in the `"components"` array is significant and MUST be preserved.

When both an internationalized and a localized version of an address are used, the localized version MUST be placed in the `"localizations"` map keyed by an appropriate [@!BCP47] language tag.

### Emails {#emails}

Other than the optional `"@type"` member, the EmailAddress type MUST include only the `"address"` member.

### Phones {#phones}

The Phone type MUST include the `"number"` member and MAY include the
`"features"` member. If present, the `"features"` member MUST contain only the
`"voice"` or `"fax"` keys, and the value associated with each key MUST be
`true`. When the `"features"` member is absent, the phone number is assumed to
be a voice number.

### Links {#links}

The Link type MUST include the "uri" member and MAY include the "kind" member.
When the "kind" member is present, its value MUST be "contact".

<!-- do we need uri links for contacts? See issue #4 -->

### Map Keys {#map-keys}

A JSContact map key MUST comply with the Id type definition in [@!RFC9553]. RPP implementations MUST use the following predefined keys:

- "org" in the "organizations" map for a single preferred organization.
- "addr" in the "addresses" map for a single preferred postal address.
- "email" in the "emails" map for the preferred email address.
<!-- Allow for multiple email/voice/fax numbers? See issue #5 -->
- "voice" in the "phones" map for the preferred voice number.
<!-- Allow for multiple email/voice/fax numbers? See issue #5 -->
- "fax" in the "phones" map for the preferred fax number.
<!-- Allow for multiple email/voice/fax numbers? See issue #5 -->
- "url" in the "links" map for a preferred contact URL; the "kind" member of
  the Link object MUST NOT be set.
- "contact-uri" in the "links" map for a contact URI as defined in [@RFC8605];
  the "kind" member of the Link object MUST be set to "contact".

When internationalized and localized versions of an organization, postal address, or email exist, the internationalized version MUST be included under
the predefined map key, while the localized version MUST be placed in the "localizations" map as described in (#localizations).

### Localizations {#localizations}

If present, localized variants of name, organization, postal address, and email MUST be added to the "localizations" map. RPP implementations MUST
expand all localizations, meaning a nested PatchObject key of the form "{key1}/{key2}/.../{keyN}" MUST NOT be used.

The internationalized form MUST contain only characters that can be represented in the 7-bit US-ASCII character set [@!RFC20]. This applies to the `"full"` member of Name, the `"name"` member of Organization, the `"value"` member of AddressComponent, and the `"address"` member of EmailAddress. A localized form MAY contain any character permitted by [@!RFC9982]. When contact information is available only in a form containing non-ASCII characters, the RPP client MUST provide a US-ASCII transliteration as the internationalized form. An RPP server MUST reject a Card whose internationalized form contains characters outside the US-ASCII character set.

An RPP server MUST support at least one localization per Card. An RPP server MAY support more localizations, as determined by server policy. An RPP server MUST reject a Card that contains more localizations than it supports. The language of each localization MUST be indicated by a [@!BCP47] language tag as the key in the "localizations" map.

The following example of a Card includes a localized version of the postal address:

~~~ json
{
  "@type": "Card",
  "version": "2.0",
  "language": "en",
  "name": {
    "full": "Joe User"
  },
  "organizations": {
    "org": { "name": "Example Org" }
  },
  "addresses": {
    "addr": {
      "components": [
        { "kind": "name",     "value": "Main Street 1" },
        { "kind": "locality", "value": "Ludwigshafen am Rhein" },
        { "kind": "region",   "value": "Rhineland-Palatinate" },
        { "kind": "postcode", "value": "67067" }
      ],
      "countryCode": "DE"
    }
  },
  "localizations": {
    "de": {
      "addresses": {
        "addr": {
          "components": [
            { "kind": "name",     "value": "Hauptstraße 1" },
            { "kind": "locality", "value": "Ludwigshafen am Rhein" },
            { "kind": "region",   "value": "Rheinland-Pfalz" },
            { "kind": "postcode", "value": "67067" }
          ],
          "countryCode": "DE"
        }
      }
    }
  }
}
~~~
Figure: Example of handling localizations in JSContact

### Profile Properties {#profile-properties}

The following entries define the properties of the JSContact profile for RPP
registered in the JSContact Profile registry as described in
(#jscontact-profile-registry). The field names are those defined in
[@!RFC10050]. Entry fields with empty values are omitted:

<!-- TODO: update Restricted Attributes, for RPP a lot more properties are restricted than in the list below -->

- Property Name: language\
  Property Context: Card

- Property Name: kind\
  Property Context: Card\
  Restricted Enum Values: individual, org

- Property Name: name\
  Property Context: Card

- Property Name: full\
  Property Context: Name\
  Restricted Attributes: mandatory

- Property Name: organizations\
  Property Context: Card

- Property Name: name\
  Property Context: Organization\
  Restricted Attributes: mandatory

- Property Name: addresses\
  Property Context: Card

- Property Name: countryCode\
  Property Context: Address\
  Restricted Attributes: mandatory

- Property Name: components\
  Property Context: Address

- Property Name: kind\
  Property Context: AddressComponent\
  Restricted Enum Values: name, locality, region, postcode

- Property Name: value\
  Property Context: AddressComponent

- Property Name: emails\
  Property Context: Card

- Property Name: address\
  Property Context: EmailAddress

- Property Name: phones\
  Property Context: Card

- Property Name: features\
  Property Context: Phone\
  Restricted Enum Values: voice, fax

- Property Name: number\
  Property Context: Phone

- Property Name: links\
  Property Context: Card

- Property Name: kind\
  Property Context: Link\
  Restricted Enum Values: contact

- Property Name: uri\
  Property Context: Link

- Property Name: localizations\
  Property Context: Card\
  Restricted PatchObject: yes

The `"@type"` member is permitted on every JSContact object in this profile and
is therefore not listed above.

Client validation MUST return an error if any JSContact properties other than those listed above are present.

# IANA Considerations

## JSContact Profile Registry {#jscontact-profile-registry}

IANA is requested to register the following entry in the "JSContact Profile" registry [@!RFC10050]:

Name: rpp\
Version: 1.0\
Reference: (#profile-properties) of this document

TODO

# Internationalization Considerations

TODO

# Security Considerations

RPP servers should impose appropriate limits on the size and complexity of
JSContact objects received from RPP clients, including the number and size of
expanded localizations.

TODO

# Change History

## Version 00 to 01

- Fixed duplicate text ( Issue #14)
- Aligned with draft-ietf-regext-rdap-jscontact-27
- Allowed the Address `"countryCode"` member and removed Address `"full"` from the
  profile properties (Issue #19)
- Restricted the internationalized form to US-ASCII; required RPP servers to
  support at least one localization, with additional localizations subject to
  server policy (Issue #6)
- Made the Address `"countryCode"` member mandatory, limited the number of each
  AddressComponent kind, and made the order of street lines significant


{backmatter}

{numbered="false"}
# Acknowledgements

TODO
