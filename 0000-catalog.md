- Feature Name: Catalog
- Start Date: 2019-07-12
- RFC PR: 
- Hyperledger Grid Issue: [GRID-95](https://jira.hyperledger.org/browse/GRID-95)

# Summary
[summary]: #summary

This RFC proposes a generic implementation for a Hyperledger Grid *Catalog*. The
implementation of *Catalog* will materialize as an assortment of grid products
that can be shared with one or more organizations, as well as implementation of
catalog related functions. The Catalog on grid will contain 5 fields. A
**catalog_id**, a **catalog_owner**, an **expiry_date**, a **Name** for the 
catalog, and a repeated field of PropertyValues for custom fields.

In addition any grid product added to a catalog will need to contain a
***status*** field and can include an optional ***prices*** field, which will be 
enforced by the catalog_product grid schema.

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

A **catalog** is a high-level construct that products adhering to the 
"catalog_product" schema can reference as an additional field.  A catalog can be 
referenced, or used to do business in a supply chain. A catalog is referenced by
a "catalog_product" via its **catalog_id**. A catalog also has an **owner** (the 
organization that creates the catalog).  It also contains an **expiry_date** as 
a mechanism for clients to exipire the catalog. Also a **Name** for the catalog. 

A catalog, like a product, can also have one or more **properties**.  Properties
are described in the Grid Primitives RFC.  A *property namespace* contains
multiple *property schemas*.  A property schema associates a name (such as
"length") with a data type (such as integer).  

## Transactions

Catalogs are managed by submitting transactions to Hyperledger Grid, which will
process them with the Grid Catalog smart contract. The following transactions
are supported:

**catalog actions:**
* CatalogCreate - create a catalog and store it in state
* CatalogDelete - removes a catalog from state
* CatalogReplicate - creates a full copy of a catalog, or a sub catalog from 
catalog_products that already exist in state

**catalog_product operations:**
* AddProductsToCatalog - adds catalog_product(s) specified by their product_id 
to the catalog (as a list)
* RemoveProductsFromCatalog - removes catalog_product(s) from a catalog (as a list)
* ActivateProduct - toggles the "status" of a catalog_product to an active state
* DeactivateProduct - toggles the "status" of a catalog_product to an inactive state
* DiscontinueProduct - removes a catalog_product from all catalogs it's in

The catalog_product operations scope will be local and global. Meaning 
AddProductsToCatalog will support adding products to a single, multiple, or all 
catalogs that an organization has. Same idea RemoveProductsFromCatalog, 
ActivateProduct, DeactiveProduct.

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

*Disclaimer: CatalogDelete is a potentially hazardous operation and needs to be 
done with care.*

Replication of a catalog is restricted to agents acting on behalf of an
organization. The purpose of replication is to enable agents to duplicate a 
catalog, or a subset of "catalog_products" in state with the intention of 
modifying the catalog_products to fit a different scheme. It reduces the 
overhead needed to recreate a catalog. Generally organizations have a single 
"master" catalog that they use to build "sub-catalogs" customized for their 
business partners. 

To perform any of the **catalog actions** agents will need the following 
permissions:
```
CatalogCreate: Agent from org needs the "can_create_catalog" permission
CatalogDelete: Agent from org needs the "can_delete_catalog" permission
CatalogReplicate: Agent from org needs the "can_duplicate_catalog" permission
```

In the case of **catalog operations** agents from the org they represent will
need the operational permission to perform the desired operation on a Grid
Catalog.
```
AddProductsToCatalog: Agent from org needs the "can_add_products_to_catalog" permission
RemoveProductsFromCatalog: Agent from org needs the "can_remove_products_from_catalog" permission
ActivateProduct: Agent from org needs the "can_activate_product_in_catalog" permission
DeactivateProduct: Agent from org needs the "can_deactivate_product_in_catalog" permission
DiscontinueProduct: Agent from org needs the "can_discontinue_product_in_catalog" permission
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## State

### Catalog Representation

The primary object stored in state is **Catalog**, which consists of a
**catalog_id** (hash of the catalog_name), an **owner** (org_id 
compatible w/Pike), an **expiry_date** (unix timestamp), a **Name**, and a 
repeated field of **PropertyValues**.  The properties available are defined by 
the Grid Property Schema Transaction Family and are restricted to the fields 
and rules of the GS1 Catalog schema (which will be defined at a later time).  
Transactions which are responsible for setting product state values must ensure 
that those properties conform with the requirements of the GS1 Catalog Property 
Schema (to be defined at a later time).

``` 
message Catalog { 
    string catalog_id = 1; 
    string owner = 2; 
    string expiry_date = 3; 
    string name = 4; 
    repeated PropertyValue properties = 5; 
} 
```

### Referencing a Catalog

Catalogs are uniquely referenced by their catalog_id. For example:

``` 
get_catalog(catalog_id) // hash(catalog_name) 
set_catalog(catalog_id, catalog) // hash(catalog_name), catalog 
```

### Catalog Addressing in the Merkle-Radix State System

In order to uniquely locate Catalogs in the Merkle-Radix state system, an
address must be constructed which identifies the storage location of the Catalog
representation.

All Grid addresses are prefixed by the 6-hex-character namespace prefix
"621dee",  Catalogs are further prefixed under the Grid namespace with reserved
enumerations of "03" ("00", "01", and "02" being reserved for other purposes)
indicating "Grid Catalogs" and an additional org_id. Currently in Grid Product, 
org_ids are a [6-10 digit GS1 company prefix](https://www.gs1-us.info/gs1-company-prefix/). 
To keep the prefix length consistant, any GS1 company prefix less than 10 digits will 
be left padded with the hexidecimal representation of the number 10 ("a") until it is 
10 digits in length. This composite address will enable inexpensive catalog operations.

Therefore, all addresses starting with:

``` 
"621dee" + "03" + "aaaa123456" // org_id (in this case a gs1_company_prefix)
```

are Grid GS1 Catalogs identified by the hash of the catalog name which can be
referenced by a "catalog_product" to group products to a specific catalog. 

The catalog_id format consists of 15-digit "alphanumeric string" which include 
a fixed amount of internal "0" padding.  After the 18-hex-characters that are 
consumed by the grid namespace prefix, the catalog, and gs1_company_prefix there 
are 52 hex characters remaining in the address.  The 15 digits of the catalog_id can 
be left padded with 60-hex-character zeroes and right padded with 
2-hex-character zeroes to accommodate potential future storage associated with 
the GS1 Catalog representation, for example:

``` 
"621dee" + "03" + "123456" + "000000000000000000000000000000000000000" +
15-character "numeric string" catalog_id + "00" // catalog_id == hash(catalog_name) 
```

Using SHA-3 from the rust-crypto crate we can tie the catalog_id to 
catalog_name to prevent duplication of catalogs when invoking the 
catalog_replicate action.  
```
extern crate crypto;
use self::crypto::digest::Digest;
use self::crypto::sha3::Sha3;

fn main(){
  // create a SHA3-256 object
  let mut hasher = Sha3::sha3_256();

  // write input message
  hasher.input_str("catalog_name ");

  // read hash digest
  let hex = hasher.result_str();
  let hex_15 = &hex[0..15];

  println!("SHA3 Hash is: {:?}", hex);
  println!("SHA3 Hash (15): {:?}", hex_15);
}

>> SHA3 Hash is: 846e4a25c881f5643b4fb4a775fc679ef2dbf44e5c67f48ba3573ccb9ddcaca1
>> SHA3 Hash (15): 846e4a25c881f56
```

The full catalog address would look like:
``` 
"621dee03123456000000000000000000000000000000000000000846e4a25c881f5600" 
```


### Catalog Product Addressing in the Merkle-Radix State System

The Grid GS1 Catalog Product state address will be prefixed with the Grid 
namespace of "621dee", in addition to "02" (product namespace), "01" for (gs1 
product namespace), and the catalog_id.

Therefore, all addresses starting with:

``` 
"621dee" + "02" + "01 + "846e4a25c881f56" // catalog_id
```

are Grid GS1 Catalog Products and are expected to contain the standard 
attributes of a Grid GS1 Product as well as the extra PropertyValues that 
are defined in the  catalog product schema. These PropertyValues are in 
addition to the expected PropertyValues of a [Grid GS1 Product](https://github.com/hyperledger/grid-rfcs/blob/fbedec06d70b16492fea9f6b1e87146c5fc56771/0000-product.md).

GTIN formats consist of 14-digit “numeric strings” which include some amount of
internal “0” padding depending on the specific GTIN format (GTIN-12,
GTIN-13, or GTIN-14).  After the 12-hex-characters that are consumed by the grid
namespace prefix, the product namespace prefix, GS1 namespce prefix, and catalog 
product namespace prefix, there are 58 hex characters remaining in the address.  
The 12 to 14 digits of the GTIN can be left padded with 40 to 42-hex-character zeroes and 
right padded with 2-hex-character zeroes to accommodate potential future storage 
associated with the GS1 Product representation, for example:

``` 
“621dee” + “02” + “01” + "846e4a25c881f56" + “00000000000000000000000000000” 
+ 14-character “numeric string” product_id + “00” // product_id == GTIN 
```

A full Catalog Product address using the example GTIN from https://www.gtin.info/ would therefore be:

``` 
“621dee0201846e4a25c881f56000000000000000000000000000000001234560001200” 
```

### Catalog Product Schema Definition

This schema defines the additional attributes required of a grid product, such that 
it may be considered a "catalog product". Catalog products should be grouped with 
other catalog products. Once grouped, the grid catalog can be shared with other 
participants in a grid network.


Any grid product added to a catalog will need to contain a _required_ ***status*** 
field and can include an _optional_ ***prices*** field, which will be enforced by 
the catalog_product grid schema.

```
Schema(
    name="Catalog Product",
    description="A grid product that can be grouped with other products via grid catalog",
    owner = "Target"
    properties=[
        PropertyDefinition(
            name="catalog_id",
            data_type=PropertyDefinition.DataType.STRING,
            description="The the id of the catalog this 'catalog product' belongs to",
            required=True
        ),
        PropertyDefinition(
            name="status",
            data_type=PropertyDefinition.DataType.ENUM,
            description="The current state of the catalog product",
            enum_options=["ACTIVE", "INACTIVE"],
            required=True
        ),
        PropertyDefinition(
            name="price",
            data_type=PropertyDefinition.DataType.STRING,
            description="The price of the catalog product"
            required=False
        )])
```

In order to store a catalog product in global state, we will extend from the 
definition of a grid product with several fixed fields, namely identifiers, as 
well as the set of dynamic property values that will conform to the catalog 
product schema definition.

```
message Product { 
    enum ProductNamespace { 
        UNSET_NAMESPACE = 0; 
        GS1 = 1; 
    }
    
    ProductNamespace product_namespace = 1; 
    string product_id = 2; 
    string owner = 3;
    repeated PropertyValue properties = 4; 
}
```

The catalog smart contract would then be responsible for validating the 
properties field against the CatalogProduct schema at run-time.

A catalog product entry would inherit from a Grid Product. In addition to 
the properties definied in the CatalogProduct schema. The data model 
would look like this:
```
CatalogProduct(
    product_id="gtin",
    product_namespace="GS1"
    owner='Target',
    properties=[
        PropertyDefinition(
            name="catalog_id",
            data_type=PropertyDefinition.DataType.STRING,
            string_value="846e4a25c881f56"
        ),
        PropertyValue(
            name="status",
            data_type=PropertyDefinition.DataType.ENUM,
            enum_value=1, # ACTIVE (default)
        ),
        PropertyValue(
            name="price"
            data_type=PropertyDefinition.DataType.STRING,
            string_value="1-10:$50, 10-50:$40, 50+:$30"  # price windows
        ),
    ])
```

## Transaction Payload and Execution (catalog operations)

CatalogPayload Transaction
CatalogPayload contains an action enum and the associated action payload. 
This allows for the action payload to be dispatched to the appropriate logic.
Only the defined actions are available and only one action payload should be 
defined in the CatalogPayload.
```
message CatalogPayload { 
    enum Actions { 
        UNSET_ACTION = 0; 
        CATALOG_CREATE = 1; 
        CATALOG_DELETE = 2; 
        CATALOG_REPLICATE = 3; 
    }

    Action action = 1;

    // Approximately when transaction was submitted, as a Unix UTC timestamp
    uint64 timestamp = 2;

    CatalogCreateAction catalog_create = 3; 
    CatalogDeleteAction catalog_delete = 4; 
    CatalogReplicateAction catalog_replicate = 5; 
} 
```
### CatalogCreateAction
CatalogCreateAction adds a new catalog to state. The transaction should be submitted by an agent, which is identified by its signing key, acting on behalf of the organization that corresponds to the owner in the create transaction. (Organizations and agents are defined by the Pike smart contract.)

message CatalogCreateAction { 
    // GS1 Company Prefix from owner + catalog_id are use as 
    // a composite key for determining the state address
    string owner = 1;
    string catalog_id = 2;
    string name = 3;
    string expiry_date = 4;
    repeated PropertyValues properties = 4; 
} 

Validation requirements:

- If a catalog with catalog_id already exists the transaction is invalid.
- The signer of the transaction must be an agent in the Pike state and must belong to an organization in Pike state, otherwise the transaction is invalid.
- The agent must have the permission can_create_catalog for the organization, otherwise the transaction is invalid.
- If the product_namespace is GS1, the organization must contain a GS1 Company Prefix in its metadata (gs1_company_prefixes), and the prefix must match the company prefix in the product_id, which is a GTIN if GS1, otherwise the transaction is invalid.
- The properties must be valid for the product_namespace. For example, if the product is GS1 product, its properties must only contain properties that are included in the GS1 Schema. If it includes a property not in the GS1 Schema the transaction is invalid.

AddProductsToCatalog
RemoveProductsFromCatalog
ActivateProduct
DeactivateProduct
DiscontinueProduct

# Rationale and alternatives
[alternatives]: #alternatives

The design of catalog in this manner leaves it extensible for domain specific
use. The concept of price has been discussed and it's been decided to include 
the notion of price within products added to a catalog. A simple implementation 
of price is fine and currently serves as a placeholder, while a more robust 
standard of modeling price data can be implemented from a future pricing-related
RFC. A GS1 pricing standard that can be leveraged in the writing of such an RFC
would be [GS1 Price 
Sync](https://www.gs1.org/docs/gdsn/3.1BMS_Price_Sync_r3p1p3_i1p3p5_23May2017.pdf).

Ideally the catalog_id should be a composite key dervied from the org_id and perhaps the catalog_name. In this current implementation catalogs/catalog_products simply maintain references to eachother. There is no concept of "list of products" within a catalog. This design makes catalog actions/operations less expensive to perform. 

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
