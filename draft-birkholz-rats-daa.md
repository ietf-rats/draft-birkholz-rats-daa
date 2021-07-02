---
title: Direct Anonymous Attestation for the Remote Attestation Procedures Architecture
abbrev: DAA for RATS
docname: draft-birkholz-rats-daa-latest
wg: RATS Working Group
stand_alone: true
ipr: trust200902
area: Security
kw: Internet-Draft
cat: info
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes

author:
- ins: H. Birkholz
  name: Henk Birkholz
  org: Fraunhofer SIT
  abbrev: Fraunhofer SIT
  email: henk.birkholz@sit.fraunhofer.de
  street: Rheinstrasse 75
  code: '64295'
  city: Darmstadt
  country: Germany
- ins: C. Newton
  name: Christopher Newton
  org: University of Surrey
  email: cn0016@surrey.ac.uk
- ins: L. Chen
  name: Liqun Chen
  org: University of Surrey
  email: liqun.chen@surrey.ac.uk

normative:
  RFC2119:
  RFC5280:
  RFC8174:

informative:
  I-D.ietf-rats-architecture: RATS
  I-D.ietf-rats-reference-interaction-models: models
  DAA:
    title: Direct Anonymous Attestation
    author:
    - ins: E. Brickell
      name: Ernie Brickell
    - ins: J. Camenisch
      name: Jan Camenisch
    - ins: L. Chen
      name: Liqun Chen
    seriesinfo:
      ACM: >
        Proceedings of the 11rd ACM conference on Computer and Communications Security
      page: 132-145
    date: 2004

--- abstract

This document maps the concept of Direct Anonymous Attestation (DAA) to the Remote Attestation Procedures (RATS) Architecture.
The role DAA Issuer is introduced and its interactions with existing RATS roles is specified.

--- middle

# Introduction

Remote ATtestation procedureS (RATS, {{-RATS}}) describe interactions between well-defined architectural constituents in support of Relying Parties that require an understanding about the trustworthiness of a remote peer.
The identity of an Attester and its corresponding Attesting Environments play a vital role in RATS.
A common way to refer to such an identity is the Authentication Secret ID as defined in the Reference Interaction Models for RATS {{-models}}.
The fact that every Attesting Environment can be uniquely identified in the context of the RATS architecture is not suitable for every application of remote attestation.
Additional issues may arise when Personally identifiable information (PII) -- whether obfuscated or in clear text -- are included in attestation Evidence or even corresponding Attestation Results.
This document illustrates how Direct Anonymous Attestation (DAA) can mitigate the issue of uniquely (re-)identifiable Attesting Environments.
To accomplish that goal, a new RATS role -- the DAA Issuer -- is introduced and its duties as well as its interactions with other RATS roles are specified.

# Terminology

This document uses the following set of terms, roles, and concepts as defined in {{-RATS}}:
Attester, Verifier, Relying Party, Conceptual Message, Evidence, Attestation Result, Attesting Environment. The role of Endorser, also defined in {{-RATS}}, needs to be adapted and details are given below.

Additionally, this document uses and adapts, as necessary, the following concepts and information elements as defined in {{-models}}:
Attester Identity, Authentication Secret, Authentication Secret ID

A PKIX Certificate is an X.509v3 format certificate as specified by {{RFC5280}}.

{::boilerplate bcp14}

# Direct Anonymous Attestation

{{dataflows}} shows the data flows between the different RATS roles involved in DAA.

~~~~
  ************       *************   ************   *****************
  * Endorser *       * Reference *   * Verifier *   * Relying Party *
  ************       * Value     *   *  Owner   *   *  Owner        *
       |             * Provider  *   ************   *****************
       |             *************          |            |
       |                       |            |            |
       |Endorsements           |Reference   |Appraisal   |Appraisal
       |                       |Values      |Policy      |Policy for
       |                       |            |for         |Attestation
       |                       |            |Evidence    |Results
       V                       |            |            |
.-----------------.            |            |            |
|   DAA Issuer    |---------.  |            |            |
'-----------------'         |  |            |            |
  ^          |         Group|  |            |            |
  |          |        Public|  |            |            |
  |Credential|           Key|  |            |            |
  |Request   |              v  v            v            |
  |          |             .----------------------.      |
  |          |          .->|      Verifier        |--.   |
  |          |          |  '----------------------'  |   |
  |          |          |                            |   |
  |          |          |Evidence         Attestation|   |
  |          |          |                     Results|   |
  |          |          |                            |   |
  |          |Credential|                            |   |
  |          |          |                            |   |
  |          v          |                            v   v
  |        .-------------.                     .---------------.
  '--------|  Attester   |                     | Relying Party |
           '-------------'                     '---------------'
~~~~
{: #dataflows title="DAA data flows"}

DAA {{DAA}} is a signature scheme that allows the privacy of users that are associated with an Attester (e.g. its owner) to be maintained.
Essentially, DAA can be seen as a group signature scheme with the feature that given a DAA signature no-one can find out who the signer is, i.e., the anonymity is not revocable.
To be able to sign anonymously, an Attester has to obtain a credential from a DAA Issuer.
The DAA Issuer uses a private/public key pair to generate credentials for a group of Attesters <!-- this could be phrased a bit confusing as below it is stated that the key-pair is used for a group of Attesters --> and makes the public key (in the form of a public key certificate) available to the verifier in order to enable them to validate the Evidence received.

In order to support these DAA signatures, the DAA Issuer MUST associate a single key pair with a group of Attesters <!-- is it clear enough what exactly "a group of Attesters" means? --> and use the same key pair when creating the credentials for all of the Attesters in this group.
The DAA Issuer’s group public key certificate replaces the individual  Attester Identity documents during authenticity validation as a part of the appraisal of Evidence conducted by a verifier.
This is in contrast to intuition that there has to be a unique Attester Identity per device.

For DAA, the role of the Endorser is essentially the same, but they now  provide Attester endorsement documents to the DAA Issuer rather than directly to the verifier. These Attester endorsement documents enable the Attester to obtain a credential from the DAA Issuer.

# DAA changes to the RATS Architecture

In order to enable the use of DAA, a new conceptual message, the Credential Request, is defined and a new role, the DAA Issuer role, is added to the roles defined in the RATS Architecture.

Credential Request:

: An Attester sends a Credential Request to the DAA Issuer to obtain a credential. This request contains information about the DAA key that the Attester will use to  create evidence and together with Attester endorsement information that is provided by the Endorser to confirm that the request came from a valid Attester.

DAA Issuer:

: A RATS role that offers zero-knowledge proofs based on public-key certificates used for a group of Attesters (Group Public Keys) {{DAA}}. How this group of Attesters is defined is not specified here, but the group must be large enough for the necessary anonymity to be assured.

Effectively, these certificates share the semantics of Endorsements, with the following exceptions:

* Upon receiving a Credential Request from an Attester, the associated group private key is used by the DAA Issuer to provide the Attester with a credential that it can use to convince the Verifier that its Evidence is valid.
To keep their anonymity the Attester randomizes this credential each time that it is used. Although the DAA Issuer knows the Attester Identity and can associate this with the credential issued, randomisation ensures that the Attester's  identity cannot be revealed to anyone, including the Issuer.
* The Verifier can use the DAA Issuer's group public key certificate, together with the randomized credential from the Attester, to confirm that the Evidence comes from a valid Attester without revealing the Attester's identity.
* A credential is conveyed from a DAA Issuer to an Attester in combination with the conveyance of the group public key certificate from DAA Issuer to Verifier.

# Additions to Remote Attestation principles

In order to ensure an appropriate conveyance of Evidence via interaction models in general, the following prerequisite considering Attester Identity MUST be in place to support the implementation of interaction models.

Attestation Evidence Authenticity:

: Attestation Evidence MUST be correct and authentic.

: In order to provide proofs of authenticity, Attestation Evidence SHOULD be cryptographically associated with an identity document that is a randomised DAA credential.

The following information elements define extensions for corresponding information elements defined in {{-models}} and that are vital to all types of reference interaction models.
Varying from solution to solution, generic information elements can be either included in the scope of protocol messages (instantiating Conceptual Messages defined by the RATS architecture) or can be included in additional protocol parameters of protocols that facilitate the conveyance of RATS Conceptual Messages.
Ultimately, the following information elements are required by any kind of scalable remote attestation procedure using DAA with one of RATS's reference interaction models.

Attester Identity ('attesterIdentity'):

: *mandatory*

: In DAA, the Attester's identity is not revealed to the verifier. The Attester is issued with a credential by the DAA Issuer that is randomised and then used to anonymously confirm the validity of their evidence.
The evidence is verified using the DAA Issuer’s group public key.

Authentication Secret IDs ('authSecID'):

: *mandatory*

: In DAA, Authentication Secret IDs are represented by the DAA Issuer’s group public key that MUST be used to create DAA credentials for the corresponding Authentication Secrets used to protect Evidence.

: In DAA, an Authentication Secret ID does not identify a unique Attesting Environment but is associated with a group of Attesting Environments.
This is because an Attesting Environment should not be distinguishable and the DAA credential which represents the Attesting Environment is randomised each time it used.

# Privacy Considerations

Privacy Considerations

# Security Considerations

Security Considerations

# IANA Considerations

We don't yet.

--- back
