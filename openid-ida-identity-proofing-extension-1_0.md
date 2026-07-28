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

This document is a targeted profile of the Format-Agnostic Digital Identity Claims and Values 1.0 specification. It isolates and defines the globally interoperable architecture of digital identity claims and values. Crucially, this profile explicitly separates the claims related to **Identity Proofing assurance** (how the human was originally vetted by the Issuer) from the **Cryptographic Security** claims (how the assertion is protected during transit). By standardizing the semantic representation of these two distinct domains, and defining the requirements for end-to-end cryptographic non-repudiation through intermediate translation layers, this profile empowers Relying Parties (RPs) to solve the problem of executing automated, risk-defensible decisions based on strict mathematical proof.


{mainmatter}


# Introduction

The digital identity ecosystem is heavily fragmented across root cryptographic formats (e.g., CBOR for ISO/IEC 18013-5 mdocs, JSON-LD for W3C VCDMs) and transport protocols.

To solve the utility problem across all enterprise architectures, this specification decouples the **Claims** from the **Protocol**. It standardizes the specific claims necessary to evaluate identity proofing rigor and end-to-end non-repudiation. Implementers can natively embed these claims into edge-based binary protocols or translate them into enterprise-friendly web formats via intermediate orchestration services.

# Scope

This specification defines the discrete claims, data types, and enumerated values associated with:

1. The identity proofing and vetting phase executed by an Issuing Authority.
2. The cryptographic pass-through evidence required to maintain non-repudiation through intermediate translation layers.

**Out of Scope:** The specific transport protocol, API definitions, or envelope formatting (e.g., JSON OIDC assertions, REST APIs, BLE/NFC handshakes) used to deliver these claims. Those mechanics MUST be defined in separate, architecture-specific Profiles.


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

| Value Category | Example Parameter Values | Description |
| :--- | :--- | :--- |
| **Proofing** | `pipp`, `uripp`, `sripp` | Physical In-Person Proofing (`pipp`), Unsupervised Remote (`uripp`), Supervised Remote (`sripp`). |

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

These claims carry cryptographic evidence generated at issuance or presentation time. Their purpose is to enable a Relying Party to:

- independently re-verify the Issuer’s signature (or equivalent structure),
- confirm device-bound presentation where applicable, and
- retain durable, format-independent evidence for non-repudiation and Examiner Defence.

The claims themselves are **carriers** of the evidence. Cryptographic assurance is obtained only when the Relying Party (or a component acting on its behalf) successfully verifies the received signatures or structures against the appropriate trust anchors. The design intentionally keeps this evidence available even after the surrounding assertion has been translated into another encoding or protocol envelope.

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

This specification requests registration of the following value in the IANA "CBOR Web Token Claims Registry" established by [@!RFC8392]. These registrations provide integer-based claim keys for the Format-Agnostic Identity Core Vocabulary, enabling high-assurance identity proofing and cryptographic pass-through evidence to be transmitted efficiently in edge-native constrained environments.

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
| `issuer_signed_receipt` | Raw structure from the root token proving the trust anchor. | `issuer_signed_receipt` | `[TBD]` |
| `device_signed_receipt` | Raw structure proving localized hardware intent and non-repudiation. | `device_signed_receipt` | `[TBD]` |
| `verifier_signature_attestation` | Cryptographic signature of an intermediate Verifier translation. | `verifier_signature_attestation` | `[TBD]` |
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
