- Feature Name: Grid Supplier Certification
- Start Date: 10/12/21
- RFC PR: https://github.com/hyperledger/grid-rfcs/pull/29

# Table of Contents

- [Summary](0000-certification.md#summary)
- [Motivation](0000-certification.md#motivation)
- [Guide-level Explanation](0000-certification.md#guide-level-explanation)
- [Reference-level 
Explanation](0000-certification.md#reference-level-explanation)
- [Drawbacks](0000-certification.md#drawbacks)
- [Rationale and Alternatives](0000-certification.md#rationale-and-alternatives)
- [Prior Art](0000-certification.md#prior-art)
- [Unresolved Questions](0000-certification.md#unresolved-questions)

# Summary
[summary]: #summary


This RFC introduces certification workflows at a supplier level.
It will primarily consist of translating the Sawtooth transactions defined in 
https://github.com/target/consensource-processor into a Grid smart contract.

[ConsenSource](https://github.com/target/consensource) is a blockchain platform 
for verifying certified suppliers. It serves as a 
transparent platform for displaying certifications and audit data 
between parties such as standards bodies, certifying bodies, and suppliers.

In order to utilize Grid workflows, a set of `PermissionAlias` objects needs to 
be
created to support the following persona-specific actions:
- `Certification bodies`
    - Issue certificates
    - Update certificates
- `Standards bodies`
    - Create standards
    - Update standard versions
    - Accredit certifying bodies
- `Suppliers`
    - Manage profile information
    - Create an audit request
    - Update request statues

For the purpose of this RFC, the supplier audit request system will be excluded.

Communicating certification details via email, web portals, or traditional 
methods create uncertainty for sourcing teams across the industry because 
they lack a shared view of the validity of a certificate. 

Siloed views of certificate data make it difficult to source and validate 
the certifications of a supplier. Grid Supplier Certification aims to 
address this challenge by providing a  mechanism for trade partners 
to collaborate on the creation and modification of a purchase order 
with a shared view into the state of the order.


# Motivation
[motivation]: #motivation

Grid Supplier Certification is designed for sharing factory certification 
data between industry participants. The discovery and validation of 
factory-level certifications is a pain point that retailers share across the 
industries. 

The communication of certificate information occurs in various ways today: 
  - Phone
  - Email
  - SMS
  - Websites

As a result, retailers need to independently verify these claims with the 
certifying body that performed the original audit. This process is 
time-consuming, expensive, and error-prone. Discrepancies between systems 
result in costs related to administrative time, product recalls, and more.

Grid Supplier Certification aims to address these pain points by
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
  communicated between organizations, leading to faster response times, 
improved 
  sourcing decisions and production planning, greater customer satisfaction, 
  and visibility into factory information.

- Improved productivity. Less time spent comparing documents and resolving 
  discrepancies means team members can focus on move value-add business 
opportunities.


The design will address the following use cases:

- Sharing of supplier, certification, and standards data on Grid
- Modeling of real-world certification issuance workflows

It is also useful to use certification data as auxiliary data in other supply 
chain solutions. 
For example, in track and trace smart contract, the location of a particular 
factory in a 
supply chain can be extended to include the certificates the factory holds. 
When presenting 
this information later to a sourcing specialist, it makes its easier for them 
to vet potential 
factories. This workflow will introduce a certificate data model that could 
also be extended 
to include product level certifications.


# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation

A certification is a business transaction that specifies the details of a 
supplier’s capabilities. The certification transaction contains information 
such as issuance date, purpose, and expiration.

A simplified certification sequence is as follows:
  - Standards body creates standard `Foobar`
  - Standards body accredits a certifying body to the `Foobar` standard
  - The certifying body performs an audit
  - Certifying body grants the `Foobar` certification to the factory

## Entities

A `PermissionAlias` is defined for standards bodies, certification bodies, and 
suppliers.

`Standards body`

```rs
let alias = PermissionAlias {
   name: “certificate.standard_body”.to_string(),
   permissions: vec![“create_standard”.to_string(), 
“update_standard_version”.to_string(), “accredit_cert_body”.to_string() ],
   transitions: vec![“confirmed”.to_string()]
}
```

`Certification body`

```rs
let alias = PermissionAlias {
   name: “certificate.cert_body”.to_string(),
   permissions: vec![“issue_certification”.to_string(), 
“update_certification”.to_string()],
   transitions: vec![“confirmed”.to_string()]
}
```

`Suppliers`

```rs
let alias = PermissionAlias {
   name: “certificate.supplier”.to_string(),
   permissions: vec![create_supplier.to_string(), update_supplier.to_string()],
   transitions: vec![“confirmed”.to_string()]
}
```

## Actions
This design introduces six smart contract actions.

- **Create supplier:** Create a new supplier 
- **Update supplier:** Update an existing supplier
- **Create standard:** Create a new standard
- **Update standard:** Updates an existing standard by incrementing the version
- **Accredit certifying body:** Accredit a certifying body to a standard
- **Issue certificate:** Issue a certificate to a supplier from an accredited 
certifying body

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


