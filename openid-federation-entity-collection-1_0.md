%%%
title = "OpenID Federation Entity Collection Endpoint 1.0 - draft 00"
abbrev = "openid-federation-entity-collection"
ipr = "none"
workgroup = "individual"
keyword = ["security", "openid", "federation"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-federation-entity-collection"
status = "standard"

[[author]]
initials="G."
surname="Zachmann"
fullname="Gabriel Zachmann"
organization="Karlsruhe Institute of Technology"
    [author.address]
    email = "gabriel.zachmann@kit.edu"
    
%%%

.# Abstract

This specification acts as an extension to [@!OpenID.Federation]. It defines an additional federation endpoint to retrieve a filterable list of Entities in a (sub-)federation.

{mainmatter}

# Introduction

This specification introduces a new federation endpoint that provides a
filterable collection of Entities subordinate -- direct or indirect -- to a
Trust Anchor. In contrast to the Subordinate Listing Endpoint of [@!OpenID.Federation],
the entity collection does not only include direct subordinates.
The response includes informational claims about the entities that MAY be used
to build user interfaces to select one or multiple Entities, for example a login
screen where End Users can select an OpenID Provider to login with.

## Requirements Notation and Conventions

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT
RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP
14 [@!RFC2119] [@!RFC8174] when, and only when, they appear in all capitals, as shown here.


# Terminology

This specification uses the terms
"Entity" as defined by OpenID Connect Core [@!OpenID.Core],
"Client" as defined by [@!RFC6749],
and "Trust Mark", "Federation Entity", "Federation Entity Key", "Trust Anchor",
"Intermediate", and "Subordinate Statement" defined in [@!OpenID.Federation].

## Endpoint Description

The Federation Entity Collection Endpoint is an optional endpoint that MAY be published by Federation Entities. It MUST use the `https` scheme and MAY include port, path, and query parameter components encoded in `application/x-www-form-urlencoded` format. It MUST NOT contain a fragment component.
Federation Entities publishing this endpoint SHOULD also publish a
`federation_resolve_endpoint`.

### Endpoint Location

The location of the Federation Entity Collection Endpoint is published in the `federation_entity` metadata, using the `federation_collection_endpoint` parameter.

The following is a non-normative example of an Entity Configuration payload, for a Trust Anchor that includes the `federation_collection_endpoint`:

```json
{
  "iss": "https://ta.example.org",
  "sub": "https://ta.example.org",
  "iat": 1590000000,
  "exp": 1590086400,
  "jwks": {
    "keys": [
      {
        "kty": "RSA",
        "kid": "key1",
        "use": "sig",
        "n": "n4EPtAOCc9AlkeQHPzHStgAbgs7bTZLwUBZdR8_KuKPEHLd4rHVTeT",
        "e": "AQAB"
      }
    ]
  },
  "metadata": {
    "federation_entity": {
      "federation_fetch_endpoint": "https://ta.example.org/fetch",
      "federation_list_endpoint": "https://ta.example.org/list",
      "federation_resolve_endpoint": "https://ta.example.org/resolve",
      "federation_collection_endpoint": "https://ta.example.org/entities"
    }
  }
}
```

## Entity Collection Request

### Request Format

When client authentication is not used, the request to the `federation_collection_endpoint` MUST be an HTTP request using the GET method with the following query parameters, encoded in `application/x-www-form-urlencoded` format:

- **from_entity_id**: (OPTIONAL) If this parameter is present, the resulting list MUST be the subset of the overall ordered response starting from the index of the Entity referenced with this parameter.   
  If the Entity Identifier in this parameter is not or not longer known to the responder, it MUST use the HTTP status code 404 and the content type `application/json` with the error code `entity_id_not_found`.  
  If the responder does not support this feature, it MUST use the HTTP status code 400 and the content type `application/json`, with the error code `unsupported_parameter`.

- **limit**: (OPTIONAL) Requested number of results included in the response.
If this parameter is present, the number of results in the returned list MUST NOT be greater than the minimum of the responder’s upper limit and the value of this parameter.
If this parameter is not present the server MUST fall back on the upper limit.  
  If the responder does not support this feature, it MUST use the HTTP status code 400 and the content type `application/json`, with the error code `unsupported_parameter`.
  
- **entity_type**: (OPTIONAL) The value of this parameter is an Entity Type Identifier. The result MUST be filtered to include only those entities that include the specified Entity Type. When multiple `entity_type` parameters are present, for example `entity_type=openid_provider&entity_type=openid_relying_party`, the result MUST be filtered to include all Entities that include any of the specified Entity Types. 
If the responder does not support this feature, it MUST use the HTTP status code 400 and the content type `application/json`, with the error code `unsupported_parameter`.

- **trust_mark_type**: (OPTIONAL) The value of this parameter is a Trust Mark Type Identifier. The result MUST be filtered to include only Entities that publish a Trust Mark of this Trust Mark Type in their Entity Configuration and that Trust Mark MUST be verified by the responder. The responder SHOULD verify the Trust Mark using the same Trust Anchor that is used to collect the Entities. When multiple `trust_mark_type` parameters are present, the result MUST be filtered to include only Entities that have a Trust Mark for all the specified Trust Mark Types.  
If the responder does not support this feature, it MUST use the HTTP status code 400 and set the content type to `application/json`, with the error code `unsupported_parameter`.

- **trust_anchor**: (RECOMMENDED) The Trust Anchor that the collection endpoint MUST use when collecting Entities. The value is an Entity Identifier. If omitted, the responder sets this parameter to its own Entity Identifier. If the responder does not have a defined Entity Identifier, it MUST use the HTTP status code 400 and set the content type to `application/json`, with the error code `invalid_request`.

- **query**: (OPTIONAL) The value of this parameter is used by the responder to
filter down the list of returned Entities to only entities that match this
parameter value. It is entirely up to the responder to define when an Entity
matches the query.  
If the responder does not support this feature, it SHOULD use the HTTP status code 400 and the content type `application/json`, with the error code `unsupported_parameter`.

-	**entity_claims**: (OPTIONAL) Array of claims to be included in the Entity Info Object included in the response for each collected Entity.  
If this parameter is NOT present it is at the discretion of the responder which claims are included or not.  
If this parameter is present and it is NOT an empty array, each Entity Info Object that represents an Entity MUST include the requested claims unless a specific claim is not available for that Entity. Also Claims that are optional to return and not present in the array MUST NOT be included in the Entity Info.  
If the responder does not support a requested claim, it MUST use the HTTP status code 400 and set the content type to `application/json`, with the error code `unsupported_parameter`.

-	**ui_claims**: (OPTIONAL) Array of claims to be included in the Entity Type UI Info Object included in the response for each returned Entity.  
If this parameter is NOT present it is at the discretion of the responder which claims are included or not.  
If this parameter is present and it is NOT an empty array, each Entity Type UI Info Object MUST include the requested claims unless a specific claim is not available for that Entity and Entity Type.  
If the responder does not support a requested claim, it MUST use the HTTP status code 400 and set the content type to `application/json`, with the error code `unsupported_parameter`.



When Client authentication is used, the request MUST be an HTTP request using the POST method, with the parameters passed in the POST body.

#### Example Request

The following is a non-normative example of a collection request:

```http
GET /collection?entity_type=openid_provider&trust_mark_type=https%3A%2F%2Frp%2Erefeds.org%2Fsitfi&trust_anchor=https%3A%2F%2Fswamid.se HTTP/1.1
Host: openid.sunet.se
```

## Entity Collection Response

### Response Format

A successful response MUST use the HTTP status code 200 and the content type `application/json`. 

The response is a JSON object as described below.

If the response is negative, it will be a JSON object and the content type MUST be `application/json` and use the errors defined here or in [@!OpenID.Federation].

### Response Claims

The claims in the entity collection response are:

- **entities**: (REQUIRED) Array of JSON objects, each representing a
Federation Entity as described in [Entity Info](#entity-info). The list of Entities MUST
only contain Entities that are in line with the requested parameters. The
responder MAY also filter down the list further at its own discretion.
- **next_entity_id**: (OPTIONAL) Entity Identifier for the next element in the
result list where the next page begins. This attribute is REQUIRED when
additional results are available beyond those included in the `entities` array.
- **last_updated**: (RECOMMENDED) Number. Time when the responder last updated the result list. This is expressed as Seconds Since the Epoch, per [@!RFC7519]. If the `last_updated` time changes between paginated calls, this might be an indication for the client that it might have received outdated information in a previous call. 

Additional claims MAY be defined and used in conjunction with the claims above.

#### Entity Info

Each JSON Object in the returned `entities` array MAY contain the following
claims:

- **entity_id**: (REQUIRED) The Entity Identifier for the subject entity of the
current record.
- **entity_types**: (RECOMMENDED) Array of string Entity Type Identifiers. If
present this claim MUST contain all Entity Type Identifiers of the subject's
Entity the responder knows about.
- **ui_infos**: (OPTIONAL) JSON Object containing information intended to be displayed to the user for each entity
type as described in [UI Infos](#ui-infos).  
If the request contains the `entity_type` parameter, the UI Infos Object MUST
only contain Entity Type Identifiers that are among the ones requested, with the exception
of the `federation_entity` Entity Type Identifier, which MAY also appear if not explicitly requested.  

- **trust_marks**: (OPTIONAL) Array of objects, each representing a Trust Mark,
as defined in Section 3 of [@!OpenID.Federation].

Additional claims MAY be defined and used in conjunction with the claims above.

##### UI Infos

UI Infos is a JSON Object containing UI-related information about a single
Entity, but differentiated by its Entity Types.

Each member name of the JSON object is an Entity Type Identifier and each
value is an Entity Type UI Info Object as defined in [Entity Type UI Info](#entity-type-ui-info).  

###### Entity Type UI Info

Entity Type UI Info is a JSON Object containing UI-related information about a
single Entity Type of an Entity.

The following claims MAY be used:

  - *display_name*: String. A human-readable name of the Entity to be presented to the End-User.
  - *description*: String. A human-readable brief description of this Entity presentable to the End-User.
  - *keywords*: JSON array with one or more strings representing search
  keywords, tags, categories, or labels that apply to this Entity.
  - *logo_uri*: String. A URL that points to the logo of this Entity.
  - *policy_uri*: String. A URL of the documentation of conditions and policies relevant to this Entity.
  - *information_uri*: String. A URL for documentation of additional information about this Entity viewable by the End-User.
  
Additional claims MAY be defined and used in conjunction with the claims above.

#### Example Response

```json
{
  "federation_entities": [
    {
      "entity_id": "https://green.example.com",
      "entity_types": [
        "federation_entity"
      ],
      "ui_infos": {
        "federation_entity": {
          "display_name": "The green organization",
          "logo_uri": "https://green.example.com/logo.png"
        }
      }
    },
    {
      "entity_id": "https://red.example.com",
      "entity_types": [
        "openid_relying_party",
        "federation_entity"
      ],
      "ui_infos": {
        "federation_entity": {
          "display_name": "Red Organization",
          "logo_uri": "https://red.example.com/logo.png"
        },
        "openid_relying_party": {
          "display_name": "Red RP",
          "logo_uri": "https://red.example.com/logo.png"
        }
      }
    },
    {
      "entity_id": "https://op.example.com",
      "entity_types": [
        "openid_provider"
      ]
    }
  ]
}
```

#  Claims Languages and Scripts

Human-readable claim values and claim values that reference human-readable values MAY be represented in multiple languages and scripts. This specification enables such representations in the same manner as defined in Section 14 of OpenID Federation [@!OpenID.Federation] and Section 5.2 of OpenID Connect Core 1.0 [@!OpenID.Core].

As described in OpenID Connect Core, to specify the languages and scripts, BCP47 [@!RFC5646] language tags are added to member names, delimited by a `#` character. For example, `family_name#ja-Kana-JP` expresses the Family Name in Katakana in Japanese, which is commonly used to index and represent the phonetics of the Kanji representation of the same name represented as `family_name#ja-Hani-JP`. 

The following is an example of an Entity Type UI Info Object with claims
represented in multiple languages:

```json
{
  "description": "Karlsruhe Institute of Technology - The Research University in the Helmholtz Association",
  "description#de": "Karlsruher Institut für Technologie - Die Forschungsuniversität in der Helmholtz-Gemeinschaft",
  "description#en": "Karlsruhe Institute of Technology - The Research University in the Helmholtz Association",
  "display_name": "Karlsruhe Institute of Technology (KIT)",
  "display_name#de": "Karlsruher Institut für Technologie (KIT)",
  "display_name#en": "Karlsruhe Institute of Technology (KIT)"
}
```

# Implementation Considerations

## Mapping Entity Configuration Claims to UI Info Response Claims 
It is up to the implementation to decide how the claims of UI Info Objects in the response are
populated. Implementations SHOULD consider the information published by entities
in their Entity Configuration and MAY consider additional information.

The following mapping between Claims in the `ui_infos` response Claim and Metadata Claims in the
Entity Configuration SHOULD be considered:

- `display_name`: The `display_name` Claim is a common Metadata Claim useable with all Entity Types. If set it SHOULD be copied to the response. If the `display_name` Claim is not set other Claims MAY be considered, such as:
  - For the `openid_relying_party` Entity Type: `client_name`
  - For the `oauth_client` Entity Type: `client_name`
  - For the `oauth_resource` Entity Type: `resource_name`
- `description`: The `description` Claim is a common Metadata Claim useable with all Entity Types. It SHOULD be copied to the response.
- `keywords`: The `keywords` Claim is a common Metadata Claim useable with all Entity Types. It SHOULD be copied to the response.
- `logo_uri`: The `logo_uri` Claim is a common Metadata Claim useable with all Entity Types. It SHOULD be copied to the response.
- `policy_uri`: The `policy_uri` Claim is a common Metadata Claim useable with all Entity Types. It SHOULD be copied to the response.
- `information_uri`: The `information_uri` Claim is a common Metadata Claim useable with all Entity Types. It SHOULD be copied to the response.

## Entity Collection Endpoint Scope

The responder is free to restrict the scope of its Entity Collection Endpoint,
such as, but not limited to:

- Only supporting a limited set of Trust Anchors.
- Filter out Entities from the response at their own discretion. Such additional filters MAY be:
  - Only Entities that have a certain Trust Mark.
  - Only Entities that have a valid Trust Chain to the Trust Anchor.
  - Only Entities that are resolvable at the Resolve Endpoint of the Entity
  providing the Entity Collection Endpoint.


# Security Considerations

In additional to the considerations below, the security considerations of
OpenID Federation 1.0 [@!OpenID.Federation] apply to this specification.

## Unsigned Response

The response from the Entity Collection Endpoint is not signed and the obtained
information should only be considered as informational.
To verify an Entity proper trust validation according to OpenID Federation 1.0 [@!OpenID.Federation]
still MUST be done.

It is also noted that Trust Marks returned in the response MAY not be verified
and clients MUST consider them as not yet verified.


{backmatter}


<reference anchor="OpenID.Core" target="http://openid.net/specs/openid-connect-core-1_0.html">
  <front>
    <title>OpenID Connect Core 1.0 incorporating errata set 2</title>
    <author initials="N." surname="Sakimura" fullname="Nat Sakimura">
      <organization>NRI</organization>
    </author>
    <author initials="J." surname="Bradley" fullname="John Bradley">
      <organization>Ping Identity</organization>
    </author>
    <author initials="M." surname="Jones" fullname="Michael B. Jones">
      <organization>Microsoft</organization>
    </author>
    <author initials="B." surname="de Medeiros" fullname="Breno de Medeiros">
      <organization>Google</organization>
    </author>
    <author initials="C." surname="Mortimore" fullname="Chuck Mortimore">
      <organization>Salesforce</organization>
    </author>
   <date day="15" month="December" year="2023"/>
  </front>
</reference>

<reference anchor="OpenID.Federation" target="https://openid.net/specs/openid-federation-1_0.html">
    <front>
        <title>OpenID Federation 1.0</title>
        <author fullname="R. Hedberg, Ed.">
            <organization>independent</organization>
        </author>
        <author fullname="Michael B. Jones">
            <organization>Self-Issued Consulting</organization>
        </author>
        <author fullname="A. Solberg">
            <organization>Sikt</organization>
        </author>
        <author fullname="John Bradley">
            <organization>Yubico</organization>
        </author>
        <author fullname="Giuseppe De Marco">
            <organization>independent</organization>
        </author>
        <author fullname="Vladimir Dzhuvinov">
            <organization>Connect2id</organization>
        </author>
        <date day="24" month="October" year="2024"/>
    </front>
</reference>

# Notices

Copyright (c) 2025 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer,
implementer, or other interested party a non-exclusive, royalty free,
worldwide copyright license to reproduce, prepare derivative works from,
distribute, perform and display, this Implementers Draft, Final
Specification, or Final Specification Incorporating Errata Corrections
solely for the purposes of (i) developing specifications,
and (ii) implementing Implementers Drafts, Final Specifications,
and Final Specification Incorporating Errata Corrections based
on such documents, provided that attribution be made to the OIDF as the
source of the material, but that such attribution does not indicate an
endorsement by the OIDF.

The technology described in this specification was made available
from contributions from various sources, including members of the OpenID
Foundation and others. Although the OpenID Foundation has taken steps to
help ensure that the technology is available for distribution, it takes
no position regarding the validity or scope of any intellectual property
or other rights that might be claimed to pertain to the implementation
or use of the technology described in this specification or the extent
to which any license under such rights might or might not be available;
neither does it represent that it has made any independent effort to
identify any such rights. The OpenID Foundation and the contributors to
this specification make no (and hereby expressly disclaim any)
warranties (express, implied, or otherwise), including implied
warranties of merchantability, non-infringement, fitness for a
particular purpose, or title, related to this specification, and the
entire risk as to implementing this specification is assumed by the
implementer. The OpenID Intellectual Property Rights policy
(found at openid.net) requires
contributors to offer a patent promise not to assert certain patent
claims against other contributors and against implementers.
OpenID invites any interested party to bring to its attention any
copyrights, patents, patent applications, or other proprietary rights
that may cover technology that may be required to practice this
specification.

# Acknowledgements

We would like to thank the following individuals for their contributions to this specification:
Niels van Dijk,
Stefan Santesson,
Phil Smart,
Zacharias Törnblom,
and the Geant Trust & Identity Incubator of Geant5-2.

# Document History

[[ To be removed from the final specification ]]

-00

* Initial version, fixing OpenID Federation issue #56.
