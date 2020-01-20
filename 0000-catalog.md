- Feature Name: Product Catalog
- Start Date: 2019-07-12
- RFC PR:
- Hyperledger Grid Issue: [GRID-95](https://jira.hyperledger.org/browse/GRID-95)

# Summary

[summary]: #summary

This RFC proposes a generic implementation for a Hyperledger Grid _Product
Catalog_. Product Catalogs will be represented by a unique identifier, which
will be referenced by the _catalog_products_ included in the catalog. That
catalog_id can be used to share a grouping of products that can be shared with 
one or more organizations. The Grid Catalog will contain 4 fields. A 
`catalog_id`, an `owner`, a `name` for the catalog, and a repeated 
field of PropertyValues for custom fields.

In addition, a base catalog product schema is included as a way to demonstrate
enforcement of additional product properties. Any grid product added to a
catalog will adhere to the "catalog_product" schema (or another custom-defined 
schema). The example schema in particular will enforce a required `catalog_id`
field, a required `status` field, an optional `return_policy`, and an 
optional `price` field.

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

A `catalog` is a high-level construct that `catalog_products` can
reference. A `catalog_product` is an instance of a Grid Product with additional 
schema defined attributes specific to the trading partner involved. The 
`catalog_product` schema can be extended to include any number of attributes that 
are required by an organization. A catalog can be referenced, or used to share 
item level information in a supply chain. A catalog is referenced by a 
"catalog_product" via its `catalog_id`. A catalog also has an `owner` (the 
organization that creates the catalog). Also a `name` for the catalog.

A catalog, like a product, can also have one or more `properties`. Properties
are described in the Grid Primitives RFC. A _property namespace_ contains
multiple _property schemas_. A property schema associates a name (such as
"length") with a data type (such as integer). A catalog may want additional 
properties to support executing custom smart contract, or trigger some type of 
check. An example could be an expiry_date for an entire catalog. 

## Transactions

Catalogs are managed by submitting transactions to Hyperledger Grid, which will
process them with the Grid Catalog smart contract. The following transactions
are supported:

**catalog actions:**

- CatalogCreate - create a catalog and store it in state
- CatalogUpdate - update a catalog in state
- CatalogDelete - deletes a catalog from state and associated catalog_products

**catalog_product actions:**

- CatalogProductCreate - creates a catalog_product and stores it in state
- CatalogProductUpdate - updates a catalog_product in state
- CatalogProductDelete - delete a catalog_product from state
- CatalogProductSetStatus - changes the "status" of a catalog_product to an
  active, inactive, or discontinued state

The CatalogProductSetStatus action will accept a parameter list of the
catalogs the action should be performed on. Meaning Activate, Deactivate, or 
Discontinue can be performed on a single, multiple, or all catalogs that an 
organization has.

## Permissions

Creation of a Grid Catalog is restricted to agents acting on behalf of the
organization in the catalog's owner field. An organization must have the GS1
company prefix in the "gs1_company_prefixes" metadata field for its [Pike](https://grid.hyperledger.org/docs/grid/nightly/master/transaction_family_specifications/pike_transaction_family.html) organization.

Deletion of a catalog is restricted to agents acting on behalf of the
organization in the catalog's owner field. A setting will turn off deletion
entirely, with the intent that it could be enabled/disabled by a Grid
administrator. Disabling delete is useful because external systems may have
references to these products and deleting them could leave dangling references.

_Disclaimer: CatalogDelete & CatalogProductDelete are potentially hazardous
operation and need to be done with care._

To perform any of the **catalog actions** agents will need the following
permissions:

```
CatalogCreate: Agent from org needs the "can_create_catalog" permission
CatalogUpdate: Agent from org needs the "can_update_catalog" permission
CatalogDelete: Agent from org needs the "can_delete_catalog" permission
```

To perform any of the **catalog_product actions** agents inherit the same
permissions needed for a Grid Product:

```
CatalogProductCreate: Agent from org needs the "can_create_product" permission
CatalogProductUpdate: Agent from org needs the "can_update_product" permission
CatalogProductDelete: Agent from org needs the "can_delete_product" permission
```

In the case of **catalog operations** agents from the org they represent will
need the operational permission to perform the desired operation on a Grid
Catalog.

```
CatalogProductSetStatus: Agent from org needs the "can_set_catalog_product_status" 
permission
```

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

## State

### Catalog Representation

The primary object stored in state is `Catalog`, which consists of a
`catalog_id` (hash of the catalog_name), an `owner` (org_id
compatible w/Pike), a `name`, and a repeated field of `PropertyValues`. 
The properties available are defined by the Grid Property Schema Transaction 
Family and are restricted to the fields and rules of the GS1 Catalog schema
(which will be defined at a later time). Currently, you can proceed to create 
a catalog that does not adhere to any schema. The GS1 Catalog Schema will be 
defined in a future RFC.

```
message Catalog {
    string catalog_id = 1;
    string owner = 2;
    string name = 3;
    repeated PropertyValue properties = 4;
}
```

### Referencing a Catalog

Catalogs are uniquely referenced by their catalog_id. For example:

```
get_catalog(org_id, catalog_id) // GS1 Company Prefix, hash(catalog_name)
set_catalog(org_id, catalog_id, catalog) // GS1 Company Prefix, hash(catalog_name), catalog
```

### Catalog Addressing in the Merkle-Radix State System

In order to uniquely locate Catalogs in the Merkle-Radix state system, an
address must be constructed which identifies the storage location of the Catalog
representation.

All Grid addresses are prefixed by the 6-hex-character namespace prefix
"621dee", Catalogs are further prefixed under the Grid namespace with reserved
enumerations of "03" ("00", "01", and "02" being reserved for other purposes)
indicating "Grid Catalogs" and an additional org_id. Currently in Grid Product,
org_ids are a [6-10 digit GS1 company prefix](https://www.gs1-us.info/gs1-company-prefix/).
To keep the prefix length consistent, any GS1 company prefix less than 10
digits will be left padded with the hexadecimal representation of the number 10 
("a") until it is 10 digits in length. This composite address will enable 
inexpensive catalog operations.

Therefore, all addresses starting with:

```
"621dee" + "03" + "aaaa123456" // org_id (in this case a gs1_company_prefix)
```

are Grid GS1 Catalogs identified by the hash of the catalog name which can be
referenced by a "catalog_product" to group products to a specific catalog.

The catalog_id consists of a 70-character "alphanumeric string" which includes
a fixed amount of internal "0" padding. After the 18-hex-characters that are
consumed by the Grid namespace prefix, the catalog prefix, and gs1_company_prefix
there
are 52 hex characters remaining in the address. Then the 15 digits of the
catalog_id can
be added with a left padding of 35-hex-character zeroes, and that string is right 
padded with 2-hex-character zeroes to accommodate potential future storage 
associated with the GS1 Catalog representation, for example:

```
"621dee" + "03" + "aaaa123456" + "00000000000000000000000000000000000" + catalog_id 
+ "00" // catalog_id == hash(catalog_name).trunc(15)
```

Using SHA-2 from the rust-crypto crate we can tie the catalog_id to catalog_name
to prevent duplication of catalogs when invoking the catalog_create action.

```
extern crate crypto;
use self::crypto::digest::Digest;
use crypto::sha2::Sha512;

fn sha2_512() {
    // create a SHA2-512 hasher
    let mut hasher = Sha512::new();

    // write input message
    hasher.input_str("catalog_name");

    // read hash digest
    let hex = hasher.result_str();
    let hex_15 = &hex[0..15];

    println!("SHA2-512 Hash is {:?}", hex);
    println!("SHA2-512 Hash (15) {:?}", hex_15);
}

>> SHA2-512 Hash is:
cad4e781e7977123893a42c7e7230e798bf5693f95c04980ba70bbe3504b194227c62e7e32d83ddb
d083979720d7025b2cc9f68ac7b572dc6a607409b2dcdc43
>> SHA2-512 Hash (15): cad4e781e797712
```

The full catalog address would look like:

```
"621dee03aaaa12345600000000000000000000000000000000000cad4e781e79771200"
```

### Referencing a Catalog Product

Catalog products are uniquely referenced by a composite key composed of their
catalog_id and product_id. For example:

```
get_catalog_product(catalog_id, product_id) // hash(catalog_name), product_id
set_catalog_product(catalog_id, product_id, catalog_product) //hash(catalog_name), product_id, catalog_product
```

### Catalog Product Addressing in the Merkle-Radix State System

The Grid GS1 Catalog Product state address will be prefixed with the Grid
namespace of "621dee", in addition to "02" (product namespace), "01" for (gs1
product namespace), and the catalog_id.

Therefore, all addresses starting with:

```
"621dee" + "02" + "01 + "cad4e781e797712" // catalog_id
```

are Grid GS1 Catalog Products and are expected to contain the standard
attributes of a Grid GS1 Product as well as the extra PropertyValues that
are defined in the catalog product schema. These PropertyValues are in
addition to the expected PropertyValues of a [Grid GS1
Product](https://github.com/hyperledger/grid-rfcs/blob/fbedec06d70b16492fea9f6b1e87146c5fc56771/0000-product.md).

GTIN formats consist of 14-digit "numeric strings" which include some amount of
internal "0" padding depending on the specific GTIN format (GTIN-12, GTIN-13, or 
GTIN-14). After the 10-hex-characters that are consumed by the Grid namespace 
prefix, the product namespace prefix, and GS1 namespace prefix. There are 
15-hex-characters consumed by the catalog id, leaving 45 hex characters
remaining in the address.

The 11 to 13 digits of the GTIN can be left padded with 30 to 32-hex-character
zeroes and right padded with 2-hex-character zeroes to accommodate potential
future storage associated with the GS1 Product representation, for example:

```
"621dee" + "02" + "01" + "cad4e781e797712" + "00000000000000000000000000000"
+ 14-character "numeric string" product_id + "00" // product_id == GTIN
```

A full Catalog Product address using the example GTIN from
https://www.gtin.info/ would therefore be:

```
"621dee0201cad4e781e797712000000000000000000000000000000001234560001200"
```

### Catalog Product Schema Definition

This schema defines the additional attributes required of a Grid product, such
that it may be considered a "catalog product". Catalog products should be 
grouped with other catalog products. Once grouped, the Grid catalog can be 
shared with other participants in a Grid network.

Any Grid product added to a catalog (thus becoming a catalog product) will need
to contain a reference fields to the original product (product_id, 
product_namespace, and owner) and a catalog_id. A catalog product schema may 
also be defined to require additional fields for that catalog_product. This 
example schema enforces a _required_ `catalog_id` & `status` field, as 
well as _optional_ `prices` & `return_policy` field.

Example schema:

```
Schema(
    name="Catalog Product",
    description="A Grid product that can be grouped with other products via
Grid catalog",
    owner = "Target"
    properties=[
        PropertyDefinition(
            name="catalog_id",
            data_type=PropertyDefinition.DataType.STRING,
            description="The id of the catalog this catalog product belongs to"
            required=True
        ),
        PropertyDefinition(
            name="status",
            data_type=PropertyDefinition.DataType.ENUM,
            description="The current state of the catalog product",
            enum_options=["ACTIVE", "INACTIVE", "DISCONTINUED"],
            required=True
        ),
        PropertyDefinition(
            name="status_change_reason",
            data_type=PropertyDefinition.DataType.STRING,
            description="The reason for the status change",
            required=True
        ),
        PropertyDefinition(
            name="price",
            data_type=PropertyDefinition.DataType.STRING,
            description="The price of the catalog product"
            required=False
        ),
        PropertyDefinition(
            name="return_policy",
            data_type=PropertyDefinition.DataType.STRING,
            description="The return policy of the of the catalog product"
            required=False
        )])
```

In order to store a catalog product in global state, we will extend from the
definition of a Grid product with several fixed fields, namely identifiers, as
well as the set of dynamic property values that will conform to the catalog
product schema definition.

**Grid Product from the [Product
RFC](https://github.com/hyperledger/grid-rfcs/blob/master/text/0005-product.md):**

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

The catalog_product smart contract would then be responsible for validating the
properties field against the CatalogProduct schema at run-time.

A catalog_product entry would reference Grid Product, in addition to
the properties defined in the CatalogProduct schema. The data model
would look like this:

```
Product(
    product_id="gtin", // Reference from Grid Product
    product_namespace="GS1" // Reference from Grid Product
    owner='Target', // Reference from Grid Product
    properties=[
        PropertyValue(
            name="catalog_id",
            data_type=PropertyDefinition.DataType.STRING,
            string_value="e70fe351d1d096c"
        ),
        PropertyValue(
            name="status",
            data_type=PropertyDefinition.DataType.ENUM,
            enum_value=1, # ACTIVE (default)
        ),
        PropertyValue(
            name="status_change_reason",
            data_type=PropertyDefinition.DataType.STRING,
            string_value="" # Intial value should be an empty string
        ),
        PropertyValue(
            name="price"
            data_type=PropertyDefinition.DataType.STRING,
            string_value="1-10:$50, 10-50:$40, 50+:$30"  # price windows
        ),
        PropertyValue(
            name="return policy"
            data_type=PropertyDefinition.DataType.STRING,
            string_value="30 days from sale"
        ),
        # ... Additional Product Properties
    ])
```

A catalog product, from a data model perspective, is a Grid product. With the 
differentiators being:
- It contains the additional fields defined by the catalog_product schema
- It's stored in the catalog_product state location within the merkle tree

## Transaction Payload and Execution

The following payload contains an action enum and the associated action
payload. This allows for the action payload to be dispatched to the appropriate 
logic. Only the defined actions (enum) are available, and only one Action 
should be performed in a payload.

### CatalogPayload Transaction

```
message CatalogPayload {
    enum Actions {
        UNSET_ACTION = 0;
        CATALOG_CREATE = 1;
        CATALOG_UPDATE = 2;
        CATALOG_DELETE = 3;
        CATALOG_PRODUCT_CREATE = 100;
        CATALOG_PRODUCT_UPDATE = 101;
        CATALOG_PRODUCT_DELETE = 102;
        CATALOG_PRODUCT_SET_STATUS = 103;
    }

    Action action = 1;

    // Approximately when transaction was submitted, as a Unix UTC timestamp
    uint64 timestamp = 2;

    CatalogCreateAction catalog_create = 3;
    CatalogUpdateAction catalog_update = 4;
    CatalogDeleteAction catalog_delete = 5;
    CatalogProductCreateAction catalog_product_create = 100;
    CatalogProductUpdateAction catalog_product_update = 101;
    CatalogProductDeleteAction catalog_product_delete = 102;
    CatalogProductSetStatusAction set_catalog_product_status = 103;
}
```

## Catalog Actions

### CatalogCreateAction

CatalogCreateAction adds a new catalog to state. The transaction should be
submitted by an agent, which is identified by its signing key, acting on behalf 
of the organization that corresponds to the owner in the create transaction. 
(Organizations and agents are defined by the Pike smart contract.)

```
message CatalogCreateAction {
    // GS1 Company Prefix from owner + catalog_id are use as
    // a composite key for determining the state address
    string owner = 1;
    string catalog_id = 2;
    string catalog_name = 3;
    repeated PropertyValues properties = 4;
}
```

Validation requirements:

- If a catalog with catalog_id already exists, the transaction is invalid.
- The signer of the transaction must be an agent in the Pike state and must
  belong to an organization in Pike state, otherwise the transaction is invalid.
- The agent must have the permission can_create_catalog for the organization,
  otherwise the transaction is invalid.

The inputs for CatalogCreateAction must include:

- Address of the Agent submitting the transaction
- Address of the Organization the Catalog is being created for
- Address of the Catalog to be created

The outputs for CatalogCreateAction must include:

Address of the Catalog created

### CatalogUpdateAction

CatalogUpdateAction updates an existing catalog in state. The transaction should be
submitted by an agent, which is identified by its signing key, acting on behalf 
of the organization that corresponds to the owner in the update transaction. 
(Organizations and agents are defined by the Pike smart contract.)

```
message CatalogUpdateAction {
    // GS1 Company Prefix from owner + catalog_id are use as
    // a composite key for determining the state address
    string owner = 1;
    string catalog_id = 2;
    string catalog_name = 3;
    repeated PropertyValues properties = 4;
}
```

Validation requirements:

- If a catalog with catalog_id does not exist, the transaction is invalid.
- The signer of the transaction must be an agent in the Pike state and must
  belong to an organization in Pike state, otherwise the transaction is invalid.
- The agent must have the permission can_update_catalog for the organization,
  otherwise the transaction is invalid.

The inputs for CatalogUpdateAction must include:

- Address of the Agent submitting the transaction
- Address of the Organization the Catalog is being created for
- Address of the Catalog to be updated

The outputs for CatalogUpdateAction must include:

Address of the Catalog to be updated

### CatalogDeleteAction

CatalogDeleteAction deletes an existing catalog from state. The transaction should be
submitted by an agent, which is identified by its signing key, acting on behalf 
of the organization that corresponds to the owner in the delete transaction.
(Organizations and agents are defined by the Pike smart contract.)

```
message CatalogDeleteAction {
    // GS1 Company Prefix from owner + catalog_id are use as
    // a composite key for determining the state address
    string owner = 1;
    string catalog_id = 2;
}
```

Validation requirements:

- If a catalog with catalog_id exists the transaction is valid, otherwise it's
  invalid.
- The signer of the transaction must be an agent in the Pike state and must
  belong to an organization in Pike state, otherwise the transaction is invalid.
- The agent must have the permission can_delete_catalog for the organization,
  otherwise the transaction is invalid.

The inputs for CatalogDeleteAction must include:

- Address of the Agent submitting the transaction
- Address of the Organization the Catalog is being created for
- Address of the Catalog to be deleted

The outputs for CatalogDeleteAction must include:

Address of the Catalog to be deleted

**_NOTE: Deleting a catalog is potentially dangerous operation that could leave 
dangling references and should be done with care._**

## Catalog_Product Actions

### CatalogProductCreateAction

The CatalogProductCreateAction adds a new catalog_product to state. The 
catalog_product references a Grid Product for the shared item level master 
data. The transaction should be submitted by an agent, which is identified by 
its signing key, acting on behalf of the organization that corresponds to the
owner in the create transaction. (Organizations and agents are defined by the 
Pike smart contract.)

```
message CatalogProductCreateAction {
    // catalog_id and product_id are used in deriving the state address
    string catalog_id = 1
    string product_id = 2 // Reference from Grid Product
    // schema defined additional fields
    repeated PropertyValues properties = 4;
}
```

Validation requirements:

- If a catalog_product with catalog_product_id and catalog_id already exists the
  transaction is invalid.
- If the Grid product the catalog_product is referencing does not exist in
  state the transaction is invalid.
- The signer of the transaction must be an agent in the Pike state and must
  belong to an organization in Pike state, otherwise the transaction is invalid.
- The agent must have the permission can_create_product for the organization,
  otherwise the transaction is invalid.
- If the product_namespace is GS1, the organization must contain a GS1 Company
  Prefix in its metadata (gs1_company_prefixes), and the prefix must match the 
  company prefix in the product_id, which is a GTIN (if GS1), otherwise the 
  transaction is invalid.
- The properties must be valid for the catalog_product schema; its properties
  must only contain properties that are included in the catalog_product Schema.

The inputs for CatalogProductCreateAction must include:

- Address of the Agent submitting the transaction
- Address of the Organization the Product is being created for
- Address of the Catalog the catalog_product is related too
- Address of the catalog_product namespace schema and the catalog_product’s
  properties must match
- Address of the Grid Product the catalog_product is referencing

The outputs for CatalogProductCreateAction must include:

- Address of the Catalog_Product created

### CatalogProductUpdateAction

CatalogProductUpdateAction updates an existing product in state. The
transaction should be submitted by an agent, identified by its signing key, 
acting on behalf of an organization that corresponds to the owner in the 
product being updated. (Organizations and agents are defined by the Pike 
smart contract.)

```
message CatalogProductUpdateAction {
    // catalog_id and product_id are used in deriving the state address
    string catalog_id = 1;
    string product_id = 2;
    // this will replace all properties currently defined
    repeated PropertyValues properties = 4;
}
```

Validation requirements:

- If a catalog_product with catalog_product_id does not exist, the transaction
  is invalid.
- The signer of the transaction must be an agent in the Pike state and must
  belong to an organization in Pike state, otherwise the transaction is invalid.
  The owner in the product must match the organization that the agent belongs to,
  otherwise the transaction is invalid.
- The agent must have the permission can_update_product for the organization,
  otherwise the transaction is invalid.
- The properties must be valid for the catalog_product schema; its properties
  must only contain properties that are included in the catalog_product Schema.

The inputs for CatalogProductUpdateAction must include:

- Address of the Agent submitting the transaction
- Address of the Organization the Product is being updated for
- Address of the Product to be updated

The outputs for CatalogProductUpdateAction must include:

- Address of the updated catalog_product

### CatalogProductDeleteAction

CatalogProductDeleteAction removes an existing catalog_product from state. The
transaction should be submitted by an agent, identified by its signing key, 
acting on behalf of the organization that corresponds to the org_id in the 
product being deleted. (Organizations and agents are defined by the Pike smart 
contract.)

```
message CatalogProductDeleteAction {
    // catalog_id and product_id are used in deriving the state address
    string catalog_id = 1;
    string product_id = 2;
}
```

If the Grid setting `grid.product.allow_delete` is set to false, this transaction
is invalid (sys admin setting). The default value for `grid.product.allow_delete`
is true. This setting is stored using the Sawtooth Settings smart contract, more 
information can be found [here](https://sawtooth.hyperledger.org/docs/core/releases/1.0/transaction_family_specifications/settings_transaction_family.html).

Validation requirements:

- If a catalog_product with catalog_product_id does not exist, the transaction is
  invalid.
- The signer of the transaction must be an agent in the Pike state and must
  belong to an organization in Pike state, otherwise the transaction is invalid.
- The owner in the product must match the organization that the agent belongs
  to, otherwise the transaction is invalid.
- The agent must have the permission “can_delete_product” for the organization
  otherwise the transaction is invalid.

The inputs for CatalogProductDeleteAction must include:

- Address of the Agent submitting the transaction
- Address of the Organization the Product is being deleted for
- Address of the catalog_product to be deleted

The outputs for CatalogProductDeleteAction must include:

- Address of the catalog_product to be deleted

**_NOTE: Deleting a catalog_product is potentially dangerous operation that
could leave dangling references and should be done with care._**

## Catalog_Product Operations

### CatalogProductSetStatus

CatalogProductSetStatusAction updates an existing catalog_product in state. The
transaction should be submitted by an agent, identified by its signing key, 
acting on behalf of an organization that corresponds to the owner in the 
product being updated. (Organizations and agents are defined by the Pike smart 
contract.)

The implementation of the CatalogProductSetStatusAction will handle iterating 
through the list of catalog_ids, and update the status of the 
catalog_product(s) accordingly. Having the address of an individual 
catalog_product be a composite key containing the catalog_id and product_id 
enables us to easily update the status across one, all, or specific catalogs.

CatalogProductSetStatusAction does not set the status for multiple 
catalog_products. It will only change the status of a single 
catalog_product. The nuance being that this status change can be reflected in 
as many, or as few catalogs an entity desires.

This flexibility is to enable support for use cases like:

- I am discontinuing a catalog_product for entity A, B, C, but not for retailer D.
- I want to discontinue catalog_products for all entity A, B, C, and D.
- I am marking a catalog_product as active for entity A and B, but not C or D just yet.
- I am making an active catalog_product inactive for entity A and B, but not C or D.

```
message CatalogProductSetStatusAction {
    enum Status {
        INACTIVE = 0;
        ACTIVE = 1;
        DISCONTINUED = 2;
    }
    // catalog_id and product_id are used in deriving the state address
    repeated string catalog_ids = 1;
    string catalog_product_id = 2;
    Status catalog_product_status  = 4;
    // Reason for the change
    string status_change_reason = 5;
}
```

Validation requirements:

- The catalog_product must be able to be updated or the transaction is invalid 
(i.e. meaning the status is not "DISCONTINUED")
- If any catalog_product with product_id does not exist, the transaction is
  invalid.
- The signer of the transaction must be an agent in the Pike state and must
  belong to an organization in Pike state, otherwise the transaction is invalid.
- The owner in the product must match the organization that the agent belongs 
  to, otherwise the transaction is invalid.
- The agent must have the permission can_update_product for the organization,
  otherwise the transaction is invalid.
- The properties must be valid for the catalog_product schema; its properties
  must only contain properties that are included in the catalog_product Schema.

The inputs for CatalogProductSetStatusAction must include:

- Address of the Agent submitting the transaction
- Address of the Organization the catalog_product(s) is/are being updated for
- Address of the catalog_product(s) to be updated

The outputs for CatalogProductSetStatusAction must include:

- Address of the updated catalog_product(s)

# Rationale and alternatives

[alternatives]: #alternatives

The design of catalog in this manner leaves it extensible for domain specific
use. The concept of price has been discussed and it's been decided to include
the notion of price within products added to a catalog. A simple implementation
of price currently serves as a placeholder, while a more robust
standard of modeling price data can be implemented from a future pricing-related
RFC. A GS1 pricing standard that can be leveraged in the writing of such an RFC
would be [GS1 Price
Sync](https://www.gs1.org/docs/gdsn/3.1BMS_Price_Sync_r3p1p3_i1p3p5_23May2017.pdf).

Ideally the catalog_id should be a composite key derived from the org_id and
perhaps the catalog_name. In this current implementation catalogs/catalog_products
simply maintain references to each other. There is no concept of "list of products"
within a catalog. This design makes catalog actions/operations less expensive to
perform.

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
is the process of continuous harmonization of item information between trading
partners which ensures that the master data is the same in all trading partners
systems.

CIF and GS1's CIS provides a view of current state of product catalogs as well
as the formats and transactions associated with them.

# Unresolved questions

[unresolved]: #unresolved-questions

- Determining the integration with Grid Schema
- Determining schemas for catalog products
- Determining pricing data model and associated function in a future RFC
