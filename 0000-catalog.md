- Feature Name: Catalog
- Start Date: 2019-07-12
- RFC PR: 
- Hyperledger Grid Issue: [GRID-95](https://jira.hyperledger.org/browse/GRID-95)

# Summary
[summary]: #summary

This RFC proposes a generic implementation for a Hyperledger Grid *Catalog*. The
implementation of *Catalog* will materialize as an assortment of grid products
that can be shared with one or more organizations, as well as implementaion of
catalog related functions. The Catalog on grid will contain 6 fields. A
**catalog_id**, a **catalog_owner** a list of grid **product_ids**, an
**expiry_date**, an *optional* **Name** for the catalog, and additional
key:value pairs for custom fields.

In addition any grid product added to a catalog will need to contain a
***status*** field and can include an optional ***prices*** field.

# Motivation
[motivation]: #motivation

The Grid Catalog implementation is designed for sharing an assortment of
products between organizations. Each product in a catalog will contain a number
of master data elements. Using the Grid Catalog, organizations can share
collections of related/unrelated products with other participants in their
network. 

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

## Entities

A **catalog** is a collection of product_ids from
[products](https://github.com/hyperledger/grid-rfcs/pull/5) that can be shared,
referenced, or used to do business in a supply chain. A catalog is referenced
using a **catalog_id**. It has an **owner** (the organization that creates the
catalog).  It also contains an **expiry_date** as a mechanism for clients to
exipire the catalog. Also an optional **Name** for the catalog. 

A catalog, like a product, can also have one or more **properties**.  Properties
are described in the Grid Primitives RFC.  A *property namespace* contains
multiple *property schemas*.  A property schema associates a name (such as
"length") with a data type (such as integer).  

## Transactions

Catalogs are managed by submitting transactions to Hyperledger Grid, which will
process them with the Grid Catalog smart contract. The following transactions
are supported:


**Catalog actions:**
* CatalogCreate - create a catalog and store it in state
* CatalogDelete - removes a catalog from state
* CatalogReplicate - creates another copy of a catalog that already exists in state

**Catalog product operations:**
* AddProductsToCatalog - adds a list of products specified by their product_id to the catalog 
* RemoveProductsFromCatalog - removes a product or products from a catalog
* ActivateProduct - toggles the "status" of a product to an active state
* DeactivateProduct - toggles the "status" of a product to an inactive state
* DiscontinueProduct - removes a product from all catalogs it's in

## Permissions

Creation of a Grid Catalog is restricted to agents acting on behalf of the
organization in the catalog's owner field.  An organization must have the GS1
company prefix in the "gs1_company_prefixes" metadata field for its Pike
organization.  (Organization-level metadata will need to be implemented in
Pike.) When a catalog is created, its owning organization is stored with the
catalog in an "owner" field.

Deletion of a catalog is restricted to agents acting on behalf of the
organization in the catalog's owner field.  A setting will turn off deletion
entirely, with the intent that it could be enabled/disabled by a Grid
administrator.  Disabling delete is useful because external systems may have
references to these products and deleting them could leave dangling references.

Replication of a catalog is restricted to agents acting on behalf of an
organization. The purpose of replication is to enable agents to duplicate a
catalog in state with the intention of modifying the catalog for a different
specification. It reduces the overhead needed to recreate a catalog only to
add/modify few or many attributes of a catalog, or the products associated.

In the case of **catalog operations** agents from the org they represent will
need the operational permission to perform the desired operation on a Grid
Catalog.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

The primary object stored in state is **Catalog**, which consists of a
catalog_id ([UUID](https://crates.io/crates/uuid)), an owner (org_id compatible
w/Pike), an expiry_date (unix timestamp), an *optional* **Name**, and a list of
property key/value pairs.  The properties available are defined by the Grid
Property Schema Transaction Family and are restricted to the fields and rules of
the GS1 Catalog schema (which will be defined at a later time). Transactions
which are responsible for setting product state values must ensure that those
properties conform with the requirements of the GS1 Catalog Property Schema (to
be defined at a later time).

``` 
message Catalog { 
    string catalog_id = 1; 
    string owner = 2; 
    string expiry_date = 3; 
    name = 4; 
    repeated string product_ids = 5;
    repeated PropertyValue properties = 6; 
} 
```

### Referencing a Catalog

Products are uniquely referenced by their product_id and product_namespace.  For
GS1, Products are referenced by the GTIN identifier. For example:

``` 
get_catalog(catalog_id) // UUID 
set_catalog(catalog_id, catalog) // UUID, catalog 
```

### Product Addressing in the Merkle-Radix State System

In order to uniquely locate Catalogs in the Merkle-Radix state system, an
address must be constructed which identifies the storage location of the Catalog
representation.

All Grid addresses are prefixed by the 6-hex-character namespace prefix
"621dee",  Catalogs are further prefixed under the Grid namespace with reserved
enumerations of "03" ("00", "01", and "02" being reserved for other purposes)
indicating "Catalos" and an additional "01" indicating "GS1 Catalog".

Therefore, all addresses starting with:

``` 
"621dee" + "03" + "01" 
```

are Grid GS1 Catalogs identified by a UUID and are expected to contain a list of product_ids with the product having additional attributes that conform with the GS1 product catalog schema.

UUID formats consist of 36-digit "alphanumeric strings" which include a fixed amount of internal "0" padding.  After the 10-hex-characters that are consumed by the grid namespace prefix, the catalog, and GS1 prefixes, there are 60 hex characters remaining in the address.  The 36 digits of the GTIN can be left padded with 24-hex-character zeroes and right padded with 2-hex-character zeroes to accommodate potential future storage associated with the GS1 Catalog
representation, for example:

``` 
"621dee" + "03" + "01" + 0000000000000000000000 +
36-character "numeric string" catalog_id + "00" // catalog_id == UUID 
```

Using the v5 constructer in the UUID crate we can tie the catalog_id to catalog_name will prevent duplication of catalogs when invoking the catalog_replicate action. 
```
use uuid::Uuid;

fn main() {
    let my_uuid = Uuid::new_v5(&Uuid::new_v4(), b"catalog_name");
    println!("{}", my_uuid);
}

>> 9db5fcec-dcab-5b04-b31b-765ccbf2bc3a
```

The full catalog address would look like:
``` 
"621dee030100000000000000000000009db5fcec-dcab-5b04-b31b-765ccbf2bc3a00" 
```


# Rationale and alternatives
[alternatives]: #alternatives

The design of catalog in this manner leaves it extensible for domain specific
use. The concept of price has been discussed and it's been decided to include a
notion of price within products added to a catalog. A simple implmentation of
price is fine and currently serves as a placeholder, while a more robust
standard of modeling price data can be implemented from a future pricing-related
RFC. A GS1 pricing standard that can be leveraged in the writing of such an RFC
would be [GS1 Price Sync](https://www.gs1.org/docs/gdsn/3.1BMS_Price_Sync_r3p1p3_i1p3p5_23May2017.pdf).

Ideally the catalog_id should be dervied from the org_id + hash of the products

# Prior art
[prior-art]: #prior-art

In the distributed ledger space there are not any example implementations of a
product catalog. There are specs and standards around catalogs that can be
leveraged in the design and implementation of a Grid Catalog.

See Ariba's® [catalog interchange format
(CIF)](https://www.essent.com/What-is-a-CIF-Catalog.html) CIF that has grown
into a de facto standard and is widely used by hundreds of thousands of
companies throughout the world.

[GS1's Catalogue Item
Synchronisation](https://www.gs1.org/docs/gdsn/3.1/BMS_GDSN_Catalogue_Item_Sync_r3p1p0_i1_p0_p6_25Aug2015.pdf)
is the process of continuous harmonisation of item information between trading
partners which ensures that the master data is the same in all trading partners
systems.

CIF and GS1's CIS provides a view of current state of product catalogs as well
as the formats and transactions associated with them.

# Unresolved questions
[unresolved]: #unresolved-questions

- Determining the integration with Grid Schema
- Determining schemas for catatlog products
- Determining pricing data model and associated function in a future RFC
