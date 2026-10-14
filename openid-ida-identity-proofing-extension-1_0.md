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

This specification also defines a small set of presentation claims, asserted by a Verifier, that describe the credential status and device binding checks performed at presentation. The cryptographic evidence behind those checks remains out of scope.

This specification is standalone. It defines every claim it uses and does not depend on, profile or extend OpenID Connect for Identity Assurance or its `verified_claims` structure.

Guidance for Relying Parties on evaluating the assurance of these claims is provided in the Security Considerations section.

# Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [@!RFC2119] [@!RFC8174] when, and only when, they appear in all capitals, as shown here.

# Terminology

Trust Framework
: A governance regime with an identified authority, operating rules and a conformance mechanism (for example, accreditation, notification or certification) under which an Issuer operates.

Policy
: A document, identified together with its edition, that defines assurance levels and the requirements for meeting them. A technical guideline or standard is a Policy, not a Trust Framework.

Proofing Event
: A single identity proofing process completed by the Issuer for a subject.

Check
: A single verification step performed within a Proofing Event.

Date-time
: A string in the format `YYYY-MM-DDThh:mm[:ss]TZD` as defined in [@!RFC3339], with a required time zone designator. A date-time is not a JWT NumericDate.

Issuer
: The entity that performed, or is accountable for, identity proofing and that issues the credential or assertion.

Verifier
: The entity that receives a presentation from a holder, performs the presentation checks and, in a Translation Binding, conveys the result to a Relying Party.

## Trust Classes

* **Proofing claims** (The Identity Proofing Domain) are asserted by the Issuer. They MUST be integrity-protected by the Issuer within the same signed structure as the identity claims they describe, for example the same mdoc Mobile Security Object or the same issuer-signed SD-JWT.
* **Presentation claims** (The Presentation Domain) are asserted by the Verifier. They MUST NOT appear within any structure signed by the Issuer.

An RP MUST reject a credential or presentation in which a presentation claim is covered by the Issuer's signature. An RP MUST NOT treat a proofing claim as Issuer-asserted if it was delivered outside the Issuer-signed structure or added by a party other than the Issuer.

Some claim names in this specification are also used in OpenID Connect for Identity Assurance. Any such correspondence is by name only: the definitions in this specification govern, and no structural or semantic dependency on that specification is implied.

# Standardized Claims and Values Registry (Normative)

To enable true jurisdictional agility and non-repudiation, Issuers and Verifiers MUST map local proofing terminology and downstream translation assertions to this standardized registry of claims and enumerated values. The claims are divided into two classes that differ in who asserts them and how they are protected.

## The Identity Proofing Domain: Vetting & Assurance Claims

These claims are asserted by the Issuer and describe the rigor of the initial onboarding phase.

### Context Claims

| Claim | Requirement | Data Type | Example Values | Description |
| :--- | :--- | :--- | :--- | :--- |
| `trust_framework` | OPTIONAL | String | `eidas`, `us_real_id` | Identifier of the Trust Framework that governed the issuance. MUST be omitted when no Trust Framework governs the issuance. A Policy MUST NOT be conveyed in this claim. |
| `policy` | REQUIRED | String | `NIST SP 800-63-4`, `ETSI TS 119 461 V2.1.1` | Identifier of the Policy, including its edition, that defines the value of `assurance_level`. |
 
### Assurance Levels

| Claim | Requirement | Data Type | Example Values | Description |
| :--- | :--- | :--- | :--- | :--- |
| `assurance_level` | REQUIRED | String | `high`, `ial2` | Assurance level achieved, as defined by the Policy identified in `policy`. |

`assurance_level` MUST be interpreted only against the Policy identified by `policy`. It MUST NOT be interpreted against `trust_framework`. An RP that does not recognize the value of `policy` MUST treat the assurance level as unknown.

`trust_framework`, `policy` and `assurance_level` MUST each carry a single value. A value that combines a framework, a policy or a level in one string (for example `loa:high:eu:eidas` or `nist_800_63:ial2`) is invalid, and an RP MUST treat a claim carrying such a value as invalid.

This specification does not define a framework-independent or harmonized assurance level. Evidence-classification detail is out of scope for this specification.

The following combinations are illustrative only.

| `trust_framework` | `policy` | `assurance_level` |
| :--- | :--- | :--- |
| `eidas` | `Commission Implementing Regulation (EU) 2015/1502` | `high` |
 
### Proofing Event Claims

| Claim | Requirement | Data Type | Description |
| :--- | :--- | :--- | :--- |
| `verification_time` | REQUIRED | Date-time | Time at which the Proofing Event completed. It is distinct from, and MUST NOT be derived from, the credential's issuance or validity time (for example `iat`, `nbf` or `validFrom`). |
| `verification_process` | REQUIRED | String | Identifier of the Proofing Event, unique within the Issuer. |
| `proofing_checks` | REQUIRED | Array of objects | One member for each Check performed in the Proofing Event. MUST contain at least one member. |

### Proofing Checks

Each member of `proofing_checks` is a JSON object (or, in CBOR, a map) with the following members:

| Member | Requirement | Data Type | Description |
| :--- | :--- | :--- | :--- |
| `check_method` | REQUIRED | String | Presence method used for this Check. See Check Method Values. |
| `check_time` | REQUIRED | Date-time | Time at which this Check finished. |
| `check_id` | REQUIRED | String | Stable identifier of this Check. When the same Check is conveyed in another credential format, the same value MUST be used. |
| `check_organization` | CONDITIONAL | String | Name of the organization that performed this Check. REQUIRED when that organization is identified by the Issuer's signature or signing certificate; MUST be omitted otherwise. |

An RP MUST reject a `check_organization` value that does not match an organization identified by the Issuer's signature or signing certificate.

### Check Method Values

`check_method` identifies how the subject was present during the Check. Presence of the subject during proofing is determined by `check_method` and SHALL NOT be separately asserted.

| Value | Description |
| :--- | :--- |
| `pipp` | Physical In-Person Proofing |
| `sripp` | Supervised Remote In-Person Proofing |
| `uripp` | Unsupervised Remote In-Person Proofing |

Other values MAY be used where a Policy or Trust Framework defines them. An RP that does not recognize a `check_method` value MUST treat the subject's presence for that Check as unknown.
 
### Assurance Namespace

To enable cross-format interoperability, Issuers MUST carry proofing claims as follows:

* **ISO/IEC 18013-5/7 (CBOR mdoc):** as data elements in the namespace `org.openid.ida.assurance.1`, using the claim names in this specification as data element identifiers. `proofing_checks` is an array of maps.
* **JWT-based formats (OpenID Connect assertions, SD-JWT):** as top-level claims using the names registered in IANA Considerations.
* **W3C Verifiable Credentials:** as properties of `credentialSubject`, with term definitions in a JSON-LD context published with this specification [TBD].

Presentation claims MUST NOT be carried in this namespace or in any other Issuer-signed structure.

---

## The Presentation Domain: Verifier Asserted Claims

These claims are asserted by the Verifier and describe checks performed for a specific presentation. When a Verifier conveys the result of a presentation to an RP, each of the following claims is REQUIRED in that conveyed payload. In a Native Edge Binding, the RP performs these checks itself and they are not conveyed.

### Presentation Claims

| Claim | Requirement | Data Type | Description |
| :--- | :--- | :--- | :--- |
| `revocation_freshness_check` | REQUIRED | Date-time | Time at which the credential's status was validated for this presentation. |
| `revocation_freshness_method` | REQUIRED | String | Mechanism by which status was validated. See Revocation Freshness Method Values. |
| `device_binding_verified` | REQUIRED | Boolean | `true` only if the Verifier verified the presentation's device signature against the holder key bound in the credential and has evidence that the key is hardware-bound; `false` otherwise. |

### Revocation Freshness Method Values

| Value | Description |
| :--- | :--- |
| `status_list` | Token Status List or W3C Bitstring Status List |
| `ocsp` | Online Certificate Status Protocol |
| `crl` | Certificate Revocation List |
| `issuer_status_api` | Status endpoint operated by the Issuer |
| `wallet_asserted` | Status asserted by the wallet or device |

Other values MAY be used where a Trust Framework defines them.

# Architectural Binding Profiles (Implementation Mechanics)

Because this specification is protocol-independent, Relying Parties MUST consume these claims via an architecture-appropriate Implementation Profile.

## Native Edge Binding (e.g., Proximity Terminals)

For RPs operating physical hardware (e.g., POS terminals, offline readers) consuming ISO/IEC 18013-5 over BLE/NFC, no format translation is required. The RP consumes the raw CBOR binary directly. The Core Vocabulary is parsed natively from the `org.openid.ida.assurance.1` namespace.

## Translation Binding (e.g., Web-Native OpenID Connect)

For web-native enterprise RPs (e.g., Core Banking Systems) that lack the capacity to process heavy binary protocols or manage edge device engagement, an intermediate Verifier is utilized. The Verifier executes the complex cryptography and translates the claims and values into a normalized JSON payload.

How the Verifier Uses This Table

When a Verifier sits between the Wallet and a Relying Party's enterprise backend, this IANA registry acts as the definitive translation map.
If the Verifier receives a CBOR payload over ISO 18013-7, it doesn't look for the string "proofing_level". It parses the binary for the assigned integer key (e.g., -260). Upon validating the math, it cross-references this IANA registry, sees that -260 perfectly maps to the JWT claim "proofing_level", and injects that string into the normalized OpenID Connect JSON envelope for the Relying Party. This ensures complete semantic parity between the physical edge and the enterprise web.

A Verifier performing a Translation Binding:

* MUST translate only claims that have a registered correspondence, and MUST NOT translate any other claim into a registered claim name;
* MUST NOT alter, normalize, combine or map proofing claim values across Policies or Trust Frameworks;
* MUST convey presentation claims in a payload that the Verifier integrity-protects (for example, by signing it), distinct from the Issuer-signed evidence; and
* when it conveys issuer-signed or device-signed evidence (for example, CBOR Mobile Security Object blocks) within a JSON payload, MUST encode that evidence (for example, as Base64URL) to allow safe nesting. Encoding provides safe transport only; it provides no integrity protection.

# Security Considerations
While this specification defines claims meant to provide Relying Parties with assurance information about a given credential or assertion, it does not define the architecture, trust model, or cryptographic properties needed to verify the assurance of the claims themselves. Depending on the use case and trust architecture, Relying Parties need to determine the credential types and cryptographic validity and integrity protections needed to accept a given credential or assertion.
This section provides security considerations for relying parties when evaluating and accepting the claims defined in this specification. The security considerations outlined assume that claims are provided to a relying party either as attributes in an identity credential, such as a verifiable digital credential, or as part of an identity assertion, such as an Open ID Connect token. The security controls chosen will be greatly impacted by relying party risk tolerance and the architecture between the issuer/identity provider, the verifier and the relying party. For example, some relying parties may require the credential or assertion to be signed by the issuer/identity provider, while others may rely only on an encrypted channel for trust conveyance. 

Where cryptographic evidence is used to support a risk-based decision, its conveyance, retention, and format are implementation-specific and are left to the applicable architecture and trust framework rather than defined as normative claims in this document. This section does highlight examples of how normative claims currently in the IDA specification can be used to express the security properties of how an assertion or credential was cryptographically protected during transport and verified. 

## Certification
This section will talk about security considerations agnostic of IdP, Issuer, Wallet or Verifier certification. However, it’s worth noting that current and evolving third-party certification regimes for identity infrastructure may impact the claims relying parties choose to accept. For example, a single claim that attests to an issuer’s identity proofing process being certified against accepted requirements may replace the need for several, granular claims about the mechanism and controls used in the proofing process. Relying parties should evaluate availability and applicability of certification when assessing the risk of accepting identity and assurance claims.

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

This specification requests registration of the following claims in the IANA "JSON Web Token Claims" registry established by [@!RFC7519]. These registrations standardize the semantic representation of how an identity was vetted and proofed prior to issuance, allowing Relying Parties to evaluate identity proofing rigor across format-agnostic architectures.

**Registry Name:** JSON Web Token Claims
**Change Controller:** OpenID Foundation
**Specification Document:** [[ This Document ]]

| Claim Name | Claim Description | Change Controller | Specification Document(s) |
| :--- | :--- | :--- | :--- |
| `trust_framework` | Trust Framework that governed the issuance. | OpenID Foundation | [[ This Document ]] |
| `policy` | Policy, including edition, that defines `assurance_level`. | OpenID Foundation | [[ This Document ]] |
| `assurance_level` | Assurance level achieved, as defined by `policy`. | OpenID Foundation | [[ This Document ]] |
| `proofing_checks` | Checks performed during the identity proofing event. | OpenID Foundation | [[ This Document ]] |
| `verification_time` | Time at which identity proofing completed. | OpenID Foundation | [[ This Document ]] |
| `verification_process` | Identifier of the identity proofing event. | OpenID Foundation | [[ This Document ]] |
| `revocation_freshness_check` | Time at which credential status was validated for a presentation. | OpenID Foundation | [[ This Document ]] |
| `revocation_freshness_method` | Mechanism by which credential status was validated. | OpenID Foundation | [[ This Document ]] |
| `device_binding_verified` | Whether the device binding was verified and the key is hardware-bound. | OpenID Foundation | [[ This Document ]] |


## CBOR Web Token (CWT) Claims Registration

This specification requests registration of the following value in the IANA "CBOR Web Token Claims Registry" established by [RFC8392]. These registrations provide integer-based claim keys for the Format-Agnostic Identity Core Vocabulary, enabling high-assurance identity proofing and cryptographic pass-through evidence to be transmitted efficiently in edge-native constrained environments.

This specification requests registration of the following claims in the IANA "CBOR Web Token (CWT) Claims" registry established by [RFC8392]. These registrations provide integer-based claim keys for the Format-Agnostic Identity Core Vocabulary, enabling high-assurance identity proofing and cryptographic pass-through evidence to be transmitted efficiently in edge-native constrained environments.

Registry Name: CBOR Web Token (CWT) Claims
Change Controller: OpenID Foundation (or IETF, depending on the final submission track)
Specification Document: [[ This Document ]]

| Claim Name | Claim Description | JWT Claim Name | Claim Key |
| :--- | :--- | :--- | :--- |
| `trust_framework` | Trust Framework that governed the issuance. | `trust_framework` | `[TBD]` |
| `policy` | Policy, including edition, that defines `assurance_level`. | `policy` | `[TBD]` |
| `assurance_level` | Assurance level achieved, as defined by `policy`. | `assurance_level` | `[TBD]` |
| `proofing_checks` | Checks performed during the identity proofing event. | `proofing_checks` | `[TBD]` |
| `verification_time` | Time at which identity proofing completed. | `verification_time` | `[TBD]` |
| `verification_process` | Identifier of the identity proofing event. | `verification_process` | `[TBD]` |
| `revocation_freshness_check` | Time at which credential status was validated for a presentation. | `revocation_freshness_check` | `[TBD]` |
| `revocation_freshness_method` | Mechanism by which credential status was validated. | `revocation_freshness_method` | `[TBD]` |
| `device_binding_verified` | Whether the device binding was verified and the key is hardware-bound. | `device_binding_verified` | `[TBD]` |

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
