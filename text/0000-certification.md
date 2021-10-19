- Feature Name: Grid Factory Certification
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

This RFC intends to introduce a certification workflow for factories.
It will primarily consist of translating the sawtooth transactions defined here: 
https://github.com/target/consensource-processor into a grid smart contract.

[ConsenSource](https://github.com/target/consensource) is a blockchain platform 
for verifying sustainability-certified suppliers. This application serves as a 
transparent platform for displaying industry certifications and audit data 
between parties (certificate accrediting groups, certificate issuers, suppliers, 
etc.).

In order to utilize grid workflow, we'll need to create PermissionAlias's and
support the following actions:
- Certification bodies
    - Issuing a certificate
    - Updating an existing certificate
- Standards bodies
    - Creation of standards
    - Update of standard versions
    - Accredation of a certifying body
- Factories
    - Creating an audit request
    - Updating the status of a request

For the purpose of this RFC the factory audit request system will be excluded.

Communicating certification details via email, web portals, or traditional 
methods creates uncertainty for sourcing teams accross the industry because 
they lack a shared view of a certificates validity. Siloed views of certificate
data create difficulty when sourcing and validating factory certifications. 
Hyperledger Grid Certification aims to address this challenge by providing a 
mechanism for trade partners to collaborate on the creation and modification of 
a purchase order, all while enjoying a shared view into the state of the order.


# Motivation
[motivation]: #motivation

The Grid Factory Certification implementation is designed for sharing factory 
certification data between industry participants. The validation of factory
level certifications is pain point retailers share accross the industry. This 
communication of certificate information occurs in various ways today: by 
phone, email, SMS, websites, etc. The primary pain point includes the discovery
and verification of the certification claims for a given factory. In practice, 
each brand and retailer ends up independently verifying these claims with the 
certifying body that performed the original audit. This process is 
time-consuming, expensive, and error prone. Discrepancies between systems 
result in costs related to administrative time, product recalls, and more.

The Grid Factory certification concept aims to address these pain points by
offering a common industry solution for sharing factory certification 
information between participants on Grid. Trade partners have the option to 
integrate this Grid component with existing systems of record.

Expected outcomes include: 

- Improved cost efficiency. An organization’s financial results can benefit 
from a reduction in administrative time spent manually validating factories 
certifications authenticity.

- Improved transaction accuracy. Automated sharing of factory certification 
data can reduce errors that stem from manual data entry.

- Increased speed. Changes in a factories certification status can be quickly 
communicated between organizations, leading to faster response times, improved 
sourcing decisions and production planning, greater customer satisfaction, and 
visibility into factory information.

- Improved productivity. Less time spent comparing documents and resolving 
discrepancies means team members can focus on move value-add business 
opportunities.

The design will address use cases including:

- Sharing of factories, certificates, and standards data on grid
- Enriching a UX experience that models the real-world workflow of issuing certs

It is also useful to use certification data as auxiliary data in other supply
chain solutions. For example, in track and trace smart contract, the location 
of a particular factory in a supply chain can be extended to include the 
certificates the factory holds. When presenting this information later to a 
sourcing specialist, it makes its easier for them to vet potential factories. 
This workflow will introduce a certificate data model that could also be 
extended to include product level certifications. 


# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation

A certification is a business transaction that specifies the details of a 
factories capabilities. The transaction contains information such as the when 
the certificate was issued, what the certificate is for, and when the 
certificate expires.

In the simplest business scenario, a standards body creates a standard, then 
the standards body accredits a certifying body. Now that accredited certifying 
body can go and audit a factory offline. Then that certifying body can issue a 
certificate to that factory for all to see on the Splinter circut. In more 
complex business scenarios, a factory can request to be audited, and manage the
status of their request.

## Entities

We will need to define `PermissionAlias`'s for standards bodies, certification bodies, factories.

Standards body:
```
let alias = PermissionAlias {
   name: “certificate.standard_body”.to_string(),
   permissions: vec![“create_standard”.to_string(), “update_standard_version”.to_string(), “accredit_cert_body”.to_string() ],
   transitions: vec![“confirmed”.to_string()]
}
```

Certification body:
```
let alias = PermissionAlias {
   name: “certificate.cert_body”.to_string(),
   permissions: vec![“issue_certification”.to_string(), “update_certification”.to_string()],
   transitions: vec![“confirmed”.to_string()]
}
```

Factories
```
let alias = PermissionAlias {
   name: “certificate.factory”.to_string(),
   permissions: vec![create_factory.to_string(), update_factory.to_string()],
   transitions: vec![“confirmed”.to_string()]
}
```

## Actions
This design introduces six smart contract actions that, with the appropriate 
permissions, a user of the system may perform. An action represents a change to 
the shared state of a certification.

- Create factory. Allows for the creations of a new factory
- Update factory. Allows for the modification of an existing factory profile
- Create standard. Allows for the creation of a new standard.
- Update standard. Allows for the modification of an existing standard. This 
will create a new version of the same standard.
- Accredit certifying body. Allows a certifying body to issue certificates for 
a given standard.
- Issue certificate. Allows a certifying body to issue a certificate to a factory.

## Transactions

## Permissions


# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## State

# Drawbacks
[drawbacks]: #drawbacks


# Rationale and alternatives
[alternatives]: #alternatives


# Prior Art
[prior-art]: #prior-art
- [Grid Workflow RFC](https://github.com/hyperledger/grid-rfcs/pull/24)
- [Grid Identity RFC](https://github.com/hyperledger/grid-rfcs/pull/23)

# Unresolved questions
[unresolved]: #unresolved-questions
