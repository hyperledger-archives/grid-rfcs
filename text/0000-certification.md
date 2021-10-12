- Feature Name: Certificate
- Start Date: 10/12/21
- RFC PR: 

# Table of Contents

- [Summary](0000-certification.md#summary)
- [Motivation](0000-certification.md#motivation)
- [Guide-level Explanation](0000-certification.md#guide-level-explanation)
- [Reference-level Explanation](0000-certification.md#reference-level-explanation)
- [Drawbacks](0000-certification.md#drawbacks)
- [Rationale and Alternatives](0000-certification.md#rationale-and-alternatives)
- [Prior Art](0000-certification.md#prior-art)
- [Unresolved Questions](0000-certification.md#unresolved-questions)

# Summary
[summary]: #summary


This RFC intends to introduce certification workflows at a factory level.
It will primarily consist of translating the sawtooth transactions defined here: 
https://github.com/target/consensource-processor into a grid smart contract.

[ConsenSource](https://github.com/target/consensource) is a blockchain platform 
for verifying sustainability-certified suppliers. This application serves as a 
transparent platform for displaying industry certifications and audit data 
between parties (certificate accrediting groups, certificate issuers, suppliers, 
etc.).

In order to utilize grid workflows, we'll need to create PermissionAlias's and
support the following actions:
- Certification bodies
    - Issuing a certificate
    - Updating an existing certificate
- Standards bodies
    - Creation of standards
    - Update of standard versions
    - Accredation of a certfying body
- Factories
    - Creating an audit request
    - Updating the status of a request


# Motivation
[motivation]: #motivation

The Grid Certificate implementation is designed for sharing factory certification 
data between participants. Certification is a common concept within supply chains
and could naturally be extended to the product level.

The design will address use cases including:

- Sharing of factories, certificates, and standards data across a network
- Enriching a UX experience that models the real-world workflow of issuing certs

It is also useful to use certification data as auxiliary data in other supply
chain solutions.  For example, in track and trace, the location of a factory can 
be extended to include the certificates the factory holds. When presenting this 
information later to a sourcing specialist, it makes its easier for them to vet
potential factories. Certifications can also be extended to the product level. 
Therefore, certificates becomes a reusable and important component within Grid.

# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation

## Entities


## Transactions


## Permissions



# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## State

### Certificate Representation

### Defined GS1 Properties

# Drawbacks
[drawbacks]: #drawbacks


# Rationale and alternatives
[alternatives]: #alternatives




# Prior Art
[prior-art]: #prior-art


# Unresolved questions
[unresolved]: #unresolved-questions
