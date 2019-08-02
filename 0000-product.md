- Feature Name: Product
- Start Date: 5/13/19
- RFC PR: [#5](https://github.com/hyperledger/grid-rfcs/pull/5)

# Summary
[summary]: #summary


This RFC proposes a generic and extensible framework for a Hyperledger Grid
Product entity as well as a more specific - and fully encapsulated - GS1
compliant Product.

GS1 is a widely used data standard in enterprises and, given that
positioning and familiarity, Grid support feels natural.  Additional
implementations of Product for specialized industries or use cases may derive
from or extend this implementation.

_For the base implementation of a product
on grid, there will be 4 fields. A product_id, a product_namespace, an owner, and properties (a repeated key-value field)._


# Motivation
[motivation]: #motivation

The Grid Product implementation is designed for sharing product master data
between participants. Product is a near universal concept within supply chains
solutions and would naturally be one of the highest areas of re-use across Grid
applications.

The design will address use cases including:

- Sharing of Product master data across a network
- Including Product in other business transactions (track and trace events,
purchases, sales, etc)
- Enriching UX experiences by including additional attribution of a Product
such as names or descriptions

It is also useful to use product information as
auxiliary data in other supply chain solutions.  For example, in track and
trace, the location and temperature of an item (an instance of a product) is
stored.  When presenting this information later to a human user, it is useful
to combine it with information about the product, such as the product’s name
and description.  Therefore, Product becomes a reusable and important component
within Grid.

# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation

## Entities

A **__product__** is an archetype of an item that is transacted, traded, or
referenced in a supply chain.  Each product has a **__product_type__**. This
RFC defines a single product_namespace for GS1; a product of the GS1 product_namespace is
called a GS1 product.  Note that this design supports extending additional
product types in the future.

A product is referenced using a **product_id**.  For GS1 products,
the product_id is a Global Trade Item Number (GTIN), which is part of
the GS1 specifications.

Each product has an **__owner__**, which is the identifier of the organization
responsible for maintaining the product.  An **__agent__** acts on behalf of an
organization, and all permissions are defined in terms of the actions agents
can perform.  Organizations and agents are defined and managed by Pike (another
component of Grid).

A product has one or more **properties**.  Properties are described in the Grid
Primitives RFC.  A *property namespace* contains multiple *property
schemas*.  A property schema associates a name (such as “length”) with a data
type
(such as integer).  GS1 products may only include properties defined in the GS1
product property namespace.

## Transactions

Products are managed by submitting transactions to Hyperledger Grid, which will
process them with the Grid Product smart contract. The following transactions
are supported:

* ProductCreate - create a Product and store it in state.
* ProductUpdate - update (replace) the properties of a Product already in
  state.
* ProductDelete - remove a Product from state.

## Permissions

Creation of GS1 products is restricted to agents acting on behalf of the
organization that is associated with the GS1 company prefix encoded in the
GTIN.  An organization must have the GS1 company prefix in the
“gs1_company_prefixes” metadata field for its Pike organization.
(Organization-level metadata will need to be implemented in Pike.) When
a product is created, its owning organization is stored with the product in an
“owner” field.

Updates to products is restricted to agents acting on behalf of the
organization stored in the product’s owner field.  Only property fields can be
updated. Product_id, product_namespace, and owner fields are immutable.

Deletion of products is restricted to agents acting on behalf of the
organization in the product’s owner field.  A setting will turn off deletion
entirely, with the intent that it could be enabled/disabled by a Grid
administrator.  Disabling delete is useful because external systems may have
references to these products and deleting them could leave dangling references.

## GTIN Validation

Creation of a resuable gtin validation function to programmatically express the
equation used to validate a GTIN. It validates gtin format to avoid mistype
errors similar to a credit card validation. It's implemented as an extensible
function such that further validation steps can be added, if needed. For details on the equation see: [Check
digit validation](https://www.gs1.org/services/how-calculate-check-digit-manually).


# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## State

### Product Representation

The primary object stored in state is “Product”, which consists of an
product_id (GS1 GTIN), an owner (org_id compatible w/Pike),
and a list of property name/value pairs.  The properties available are defined
by the Grid Property Schema Transaction Family and are restricted to the fields
and rules of the GS1 Product schema.  Transactions which are responsible for
setting product state values must ensure that the properties conform with the
requirements of the GS1 Product Property Schema.

```
message Product {
    enum ProductNamespace {
        UNSET_NAMESPACE = 0;
        GS1 = 1;
    }
    string product_id = 1;
    ProductNamespace product_namespace = 2;
    string owner = 3;
    repeated PropertyValue properties = 4;
}
```

The GS1 GTIN is an identifier (product_id) used to identify trade items.  A
GTIN is the data
transmitted from a barcode scan and is made up of a company prefix (or GS1-8
prefix) and item reference.  We will initially support GTIN-12, GTIN-13, and
GTIN-14. (GTIN-8 may be supported in the future.) The GS1 GTIN specification is
documented in section 3.3.2, Identification of a trade item (GTIN): AI (01), on
page 140 of the **GS1 General Specification**:

https://www.gs1.org/sites/default/files/docs/barcodes/GS1_General_Specifications.pdf

The GS1-8 prefix is a unique string of 3 digits, and the GS1 company prefix
consists of 4-12 digits.  The GS1-8 prefixes is issued by the GS1 Global
Office. The GS1-8 prefix is allocated to a GS1 member organization to issue the
GTIN-8’s to other areas.  The GS1 company prefix can be issued by the GS1
Global Office or a member organization. Usually issued to a company itself.

Also relevant is the GS1-8 Prefix or GS1 Company Prefix which namespaces the
GTIN by issuing organization.  GS1 manages the master data for company
prefixes. Owning companies issue their own GTINs within their prefixed
namespace.  Details about GS1 Company Prefixes can be found in section 1.4.4,
GS1 Company Prefix, on page 20 of the GS1 General Specifications.

### Referencing Products

Products are uniquely referenced by their product_id and product_namespace.  For
GS1, Products are referenced by the GTIN identifier. For example:

```
    get_product(product_id) // GTIN
    set_product(product_id, product) // GTIN, gs1Product
```

### Product Addressing in the Merkle-Radix State System

In order to uniquely locate GS1 products in the Merkle-Radix state system, an
address must be constructed which identifies the storage location of the
Product representation.

All Grid addresses are prefixed by the 6-hex-character namespace prefix
“621dee”,  Products are further prefixed under the Grid namespace with reserved
enumerations of “02” (“00” and “01” being reserved for other purposes)
indicating “Products” and an additional “01” indicating “GS1 Products”.

Therefore, all addresses starting with:

```
    “621dee” + “02” + “01”
```

are Grid GS1 Products identified by a GTIN and are expected to contain
a Product representation which conforms with the GS1 product schema.

GTIN formats consist of 14-digit “numeric strings” which include some amount of
internal “0” padding depending on the specific GTIN format (GTIN-8, GTIN-12,
GTIN-13, or GTIN-14).  After the 10-hex-characters that are consumed by the
grid namespace prefix, the product, and GS1 prefixes, there are 60 hex
characters remaining in the address.  The 14 digits of the GTIN can be left
padded with 44-hex-character zeroes and right padded with 2-hex-character
zeroes to accommodate potential future storage associated with the GS1 Product
representation, for example:

```
    “621dee” + “02” + “01” +“00000000000000000000000000000000000000000000” +
    14-character “numeric string” product_id + “00” // product_id == GTIN
```

A full GS1 Product address using the example GTIN from https://www.gtin.info/
would therefore be:

```
“621dee0201000000000000000000000000000000000000000000000001234560001200”
```

## Transaction Payload and Execution

### ProductPayload Transaction

ProductPayload contains an action enum and the associated action payload.  This
allows for the action payload to be dispatched to the appropriate logic.

Only the defined actions are available and only one action payload should be
defined in the ProductPayload.

```
message ProductPayload {
    enum Actions {
        UNSET_ACTION = 0;
        PRODUCT_CREATE = 1;
        PRODUCT_UPDATE = 2;
        PRODUCT_DELETE = 3;
    }

    Action action = 1;

    ProductCreateAction product_create = 2;
    ProductUpdateAction product_update = 3;
    ProductDeleteAction product_delete = 4;
}
```

### ProductCreateAction

ProductCreateAction adds a new product to state. The transaction should be
submitted by an agent, which is identified by its signing key, acting on behalf
of the organization that corresponds to the owner in the create transaction.
(Organizations and agents are defined by the Pike smart contract.)

```
message ProductCreateAction {
    enum Product_Namespace {
        UNSET_NAMESPACE = 0;
        GS1 = 1;
    }
    // product_namespace and product_id are used in deriving the state address
    Product_Namespace product_namespace = 1;
    string product_id = 2;
    string owner = 3;
    repeated PropertyValues properties = 4;
}
```

Validation requirements:

* If a product with product_id already exists the transaction is invalid.
* The signer of the transaction must be an agent in the Pike state and must
belong to an organization in Pike state, otherwise the transaction is invalid.
The agent must have the permission can_create_product for the organization,
otherwise the transaction is invalid.
* If the product_namespace is GS1, the organization must contain a GS1 Company Prefix in its metadata (gs1_company_prefixes), and the prefix must match the company prefix in the product_id, which is a gtin if GS1, otherwise the transaction is invalid.
* The properties must be valid for the product_namespace. For example, if the productis GS1 product, its properties must only contain properties that are includedin the GS1 Schema. If it includes a property not in the GS1 Schema the
transaction is invalid.  _The base GS1 schema will be defined in a future RFC._

The product will be set in state.

The inputs for ProductCreateAction must include:

* Address of the Agent submitting the transaction
* Address of the Organization the Product is being created for
* Address of the Product Type Schema the  product’s properties must match
* Address of the Product to be created

The outputs for ProductCreateAction must include:

* Address of the Product created

### ProductUpdateAction

ProductUpdateAction updates an existing product in state. The transaction
should be submitted by an agent, identified by its signing key, acting on
behalf of an organization that corresponds to the owner in the product being
updated. (Organizations and agents are defined by the Pike smart contract.)

```
message ProductUpdateAction {
    enum Product_Namespace {
        UNSET_NAMESPACE = 0;
        GS1 = 1;
    }
    // product_namespace and product_id are used in deriving the state address
    Product_Namespace product_namespace = 1;
    string product_id = 2;
    // this will replace all properties currently defined
    repeated PropertyValues properties = 4;
}
```

Validation requirements:

* If a product with product_id does not exist the transaction is invalid.
* The signer of the transaction must be an agent in the Pike state and must
belong to an organization in Pike state, otherwise the transaction is invalid.
* The owner in the product must match the organization that the agent belongs to, otherwise the transaction is invalid.
* The agent must have the permission can_update_prouduct for the organization,
otherwise the transaction is invalid.
* The new properties must be valid for the product_namespace. For example, if theproduct is GS1 product, its properties must only contain properties that are
included in the GS1 Schema. If it includes a property not in the GS1 Scheme the
transaction is invalid.

The properties in the product will be swapped for the new properties and the
updated product will be set in state.

The inputs for ProductUpdateAction must include:

* Address of the Agent submitting the transaction
* Address of the Organization the Product is being updated for
* Address of the Product Type Schema the product’s properties must match
* Address of the Product to be updated

The outputs for ProductUpdateAction must include:

* Address of the Product updated

### ProductDeleteAction

ProductDeleteAction removes an existing product from state. The transaction
should be submitted by an agent, identified by its signing key, acting on
behalf of the organization that corresponds to the org_id in the product being
updated. (Organizations and agents are defined by the Pike smart contract.)

```
message ProductDeleteAction {
    enum Product_Namespace {
        UNSET_NAMESPACE = 0;
        GS1 = 1;
    }
    // product_namespace and product_id are used in deriving the state address
    Product_Namespace product_namespace = 1;
    string product_id = 2;
 }
```

If the grid setting grid.product.allow_delete is set to false, this transaction
is invalid. The default value for grid.product.allow_delete is true. This
setting is stored using the Sawtooth Settings smart contract, more information
can be found [here](https://sawtooth.hyperledger.org/docs/core/releases/latest/transaction_family_specifications/settings_transaction_family.html).

Validation requirements:

* If a product with product_id does not exist the transaction is invalid.
* The signer of the transaction must be an agent in the Pike state and must
belong to an organization in Pike state, otherwise the transaction is invalid.
* The owner in the product must match the organization that the agent belongs to, otherwise the transaction is invalid.
* The agent must have the permission “can_delete_product” for the organization,
otherwise the transaction is invalid.

The inputs for ProductDeleteAction must include:

* Address of the Agent submitting the transaction
* Address of the Organization the Product is being deleted for
* Address of the Product to be deleted

The outputs for ProductUpdateAction must include:

* Address of the Product to be deleted

### Defined GS1 Properties

An initial set of GS1 properties will be predefined within Grid. Each property
will have a property definition of the following format:

```
PropertyDefinition(
    name="<GS1 Property>",
    data_type=PropertyDefinition.DataType.STRING,
    required=False,
    description="A description of the GS1 data."
)
```

# Drawbacks
[drawbacks]: #drawbacks

Each GTIN format, except GTIN-8, contains an GS1 Company Prefix organizational
identifier encoded into it, and this RFC duplicates the organization
identification with a Grid-specific owner field to tie the product to
organizations managed in Pike. In the future, it may be good to use the GTIN’s
organizational identifier directly.

Currently, Pike does not provide a place to store a GS1 Company Prefix, and the
permissions around who is authorized to provide this prefix is complicated and
not a direct extension of how Pike currently works. As such it is out of the
scope of this RFC.

Instead, Pike’s organization definition will be extended to include metadata,
similar to Agent’s metadata. Metadata is a Key,Value store of arbitrary data.
The GS1 Company Prefix will be stored in the metadata under
“gs1_company_prefix”.

Solving how to properly provide GS1 Company Prefixes to a Pike Organization
will be solved in a future RFC.

Trade items can include non-material goods, such as services, which will also
need to be represented within Grid. This is not covered by this RFC.

To expand the product schema to support all GS1 properties as well as keeping
it all organized, there will need to be some refactoring done at the grid
primitive level to support lists in the schema.


# Rationale and alternatives
[alternatives]: #alternatives

Starting with capturing a base product allows us to later extend the definition
with the complexities associated with products in general. We will be able to
iterate and develop extensible product/schema definitions for each product
space i.e.  clothing, furniture, etc.

Some details on products that can be used for a future iteration: [GS1 Apparel
and General
Merchandise](https://www.gs1us.org/DesktopModules/Bring2mind/DMX/Download.aspx?Command=Core_Download&EntryId=1067&language=en-US&PortalId=0&TabId=134)

### Clothing:

Material content:\
This element is used to indicate the material composition.  This element is
used in conjunction with the percentage.

Material Percentage:\
Net weight percentage of a product material of the first main material. The
percentages must add up to 100.

Material Percentage:\
Net weight percentage of a product material of the first main material. The
percentages must add up to 100.

### Furniture:

Number of pieces:\
The total number of separately packaged components comprising a single trade
item.

Look at section 3.5.7 Packaging component number AI (243)

### Possible payload modifications:

The owner field is not a _HARD_ requirement in in the product_state.proto. The
owner (organization) could later be derived from the gtin, as we learn more
about gtins and integrate some kind of lookup functionality. That functionality
is not included in this RFC and could be implemented at a later time.

# Prior Art
[prior-art]: #prior-art

[Pike Transaction Family
Specification](https://github.com/hyperledger/sawtooth-sabre/blob/master/contracts/sawtooth-pike/docs/source/pike_transaction_family.rst)

The Sawtooth Supply Chain mechanism to define field types.  [Sawtooth Supply
Chain Expanded Data
Types](https://github.com/hyperledger/sawtooth-rfcs/blob/master/text/0013-supply-chain-expand-data-types.md)

# Unresolved questions
[unresolved]: #unresolved-questions