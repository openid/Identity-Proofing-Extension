content = """**Workgroup:**
OpenID eKYC-IDA  
**Published:**
15 May 2026  
**Author:**
J. Cafik  
Independent  

# Format-Agnostic Digital Identity Claims and Values 1.0: Identity Proofing Profile

### Abstract
This document is a targeted profile of the *Format-Agnostic Digital Identity Claims and Values 1.0* specification. It isolates and defines the globally interoperable framework of digital identity claims and values specifically related to **Identity Proofing assurance from Issuers**. By standardizing the semantic representation of how an identity was vetted and proofed prior to issuance, this profile empowers Relying Parties (RPs) to execute automated risk decisions based on the rigorousness of the Issuer's identity proofing phase.

---

### Table of Contents
1.  Introduction
2.  Scope
3.  Standardized Parameter Value Registry (Normative)
    3.1. Assurance Namespace
    3.2. Proofing Assurance Classifications
4.  Standardized Claims and Values Registry (Normative)
    4.1. Proofing Context Claims
    4.2. Proofing Assurance Levels
    4.3. Proofing Verification Methods (`check_method`)
5.  Compliance Elements: Provenance & Jurisdictional Assurance
Author's Address

---

### 1. Introduction
To transition to Verifiable Digital Credentials (VDCs), Relying Parties (RPs) require a standardized method to evaluate the confidence level of the initial identity proofing performed by the Issuer. This profile standardizes the claims and values necessary to transport identity proofing assurance data across different formats, allowing RPs to establish a defensible "reasonable belief" regarding the rigor of the vetting process.

### 2. Scope
This specification extends the OpenID Identity Assurance Schema Definition 1.0 by isolating the specific claims, namespaces, and enumerated values associated strictly with the **identity proofing and vetting phase** executed by the Issuing Authority. It excludes post-issuance operational metrics, wallet hardware security assertions, and ongoing revocation checks.

### 3. Standardized Parameter Value Registry (Normative)

#### 3.1. Assurance Namespace
To enable cross-format interoperability of proofing claims, implementations MUST utilize the registered OpenID IDA Assurance namespace: 
* **ISO/IEC 18013-5/7 (CBOR mdoc):** `org.openid.ida.assurance.1`
* **W3C Verifiable Credentials (JSON/SD-JWT):** `assurance_level` property mapped within the credential `@context`.

#### 3.2. Proofing Assurance Classifications
* **`issuance_assurance_classification`**: The specific vetting standard met at the time of issuance (e.g., `status:full_compliance:us:real_id`, `loa:high:eu:eidas`). This serves as the primary integer weight representing the Issuer's proofing rigor.

---

### 4. Standardized Claims and Values Registry (Normative)
To enable true jurisdictional agility, Issuers MUST map local proofing terminology to this standardized registry of claims and enumerated values.

#### 4.1. Proofing Context Claims
These claims establish the regulatory framework and the environmental context under which the identity was originally proofed.

| Claim | Example Parameter Values | Data Type | Description |
| :--- | :--- | :--- | :--- |
| **`context_uri`** | `urn:openid:assurance:us:real_id`, `urn:openid:assurance:eu:eidas` | String | Defines the specific legal or regulatory trust framework governing the initial proofing and vetting. |
| **`presence_equivalence`** | `in_person`, `remote_supervised` | String | Indicates the context of the user's presence during the initial proofing phase. |

#### 4.2. Proofing Assurance Levels
These claims provide the discrete variables required to communicate the confidence established during the identity proofing lifecycle.

| Claim | Example Parameter Values | Data Type | Description |
| :--- | :--- | :--- | :--- |
| **`issuance_assurance_classification`** | `status:full_compliance:us:real_id`, `loa:high:eu:eidas` | String | Asserts the rigor of the vetting process and confidence established during proofing by the Issuer. |
| **`proofing_level`** | `ial:2`, `ip:3` | String | Harmonized assurance level mapping representing the Identity Assurance Level (IAL) or Identity Proofing (IP) level for cross-sector compatibility. |

#### 4.3. Proofing Verification Methods (`check_method`)
Provides the methodology used by the Issuer to validate the identity evidence during the proofing phase.

| Value Category | Example Parameter Values | Description |
| :--- | :--- | :--- |
| **`Proofing`** | `pipp`, `uripp`, `sripp` | **Physical In-Person Proofing** (`pipp`), **Unsupervised Remote** (`uripp`), **Supervised Remote** (`sripp`). |

---

### 5. Compliance Elements: Provenance & Jurisdictional Assurance

**Requirement:** Verify the credential was issued under recognized, government-mandated proofing standards.

**Implementation:** Establish a standardized, format-agnostic namespace (e.g., `org.openid.ida.assurance.1`) that Issuers MUST incorporate natively into the credential to explicitly declare the proofing standards met. 

**RP Action:** RPs SHOULD explicitly request proofing data elements from this namespace via Selective Disclosure during the presentation phase to evaluate the native assurance level of the Issuer's vetting process prior to full PII payload extraction.

---

**Author's Address**

Juliana Cafik  
Independent  
Email: juliana@cafik.ca  
"""
 
