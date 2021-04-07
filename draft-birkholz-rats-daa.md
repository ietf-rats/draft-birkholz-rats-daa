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
  turtles:
    title: "Turtles All the Way Down: Foundation, Edifice, and Ruin in Faulkner and McCarthy"
    author:
    - ins: R. Rudnicki
      name: Robert Rudnicki
    seriesinfo:
      The Faulkner Journal: 25.2
      DOI: "10.1353/fau.2010.0002"
    date: 2010

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
Attester, Verifier, Relying Party, Conceptual Message, Evidence, Attestation Result, Attesting Environment

Additionally, this document uses and augments the following concepts and information elements as defined in {{-models}}:
Attester Identity, Authentication Secret, Authentication Secret ID

A PKIX Certificate is an X.509v3 format certificate as specified by {{RFC5280}}.

{::boilerplate bcp14}

# Direct Anonymous Attestation

DAA {{DAA}} is a signature scheme that allows preservation of the privacy of users that are associated with an Attester (e.g. its owner).
Essentially, DAA can be seen as a group signature scheme with the feature that given a DAA signature no-one can find out who the signer is, i.e., the anonymity is not revocable.
To be able to sign anonymously, an Attester has to obtain a credential from a DAA Issuer.
The DAA Issuer uses a private/public key pair to generate a credential for an Attester <!-- this could be phrased a bit confusing as below it is stated that the key-pair is used for a group of Attesters --> and makes the public key (in the form of a public key certificate) available to the verifier in order to enable them to validate the DAA signature obtained as part of the Evidence.

In order to support these DAA signatures, the DAA Issuer MUST associate a single key pair with each group of Attesters <!-- it it clear enough what exactly "a group of Attesters" means? --> and use the same key pair when creating the credentials for all of the Attesters in this group.
The DAA Issuer’s public key certificate used by a group of Attesters replaces the individual  Attester Identity documents during authenticity validation as a part of the appraisal of Evidence conducted by a Verifier.
This is in contrast to intuition that there has to be a unique Attester Identity per device.

<!-- revise: adding the DAA Issuer role independent of the Endorser role might be preferable? --> This document extends the duties of the Endorser role as defined by the RATS architecture with respect to the provision of these Attester Identity documents to Attesters.
The existing duties of the Endorser role and the duties of a DAA Issuer are quite similar as illustrated in the following subsections.

## Endorsers [old section, with maybe useful content]

Via its Attesting Environments, an Attester only generates Evidence about its Target Environments.
After being appraised to be trustworthy, a Target Environment may become a new Attesting Environment in charge of generating Evidence for further Target Environments.
{{-RATS}} explains this as Layered Attestation.
Layered Attestation has to start with an initial Attesting Environment. In essence, there cannot be turtles all the way down {{turtles}}.
At this rock bottom of Layered Attestation, the Attesting Environments are always called Roots of Trust (RoT).
An Attester cannot generate Evidence about its own RoTs by design.
As a consequence, a Verifier requires trustable statements about this subset of Attesting Environments from a different source than the Attester itself.
The corresponding trustable statements are called Endorsements and originate from external, trustable entities that take on the role of an Endorser (e.g., supply chain entities).

# DAA augments to the RATS Architecture

In order to enable the use of DAA, the DAA Issuer role is added to the RATS Architecture role takes on the duties of a DAA Issuer in addition to its already defined duties.
Effectively, these certificates share the semantics of Endorsements, with the following exceptions:

DAA Issuer:

: A RATS role that offers zero-knowledge proofs based on public-key certificates used for a group of Attesters {{DAA}}.

<!-- the following text should be re-used in this section -->

* The associated private keys are used by the DAA Issuer to provide an Attester with a credential that it can use to convince the Verifier that its Evidence is valid.
To keep their anonymity the Attester randomizes this credential each time that it is used.
* The Verifier can use the DAA Issuer's public key certificate, together with the randomized credential from the Attester, to confirm that the Evidence comes from a valid Attester.
* A credential is conveyed from an Endorser to an Attester in combination with the conveyance of the public key certificates from Endorser to Verifier.

The zero-knowledge proofs required cannot be created by an Attester alone -- like the Endorsements of RoTs -- and have to be created by a trustable third entity -- like an Endorser.
Due to that semantic overlap, the Endorser role is augmented via the definition of DAA duties as defined below. This augmentation enables the Endorser to convey trustable third party statements both to Verifier roles and Attester roles.

# Additions to Remote Attestation principles

revise: the following section content is included to derive new section content]

In order to ensure an appropriate conveyance of Evidence via interaction models in general, the following set of prerequisites MUST be in place to support the implementation of interaction models:

## Attester Identity [exemplary proposal for new structure]

<!-- : The provenance of Evidence with respect to a distinguishable Attesting Environment MUST be correct and unambiguous. -->

 An Attester Identity MAY be a unique identity, it MAY be included in a zero-knowledge proof (ZKP), or it MAY be part of a group signature, or it MAY be a randomised DAA credential.

Attestation Evidence Authenticity:

: Attestation Evidence MUST be correct and authentic.

: In order to provide proofs of authenticity, Attestation Evidence SHOULD be cryptographically associated with an identity document (e.g. an PKIX certificate or trusted key material, or a randomised DAA credential), or SHOULD include a correct and unambiguous and stable reference to an accessible identity document.

<!-- # Generic Information Elements -->

This section defines the information elements that are vital to all kinds interaction models.
Varying from solution to solution, generic information elements can be either included in the scope of protocol messages (instantiating Conceptual Messages) or can be included in additional protocol parameters or payload.
Ultimately, the following information elements are required by any kind of scalable remote attestation procedure using one or more of the interaction models provided.

Attester Identity ('attesterIdentity'):

: *mandatory*

: A statement about a distinguishable Attester made by an Endorser without accompanying evidence about its validity - used as proof of identity.

: In DAA, the Attester's identity is not revealed to the verifier.
The Attester is issued with a credential by the Endorser that is randomised and then used to anonymously confirm the validity of their evidence.
The evidence is verified using the Endorser’s public key.

Authentication Secret IDs ('authSecID'):

: *mandatory*

: A statement representing an identifier list that MUST be associated with corresponding Authentication Secrets used to protect Evidence.

: In DAA, Authentication Secret IDs are represented by the Endorser (DAA issuer)’s public key that MUST be used to create DAA credentials for the corresponding Authentication Secrets used to protect Evidence.

: Each Authentication Secret is uniquely associated with a distinguishable Attesting Environment. Consequently, an Authentication Secret ID also identifies an Attesting Environment.

: In DAA, an Authentication Secret ID does not identify a unique Attesting Environment but associated with a group of Attesting Environments.
This is because an Attesting Environment should not be distinguishable and the DAA credential which represents the Attesting Environment is randomised each time it used.

--- back
