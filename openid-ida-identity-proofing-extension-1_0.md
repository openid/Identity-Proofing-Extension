%%%
title = "Format-Agnostic Digital Identity Claims and Values: Identity Proofing Profile 1.0 - draft 00"
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

This document is a targeted profile of the *Format-Agnostic Digital Identity Claims and Values 1.0* specification. It isolates and defines the globally interoperable framework of digital identity claims and values specifically related to **Identity Proofing assurance from Issuers**. By standardizing the semantic representation of how an identity was vetted and proofed prior to issuance, this profile empowers Relying Parties (RPs) to execute automated risk decisions based on the rigorousness of the Issuer's identity proofing phase.


{mainmatter}


# Introduction

To transition to Verifiable Digital Credentials (VDCs), Relying Parties (RPs) require a standardized method to evaluate the confidence level of the initial identity proofing performed by the Issuer. This profile standardizes the claims and values necessary to transport identity proofing assurance data across different formats, allowing RPs to establish a defensible "reasonable belief" regarding the rigor of the vetting process.


# Scope

This specification extends the OpenID Identity Assurance Schema Definition 1.0 by isolating the specific claims, namespaces, and enumerated values associated strictly with the **identity proofing and vetting phase** executed by the Issuing Authority. It excludes post-issuance operational metrics, wallet hardware security assertions, and ongoing revocation checks.


# Standardized Parameter Value Registry (Normative)

## Assurance Namespace

To enable cross-format interoperability of proofing claims, implementations MUST utilize the registered OpenID IDA Assurance namespace: 
* **ISO/IEC 18013-5/7 (CBOR mdoc):** `org.openid.ida.assurance.1`
* **W3C Verifiable Credentials (JSON/SD-JWT):** `assurance_level` property mapped within the credential `@context`.

## Proofing Assurance Classifications
* **`issuance_assurance_classification`**: The specific vetting standard met at the time of issuance (e.g., `status:full_compliance:us:real_id`, `loa:high:eu:eidas`). This serves as the primary integer weight representing the Issuer's proofing rigor.


# Standardized Claims and Values Registry (Normative)

To enable true jurisdictional agility, Issuers MUST map local proofing terminology to this standardized registry of claims and enumerated values.

## Proofing Context Claims

These claims establish the regulatory framework and the environmental context under which the identity was originally proofed.

| Claim | Example Parameter Values | Data Type | Description |
| :--- | :--- | :--- | :--- |
| **`context_uri`** | `urn:openid:assurance:us:real_id`, `urn:openid:assurance:eu:eidas` | String | Defines the specific legal or regulatory trust framework governing the initial proofing and vetting. |
| **`presence_equivalence`** | `in_person`, `remote_supervised` | String | Indicates the context of the user's presence during the initial proofing phase. |

## Proofing Assurance Levels

These claims provide the discrete variables required to communicate the confidence established during the identity proofing lifecycle.

| Claim | Example Parameter Values | Data Type | Description |
| :--- | :--- | :--- | :--- |
| **`issuance_assurance_classification`** | `status:full_compliance:us:real_id`, `loa:high:eu:eidas` | String | Asserts the rigor of the vetting process and confidence established during proofing by the Issuer. |
| **`proofing_level`** | `ial:2`, `ip:3` | String | Harmonized assurance level mapping representing the Identity Assurance Level (IAL) or Identity Proofing (IP) level for cross-sector compatibility. |

## Proofing Verification Methods (`check_method`)

Provides the methodology used by the Issuer to validate the identity evidence during the proofing phase.

| Value Category | Example Parameter Values | Description |
| :--- | :--- | :--- |
| **`Proofing`** | `pipp`, `uripp`, `sripp` | **Physical In-Person Proofing** (`pipp`), **Unsupervised Remote** (`uripp`), **Supervised Remote** (`sripp`). |

---


# Compliance Elements: Provenance & Jurisdictional Assurance

**Requirement:** Verify the credential was issued under recognized, government-mandated proofing standards.

**Implementation:** Establish a standardized, format-agnostic namespace (e.g., `org.openid.ida.assurance.1`) that Issuers MUST incorporate natively into the credential to explicitly declare the proofing standards met. 

**RP Action:** RPs SHOULD explicitly request proofing data elements from this namespace via Selective Disclosure during the presentation phase to evaluate the native assurance level of the Issuer's vetting process prior to full PII payload extraction.


# Acknowledgements

We would like to thank the following individuals for their contributions to the specification:
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
