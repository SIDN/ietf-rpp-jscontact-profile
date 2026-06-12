%%%
title = "JSContact Profile for RESTful Provisioning Protocol (RPP)"
abbrev = "JSContact Profile for RPP"
area = "Internet"
workgroup = "Network Working Group"
submissiontype = "IETF"
keyword = [""]
TocDepth = 4
date = 2026-04-30

[seriesInfo]
name = "Internet-Draft"
value = "draft-wullink-rpp-jscontact-profile-00"
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

This document defines the JSContact Profile as defined in [@!I-D.ietf-calext-jscontact-profiles] for the [RESTful Provisioning Protocol](https://datatracker.ietf.org/wg/rpp/about/) (RPP). The JSContact Profile for RPP specifies how the JSContact format defined in [@!RFC9982] can be used as a standardized representation of contact information within RPP operations, enabling interoperability and consistency across RPP implementations that manage contact data.

{mainmatter}

# Introduction

TODO: write introduction, motivation, scope, etc.

# Terminology

In this document the following terminology is used.

RESTful Provisioning Protocol or RPP - The protocol described in this document.

URL - A Uniform Resource Locator as defined in [@!RFC3986].

RPP client - An HTTP user agent performing an RPP request

RPP server - An HTTP server responsible for processing requests and returning results in any supported media type.

# Conventions Used in This Document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT","SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [@!RFC2119].

# Using JSContact in RPP

## JSContact Profile for RPP

Since JSContact is a generalized representation of contact data, many of its
capabilities are not optimized for interoperable use in RPP payloads. This
document therefore defines a usage profile for JSContact that simplifies
implementation for both RPP clients and RPP servers.

The JSContact profile for RPP is compliant with the rules described in
[@!I-D.ietf-calext-jscontact-profiles]. The profile properties are formally
listed in (#profile-properties). All types mentioned in this section refer to
the JSContact specification [@!RFC9982].

### Version {#version}

The Card "version" member value MUST be "2.0".

### Kind {#kind}

The Card "kind" member value MUST be "individual" (default) or "org" to
represent an individual or an organization, respectively.

### Language {#language}

The Card "language" member SHOULD be set when localizations are specified
(i.e., when the "localizations" member is present and non-null).

### Name {#name}

The Card "name" member MUST include the "full" member and MAY include the
"components" member.

When present, each NameComponent MUST include only the "kind" and "value"
members. The "kind" member value MUST be "title", "given" or "surname".

A> PK> I think we should remove MAY for "components". In EPP there is only name and any optionality makes implementations difficult.

### Organizations {#organizations}

The Organization type MUST include only the "org" member.

### Addresses {#addresses}

The Address type MUST include at least one of the "full", "components", or
"countryCode" members.

A> PK> also here. EPP has address broken down. Don't use full and only use components.

When present, each AddressComponent MUST include only the "kind" and "value"
members. The "kind" member value MUST be "name", "locality", "region",
"postcode", or "country".

A> PK> here do we need "country" if we have "countryCode" above?

When both an internationalised (`int`) and a localised (`loc`) version of postal address data exist (EPP Compatibility Profile), the internationalised version MUST be represented as the main Card properties, and the localised version MUST be placed in the `localizations` map keyed by an appropriate BCP 47 language tag.

### Emails {#emails}

The EmailAddress type MUST include only the "address" member.

### Phones {#phones}

The Phone type MUST include the "number" member and MAY include the "features"
member. When the "features" member is present, its values MUST be "voice" or
"fax". When the "features" member is absent, the phone number is assumed to be
a voice number.

### Links {#links}

The Link type MUST include the "uri" member and MAY include the "kind" member.
When the "kind" member is present, its value MUST be "contact".

A> PK> why do we need uri links for contacts?
<!--TODO: allow other link kinds such as company-website? -->

### Map Keys {#map-keys}

Since most JSContact collections are represented as maps, map keys must be
defined. A JSContact map key MUST comply with the Id type definition in
[@!RFC9982]. To aid interoperability, RPP implementations SHOULD use the
following predefined keys:

- "org" in the "organizations" map for a single preferred organization.
- "addr" in the "addresses" map for a single preferred postal address.
- "email" in the "emails" map for the preferred email address.
A> PK> We need to cover for multiple E-mails. I think RDAP profile has some definition
- "voice" in the "phones" map for the preferred voice number.
A> PK> We need to cover for multiple voice numbers. I think RDAP profile has some definition
- "fax" in the "phones" map for the preferred fax number.
A> PK> We need to cover for multiple fax numbers. I think RDAP profile has some definition
- "url" in the "links" map for a preferred contact URL; the "kind" member of
  the Link object MUST NOT be set.
- "contact-uri" in the "links" map for a contact URI; the "kind" member of
  the Link object MUST be set to "contact".

When internationalized and localized versions of an organization, postal
address, or email exist, the internationalized version MUST be included under
the predefined map key, while the localized version MUST be placed in the
"localizations" map as described in (#localizations).


### Localizations {#localizations}

A RPP request MUST NOT include a "localizations" member, as RPP clients are not expected to provide localized contact information. However,
A RPP response MAY include a "localizations" member to provide localized versions of the contact information.

A> PK> create/update requests must be able to include localisations, otherwise how to provision them?

If present, localized variants of name, organization, postal address, and
email MUST be added to the "localizations" map. RPP implementations MUST
expand all localizations, meaning a nested PatchObject key of the form
"{key1}/{key2}/.../{keyN}" MUST NOT be used.

A> PK> For EPP compatibility at most one localisation can be present

The following is an elided example of a Card including a localized version of
the postal address:

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
        { "kind": "postcode", "value": "67067" },
        { "kind": "country",  "value": "Germany" }
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
            { "kind": "postcode", "value": "67067" },
            { "kind": "country",  "value": "Deutschland" }
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

The properties of the JSContact profile for RPP registered in the JSContact
Profile registry as described in (#jscontact-profile-registry) are:

<!-- TODO: update Restricted Attributes, for RPP a lot more properties are restricted than in the table below -->

| Property    | Context          | Restricted  | Restricted Enum                          |
|             |                  | Attributes  | Values                                   |
|:------------|:-----------------|:------------|:-----------------------------------------|
| language    | Card             |             |                                          |
| kind        | Card             |             | individual, org                          |
| name        | Card             |             |                                          |
| full        | Name             | mandatory   |                                          |
| components  | Name             |             |                                          |
| kind        | NameComponent    |             | title, given, surname                    |
| value       | NameComponent    |             |                                          |
| organizations | Card           |             |                                          |
| name        | Organization     |             |                                          |
| addresses   | Card             |             |                                          |
| full        | Address          |             |                                          |
| countryCode | Address          |             |                                          |
| components  | Address          |             |                                          |
| kind        | AddressComponent |             | name, locality, region, postcode, country|
| value       | AddressComponent |             |                                          |
| emails      | Card             |             |                                          |
| address     | EmailAddress     |             |                                          |
| phones      | Card             |             |                                          |
| features    | Phone            |             | voice, fax                               |
| number      | Phone            |             |                                          |
| links       | Card             |             |                                          |
| kind        | Link             |             | contact                                  |
| uri         | Link             |             |                                          |
| localizations | Card           |             | yes                                      |

Client validation MUST return an error if any JSContact properties other than those listed above are present.

# IANA Considerations

## JSContact Profile Registry

IANA is requested to register the following entry in the "JSContact Profile" registry [I-D.ietf-calext-jscontact-profiles]:

Name:
rpp
Profile Version: 1.0
Reference: TODO

TODO

# Internationalization Considerations

TODO

# Security Considerations

TODO

# Change History

## Version 00

- Created initial document and added boilerplate sections.

{backmatter}

{numbered="false"}
# Acknowledgements

TODO
