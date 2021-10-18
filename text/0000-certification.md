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
    - Manage their profile information
    - Creating an audit request
    - Updating the status of a request

For the purpose of this RFC the factory actions will be excluded.

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

A certification is a business transactions that specifies the details of a factories 
capabilities. The transaction contains information such as the when the certificate
was issued, what the certificate is for, and when the certificate expires.

In the simplest business scenario, a standards body creates a standard, then the
standards body accredits a certifying body. Now that accredited certifying body 
can go and audit a factory offline, then issue a certificate to that factory for
all to see on the Splinter network. In more complex business scenarios, a factory
can request to be audited, manage the status of their request, and profile data.

## Entities

## Actions
This design introduces four smart contract actions that, with the appropriate 
permissions, a user of the system may perform. An action represents a change to 
the shared state of a certification.

- Create standard. Allows for the creation of a new standard.
- Update standard. Allows for the modification of an existing standard. This will
create a new version of the same standard.
- Accredit certifying body. Allows a certifying body to issue certificates for a 
given standard.
- Issue certificate. Allows a certifying body to issue a certificate to a factory.


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
