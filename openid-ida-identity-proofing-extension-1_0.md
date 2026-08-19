%%%
title = "Format-Agnostic Digital Identity Claims and Values: Identity Proofing Extension 1.0 - draft 00"
abbrev = "Format-Agnostic Identity Claims and Values"
ipr = "none"
workgroup = "OpenID eKYC-IDA"
keyword = ["security", "openid", "identity assurance", "ekyc", "claims", "mdoc", "vcdm", "format-agnostic"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-ida-identity-proofing-extension-1_0"
status = "standard"

[[author]]
initials="J."
surname="Cafik"
fullname="Juliana Cafik"
organization="Independent"
    [author.address]
    email = "juliana@cafik.ca"

%%%

.# Abstract

This specification is an extension of OpenID Connect for Identity Assurance Claims Registration 1.0. It defines discrete claims and values describing Identity Proofing assurance (the vetting and issuance process performed by an issuer or identity provider). By standardizing the semantic representation of these claims, this profile provides Relying Parties (RPs) with critical assurance information that enables automated risk decisions when accepting an identity credential or assertion. Considerations for evaluating the cryptographic evidence and trust architecture associated with a given presentation are addressed in Security Considerations and are left to the implementation architectures rather than defined here as normative claims.

{mainmatter}


# Introduction

Individuals and organizations now rely on a diverse ecosystem of identities and credentials. While this growth increases convenience and enables new forms of trusted engagement, it also introduces greater complexity for relying parties that must determine whether to trust a given identity assertion or credential. Making informed risk decisions requires additional assurance information about the process used to issue the identity assertion or credential that is being presented to an RP. This specification defines a set of identity claims and values that can represent the identity proofing process. It's anticipated that these claims can be included in an identity assertion such as OpenID Connect (OIDC) tokens or in credentials such as Verifiable Digital Credentials (VDCs). The claims in this specification empowers Relying Parties with identity evidence that supports executing automated and defensible risk decisions.

# Scope

This specification defines discrete claims, data types, and enumerated values that represents the identity proofing and vetting executed by an issuer or identity provider during credential or identity issuance. These claims can accompany an identity credential or assertion, and are intended to be presented to relying parties during credential or assertion presentation. Claims defined in this specification are meant to be architecture and protocol agnostic. For this reason, the following items are out of scope: Transport protocols, API definitions, envelope formatting (JSON OIDC assertions, REST APIs, BLE/NFC handshakes), cryptographic evidence conveyance mechanisms (e.g., issuer-signed receipts, device-signed receipts, verifier attestations, or equivalent), and other architecture-specific delivery mechanics.

Guidance for Relying Parties on evaluating the assurance of these claims is provided in the Security Considerations section.

# Standardized Claims and Values Registry (Normative)

To enable true jurisdictional agility and non-repudiation, Issuers and Verifiers MUST map local proofing terminology and downstream translation assertions to this standardized registry of claims and enumerated values. This registry is strictly divided into two operational domains: Identity Proofing and Cryptographic Security.

## The Identity Proofing Domain: Vetting & Assurance Claims

These claims are asserted by the Issuer and describe the rigor of the initial onboarding phase.

### Context & Environmental Claims

| Claim | Example Parameter Values | Data Type | Description |
| :--- | :--- | :--- | :--- |
| `context_uri` | `urn:openid:assurance:us:real_id` | String | Defines the legal or regulatory standard governing the initial proofing. |
| `presence_equivalence` | `in_person`, `remote_supervised` | String | Indicates the context of the user's presence during the proofing phase. |

### Assurance Levels & Classifications

| Claim | Example Parameter Values | Data Type | Description |
| :--- | :--- | :--- | :--- |
| `issuance_assurance_classification` | `loa:high:eu:eidas` | String | Asserts the vetting rigor and confidence established by the Issuer. |
| `proofing_level` | `ial:2`, `ip:3` | String | Harmonized mapping representing the Identity Assurance Level (IAL). |

### Proofing Verification Methods (`check_method`)
The values in the table below are conveyed via the `check_method` claim registered in the IANA JWT and CWT tables later in this document.

| Claim | Value | Description |
| :--- | :--- | :--- |
| `check_method` | `pipp` | Physical In-Person Proofing |
| `check_method` | `uripp` | Unsupervised Remote In-Person Proofing |
| `check_method` | `sripp` | Supervised Remote In-Person Proofing |

### Assurance Namespace

To enable cross-format interoperability of proofing claims, implementations MUST utilize a registered OpenID IDA Assurance namespace:
* **ISO/IEC 18013-5/7 (CBOR mdoc):** `org.openid.ida.assurance.1`
* **W3C Verifiable Credentials (JSON/SD-JWT):** `assurance_level` property mapped within the credential `@context`.

---

## The Cryptographic Security & Presentation Domain

These claims establish the active trust architecture at the moment of presentation, providing an Examiner Defense against synthetic mimicry and proxy compromise.

### Pass-Through Cryptographic Evidence

| Claim | Data Type | Description |
| :--- | :--- | :--- |
| `issuer_signed_receipt` | Object/String/Binary | The raw structure (e.g., `IssuerSigned` block) from the root token, allowing the RP to independently verify the trust anchor. |
| `device_signed_receipt` | Object/String/Binary | The raw structure (e.g., `DeviceSigned` block), proving a localized hardware unlock occurred and enforcing strict non-repudiation. |
| `verifier_signature_attestation` | Object/String/Binary | Cryptographic signature of an intermediate Verifier (if used), binding a translated payload to the root receipts. |

### Live Presentation Metrics

| Claim | Data Type | Description |
| :--- | :--- | :--- |
| `revocation_freshness_check` | String (DateTime) | Timestamp confirming the exact moment the credential's status was validated. |
| `revocation_freshness_method` | String | Mechanism used to validate status (e.g., `cached_vical`, `status_list`, `ocsp`, `token_status_api`), determining the risk of cache poisoning. |
| `device_binding_verified` | Boolean | Declares whether the presentation key is bound securely to physical hardware (`TRUE`/`FALSE`). |

# Architectural Binding Profiles (Implementation Mechanics)

Because this specification is protocol-independent, Relying Parties MUST consume these claims via an architecture-appropriate Implementation Profile.

## Native Edge Binding (e.g., Proximity Terminals)

For RPs operating physical hardware (e.g., POS terminals, offline readers) consuming ISO/IEC 18013-5 over BLE/NFC, no format translation is required. The RP consumes the raw CBOR binary directly. The Core Vocabulary is parsed natively from the `org.openid.ida.assurance.1` namespace.

## Translation Binding (e.g., Web-Native OpenID Connect)

For web-native enterprise RPs (e.g., Core Banking Systems) that lack the capacity to process heavy binary protocols or manage edge device engagement, an intermediate Verifier is utilized. The Verifier executes the complex cryptography and translates the claims and values into a normalized JSON payload.

How the Verifier Uses This Table

When a Verifier sits between the Wallet and a Relying Party's enterprise backend, this IANA registry acts as the definitive translation map.
If the Verifier receives a CBOR payload over ISO 18013-7, it doesn't look for the string "proofing_level". It parses the binary for the assigned integer key (e.g., -260). Upon validating the math, it cross-references this IANA registry, sees that -260 perfectly maps to the JWT claim "proofing_level", and injects that string into the normalized OpenID Connect JSON envelope for the Relying Party. This ensures complete semantic parity between the physical edge and the enterprise web.

* **Encoding Mandate:** When utilizing a JSON translation binding, all non-JSON cryptographic structures (e.g., CBOR MSO blocks) mapped to `issuer_signed_receipt` and `device_signed_receipt` MUST be encoded (e.g., Base64URL) to allow safe nesting within the JSON envelope.

# Security Considerations

## The Cryptographic Security & Presentation Domain

Some credential formats enables claims to carry cryptographic evidence generated at issuance or presentation time. The  purpose of these cryptographic protections is to enable a Relying Party to:

- independently verify the Issuer’s signature (or equivalent structure),
- confirm device-bound presentation where applicable, and
- retain verifiable evidence for non-repudiation.

Cryptographic assurance is obtained only when the Relying Party (or a component acting on its behalf) successfully verifies the received signatures or structures against the appropriate trust anchors. The design intentionally keeps this evidence available even after the surrounding assertion has been translated into another encoding or protocol envelope.

### Pass-Through Cryptographic Evidence

| Claim | Data Type | Description |
| :--- | :--- | :--- |
| `issuer_signed_receipt` | Object/String/Binary | Carries the Issuer’s cryptographic signature (or the equivalent signed structure) over the core identity data. Allows the RP to re-verify the trust anchor independently of any intermediate translation. |
| `device_signed_receipt` | Object/String/Binary | Carries evidence of a device-bound signature (or equivalent hardware-backed attestation) produced at presentation time. Supports proof of possession and local user intent. |
| `verifier_signature_attestation` | Object/String/Binary | Carries a signature produced by an intermediate Verifier that binds the translated payload to the original receipts. Provides accountability for the translation step. |

### Live Presentation Metrics

| Claim | Data Type | Description |
| :--- | :--- | :--- |
| `revocation_freshness_check` | String (DateTime) | Timestamp confirming the exact moment the credential's status was validated. |
| `revocation_freshness_method` | String | Mechanism used to validate status (e.g., `cached_vical`, `status_list`, `ocsp`, `token_status_api`). Enables the RP to assess residual risk of cache poisoning or stale status. |
| `device_binding_verified` | Boolean | Assertion by the presenter or intermediate Verifier that the presentation key is bound to hardware. This is a claim *about* binding status, not cryptographic proof of that binding. |

**Limitation.** These claims supply the cryptographic material necessary for independent verification and long-term evidence retention. They do not, by themselves, constitute a complete security proof. Correct verification of the carried signatures/structures, proper trust-anchor management, and evaluation of freshness and device-binding status remain the responsibility of the Relying Party.

## The Envelope vs. The Receipt (Format Translation Integrity)

# Security Considerations

## The Envelope vs. The Receipt (Format Translation Integrity)

When a Relying Party utilises an intermediate Verifier and a Translation Binding (such as OIDC JSON), the RP typically relies on the JSON envelope for immediate business logic. The JSON envelope alone is insufficient for Examiner Defence: it could be synthesised by a compromised or malicious Verifier.

The RP **shall not** treat the translated JSON envelope as the sole root of trust.  
The RP **shall** extract and archive the format-agnostic cryptographic evidence contained in the `issuer_signed_receipt` and `device_signed_receipt` fields (and, when present, `verifier_signature_attestation`).

While the RP may consume the JSON translation for operational speed, the retained encoded binary (or equivalent) receipts constitute the immutable evidence required for non-repudiation and KYC Examiner Defence. This requirement applies equally to native edge presentations, pure VCDM flows, and translated OIDC flows.

## Decoupled Trust Resolution (Asynchronous Caching)

To eliminate runtime network latency and protect user privacy, architectures processing these claims **should** decouple the transaction path from the trust-list / VICAL resolution path. Synchronous API calls to a Verified Issuer Certificate Authority List (VICAL) or equivalent during a live transaction introduce availability and privacy risks.

The RP **shall** evaluate the `revocation_freshness_method` claim (when present) to determine whether status validation was performed via:

- an asynchronous, locally cached registry,
- a synchronous network call, or
- the wallet / device itself.

This evaluation is a necessary input to the RP’s residual-risk assessment.

## Independence from End-User Attribute Claims

The security requirements in this section apply to the cryptographic evidence and presentation-integrity claims defined in this specification. They are independent of whether any end-user attribute claims registered in OpenID Connect for Identity Assurance Claims Registration 1.0 are also present in the assertion.

## Decoupled Trust Resolution (Asynchronous Caching)

To eliminate runtime network latency and protect user privacy, architectures processing these claims **should** decouple the transaction path from the trust-list / VICAL resolution path. Synchronous API calls to a Verified Issuer Certificate Authority List (VICAL) or equivalent during a live transaction introduce availability and privacy risks.

The RP **shall** evaluate the `revocation_freshness_method` claim (when present) to determine whether status validation was performed via:

- an asynchronous, locally cached registry,
- a synchronous network call, or
- the wallet / device itself.

This evaluation is a necessary input to the RP’s residual-risk assessment.# Compliance Elements: Provenance & Jurisdictional Assurance

## Namespace Enforcement

Establish a standardized, format-agnostic namespace (e.g., `org.openid.ida.assurance.1`) that Issuers MUST incorporate natively into the credential to explicitly declare the proofing standards met.

## Selective Disclosure Request

RPs SHOULD explicitly request proofing data elements from this namespace via Selective Disclosure during the presentation phase to evaluate the native assurance level of the Issuer's vetting process prior to full PII payload presentation and extraction.

# IANA Considerations

## JSON Web Token Claims Registration

This specification requests registration of the following value in the IANA "JSON Web Token Claims Registry" established by [@!RFC7519]. These registrations standardize the semantic representation of how an identity was vetted and proofed prior to issuance, allowing Relying Parties to evaluate identity proofing rigor across format-agnostic architectures.

**Registry Name:** JSON Web Token Claims
**Change Controller:** OpenID Foundation
**Specification Document:** [[ This Document ]]

| Claim Name | Claim Description | Change Controller | Specification Document(s) |
| :--- | :--- | :--- | :--- |
| `context_uri` | Defines the legal or regulatory standard governing the initial identity proofing. | OpenID Foundation | [[ This Document ]] |
| `presence_equivalence` | Indicates the context of the user's presence during the initial proofing phase. | OpenID Foundation | [[ This Document ]] |
| `issuance_assurance_classification` | Asserts the vetting rigor and confidence established by the Issuer at the time of issuance. | OpenID Foundation | [[ This Document ]] |
| `proofing_level` | Harmonized mapping representing the Identity Assurance Level (IAL) or Identity Proofing (IP) level. | OpenID Foundation | [[ This Document ]] |
| `check_method` | The verification methodology used by the Issuer to validate the identity evidence during onboarding. | OpenID Foundation | [[ This Document ]] |

## CBOR Web Token (CWT) Claims Registration

This specification requests registration of the following value in the IANA "CBOR Web Token Claims Registry" established by [RFC8392]. These registrations provide integer-based claim keys for the Format-Agnostic Identity Core Vocabulary, enabling high-assurance identity proofing and cryptographic pass-through evidence to be transmitted efficiently in edge-native constrained environments.

Registry Name: CBOR Web Token (CWT) Claims
Change Controller: OpenID Foundation (or IETF, depending on the final submission track)
Specification Document: [[ This Document ]]

| Claim Name | Claim Description | JWT Claim Name | Claim Key |
| :--- | :--- | :--- | :--- |
| `context_uri` | Defines the legal/regulatory standard governing the initial proofing. | `context_uri` | `[TBD]` |
| `presence_equivalence` | Indicates the context of the user's presence during proofing. | `presence_equivalence` | `[TBD]` |
| `issuance_assurance_classification` | Asserts the vetting rigor and confidence established by the Issuer. | `issuance_assurance_classification` | `[TBD]` |
| `proofing_level` | Harmonized mapping representing the Identity Assurance Level. | `proofing_level` | `[TBD]` |
| `check_method` | The verification methodology used during onboarding. | `check_method` | `[TBD]` |
| `revocation_freshness_check` | Timestamp confirming the exact moment the credential's status was validated. | `revocation_freshness_check` | `[TBD]` |
| `revocation_freshness_method` | Mechanism used to validate the credential's status. | `revocation_freshness_method` | `[TBD]` |
| `device_binding_verified` | Declares whether the presentation key is securely bound to hardware. | `device_binding_verified` | `[TBD]` |

Note to RFC Editor: Please replace [TBD] with the integer values assigned by IANA, typically allocated from the standard specification space (e.g., negative integers for early allocations or standard positive integers post-RFC).


# Acknowledgements

We would like to thank the following individuals for their feedback and contributions that helped evolve this document:
Bill Fisher,
Naohiro Fujie,
Michael B. Jones,
and
Ryan Galluzzo.


{backmatter}


# Notices

Copyright (c) 2026 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft, Final Specification, or Final Specification Incorporating Errata Corrections solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts, Final Specifications, and Final Specification Incorporating Errata Corrections based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy (found at openid.net) requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. OpenID invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that may cover technology that may be required to practice this specification.


# Document History

[[ To be removed from the final specification ]]

-00

*  Initial version
