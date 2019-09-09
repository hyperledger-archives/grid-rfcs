- Feature Name: Primitives
- Start Date: TBD
- RFC PR: TBD

# Summary
[summary]: #summary

Grid Primitives for Hyperledger Grid provides a reusable, standard approach to
defining, storing, and consuming properties within smart contracts, software
libraries, and network-based APIs.

# Motivation
[motivation]: #motivation

Several components within Grid will store and retrieve properties which are
defined at runtime. To properly store and validate these properties, we need
property definitions which minimally include the property’s type (integer,
string, enum, etc.). In addition, the properties (for example, product
description, GPS lat/long, or product dimensions) should always be stored and
exchanged using the same format within Grid components.

# Guide-Level Explanation
[guide-level-explanation]: #guide-level-explanation

## PropertyDefinition

A property is defined using a `PropertyDefinition` which includes the following:

- Data type (one of: BYTES, BOOLEAN, NUMBER, STRING, ENUM, STRUCT, LAT_LONG,
  DATETIME)
- Name
- Type description
- Optionality (whether or not the field is required)

## Data types

### Bytes

A Bytes data type is an array of raw bytes.  This can be used to store
arbitrary, opaque data. For example, a property with the Bytes data type could
be used to store serialized JSON objects containing application metadata for a
field, such as an image URL or style name.

### Booleans

A boolean data type restricts a value to True and False. Though boolean types
could be stored in other integer (or byte) types using 0 or 1, an explicit
boolean type assists in capturing intent and restricting the value.

### Strings

A string data type contains a standard UTF-8 encoded string value.

### Numbers

Numbers are represented as an integer with a given precision.  This can be
thought of as akin to scientific notation. An instance of a number with this
property definition is represented as a value (the significand) with the
exponent (the order of magnitude) defined in the schema itself. So for example:

```
(value: 24, exponent: 3)  -> 24 * 10^3  -> 24000
(value: 24, exponent: -3) -> 24 * 10^-3 -> 0.024
(value: 24, exponent: 0)  -> 24 * 10^0  -> 24
```

Importantly, this exponent will be set on a Property's schema, not when the
value is actually input. It will affect the semantic meaning of integers stored
under a Property, not any of the actual operations done with them. Properties
with an exponent of 3 or -3 will always be expressed as a whole integer of
thousands or thousandths. For this reason, the exponent should be thought of
more as a unit of measure than as true scientific notation. 

Standard integers are represented with the exponent set to zero. 

### Enums

An enum data type restricts values to a limited set of possible values. The
definition for this data type includes a list of string names describing all the
possible variants of the enum.

### Structs

A struct is a recursively defined collection of other named properties that
represents two or more intrinsically linked values, like X/Y coordinates or RGB
colors. These values can be of any Grid primitive data type, including STRUCT,
allowing nesting to an arbitrary depth. Although versatile and powerful, structs
are heavyweight and should be used conservatively; restrict struct use to
linking values that must always be updated together. The smart contract will
enforce this usage, rejecting any transactions that do not have a value for
every property in a struct.

Note that although structs are built using a list of PropertyDefinitions, any
nested use of the required property is meaningless and will be rejected by the
smart contract. As Properties are set in their entirety, either all of the
struct is required or none of it is. In other words, partial structs are not
allowed.

### Latitude/Longitude

Latitude/Longitude (Lat/Long) values are represented as a pre-defined struct
made up of a latitude, longitude pair.  Both latitude and longitude are
represented as signed integers indicating millionths of degrees.

### Datetime

Datetime values are represented using the
[ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) combined date-time formats.
For example, "2007-04-05T14:30Z" or "2007-04-05T12:30-02:00".

## Schema

Property definitions are collected into a Schema data type, which defines all
the possible properties for an item that belongs to a given Schema. A Schema
includes the following:

- *name* - the name and identifier for the Schema
- *description* - description of the Schema
- *owner* - the Pike compatible organization identifier for the owning
  organization of the Schema
- *properties* - a list of `PropertyDefinitions` that define the attributes of
  the entity being defined by the Schema

Schemas are used to validate a collection of property values defined in a data
structure.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

A schema and its property definitions are stored in state in the protobuf
message format.  These messages are defined as follows:

```
message PropertyDefinition {
    enum DataType {
        UNSET_DATA_TYPE = 0;
        BYTES = 1;
        BOOLEAN = 2;
        NUMBER = 3;
        STRING = 4;
        ENUM = 5;
        STRUCT = 6;
        LAT_LONG = 7;
        DATETIME = 8;
    }

    // The name of the property
    string name = 1;
    // The data type of the value; must not be set to UNSET_DATA_TYPE.
    DataType data_type = 2;
    // Indicates that this is a required property in the Schema
    bool required = 3;
    // An optional description of the field.
    string description = 4;

    // The exponent for a NUMBER property
    sint32 number_exponent = 10;
    // The list of values for an ENUM property; must not be empty for
    // properties of that type.
    repeated string enum_options = 11;
    // The list of property definitions for a STRUCT property; must not be
    // empty for properties of that type.
    repeated PropertyDefinition struct_properties = 12;
}

message Schema {
    // The name of the Schema.  This is also the unique identifier for the
    // Schema.
    string name = 1;
    // An optional description of the schema.
    string description = 2;
    // The Pike organization identifier that has rights to modify the schema.
    string owner = 3;

    // The property definitions that make up the Schema; must not be empty.
    repeated PropertyDefinition properties = 10;
}

// A SchemaList is used to mitigate hash collisions.
message SchemaList {
    repeated Schema schemas = 1;
}

message LatLong {
      // Coordinates are expected to be in millionths of a degree
      sint64 latitude = 1;
      sint64 longitude = 2;
}

message PropertyValue {
    // The name of the property value.  Used to validate the property against a
    // Schema.
    string name = 1;
    // The data type of the property.  Indicates in which value field the actual
    // value may be found.  Must not be set to `UNSET_DATA_TYPE`.
    PropertyDefinition.DataType data_type = 2;

    // The value fields for the possible data types.  Only one of these will
    // contain a value, determined by the value of `data_type`
    bytes bytes_value = 10;
    bool boolean_value = 11;
    sint64 number_value = 12;
    string string_value = 13;
    uint32 enum_value = 14;
    repeated PropertyValue struct_values = 15;
    LatLong lat_long_value = 16;
    string datetime_value = 17;
}
```

## Property Examples

The following examples show how these property definitions and instance values
would be constructed using the above protobuf messages.  The examples use a
Python-like syntax.

### Bytes

A bytes value would be represented as follows:

```
PropertyDefinition(
    name="user_data",
    data_type=PropertyDefinition.DataType.BYTES,
    description="Arbitrary serialized user data."
)
```

Because this is a protobuf message, the default value for this field is an empty
byte array.

### Boolean

A boolean value would be represented as follows:

```
PropertyDefinition(
    name="is_enabled",
    data_type=PropertyDefinition.DataType.BOOLEAN,
    required=True,
    description="Indicates that the containing struct is enabled."
)
```

The value would be represented as:

```
PropertyValue(
    name="is_enabled",
    data_type=PropertyDefinition.DataType.BOOLEAN,
    boolean_value=True
)
```

Because this is a protobuf message, the default value for this field is `False`.

### String

A UTF-8 string value would be represented as follows:

```
PropertyDefinition(
    name="title",
    data_type=PropertyDefinition.DataType.STRING,
    required=True,
    description="A blog post title."
)
```

The value would be represented as:

```
PropertyValue(
    name="title"
    data_type=PropertyDefinition.DataType.STRING,
    string_value="My Very Nice Blog Example"
)
```

Because this is a protobuf message, the default value for this field is the
empty string.

### Number

An integer value would be represented as the following type:

```
PropertyDefinition(
    name="quantity",
    data_type=PropertyDefinition.DataType.NUMBER,
    number_exponent=0,
    required=True,
    description="The count of values in this container"
)
```

This example shows an instance of a quantity of 23:

```
PropertyValue(
    name="quantity",
    data_type=PropertyDefinition.DataType.NUMBER,
    number_value=23,
)
```

A fractional value would be represented as the following type:

```
PropertyDefinition(
    name="price",
    data_type=PropertyDefinition.DataType.NUMBER,
    number_exponent=-2,
    required=True,
    description="The price of this object"
)

```

This example shows an instance of a price with the value $0.23:

```
PropertyValue(
    name="price",
    data_type=PropertyDefinition.DataType.NUMBER,
    number_value=23,
)
```

Because this is a protobuf message, the default exponent is `0` when the schema
is created. Likewise, the default value for this property instance is `0`.

### Enum

An enum value would be represented as:

```
PropertyDefinition(
    name='color',
    data_type=PropertyDefinition.DataType.ENUM,
    enum_options=['white', 'red', 'green', 'blue', 'blacklight'],
    required=True
)
``` 

An instance of this enum, indicating "red", would be as follows:

```
PropertyValue(
    name='color',
    data_type=PropertyDefinition.DataType.ENUM,
    enum_value=1
)
```

Due to the use of protobuf, the default value for this property instance is `0`.
It is left to the smart-contract implementer to determine if this would result
in an error condition.

### Struct

A struct value would be represented as follows:

```
PropertyDefinition(
    name='shock',
    data_type=PropertyDefinition.DataType.STRUCT,
    struct_properties=[
        PropertyDefinition(
            name='speed',
            data_type=PropertyDefinition.DataType.NUMBER,
            number_exponent=-6),
        PropertyDefinition(
            name='duration',
            data_type=PropertyDefinition.DataType.NUMBER,
            number_exponent=-6),
    ],
    required=True
)
```

An instance of the “shock" struct would be as follows:

```
PropertyValue(
    name='shock',
    data_type=PropertyDefinition.DataType.STRUCT,
    struct_values=[
        PropertyValue(
            name='speed',
            data_type=PropertySchema.DataType.NUMBER,
            number_value=500000),
        PropertyValue(
            name='duration',
            data_type=PropertySchema.DataType.NUMBER,
            number_value=10000)
        ])
```

The property value for a struct must contain all the struct values from the
property definition, or it is invalid.  The defaults for the struct values
themselves depend on their data types and/or the smart-contract implementer
validation rules.

### Latitude/Longitude

A latitude/longitude (lat/long) value would be represented as follows:

```
PropertyDefintion(
    name='origin',
    data_type=PropertyDefinition.DataType.LAT_LONG,
    required=True
)
```

A lat/long instance would be as follows:

```
PropertyValue(
    name='origin',
    data_type=PropertyDefinition.DataType.LAT_LONG,
    lat_long_value=LatLong(
        latitude=44977753,
        longitude=-93265015)
)
```

Due to the use of protobuf, the default values for `LatLong` would be `(0, 0)`.
While this is a valid lat/long, it could be used to indicate an error, depending
on the choice of the smart-contract implementer.

### Datetime

A datetime value would be represented as follows:
```
PropertyDefinition(
    name='created_at',
    data_type=PropertyDefinition.DataType.DATETIME,
    required=True
)
```

A datetime instance for this definition would be as follows:
```
PropertyValue(
    name='created_at',
    data_type=PropertyDefinition.DataType.DATETIME,
    datetime_value='2019-05-31T14:53:18+0000'
)
```

Due to the use of protobuf, the default value for `datetime_value` is an empty
string, and therefore an invalid value.  This field must be set for the
`DATETIME` data type.

## Schema Example

A complete object representation can be built from the property definition
messages, and instances can be represented by constructing items with the
property value messages.  

Suppose there is a requirement to store different types of lightbulbs as part
of an application. A lightbulb may consist of the following properties: size,
bulb type, energy rating, and color.

We could define a `Lightbulb` schema as follows:

```
Schema(
    name="Lightbulb",
    description="Example Lightbulb schema",
    owner = "philips001"
    properties=[
        PropertyDefinition(
            name="size",
            data_type=PropertyDefinition.DataType.NUMBER,
            description="Lightbulb radius, in millimeters",
            number_exponent=0,
            required=True
        ),
        PropertyDefinition(
            name="bulb_type",
            data_type=PropertyDefinition.DataType.ENUM,
            enum_options=["filament", "CF", "LED"],
            required=True
        ),
        PropertyDefinition(
            name="energy_rating",
            data_type=PropertyDefinition.DataType.NUMBER,
            number_exponent=-2,
            description="EnergyStar energy rating (percent)",
        )
        PropertyDefinition(
            name="color",
            data_type=PropertyDefinition.DataType.STRUCT,
            description="A named RGB Color value",
            struct_properties=[
                PropertyDefinition(
                    name='name',
                    data_type=PropertyDefinition.DataType.STRING,
                ),
                PropertyDefinition(
                    name='rgb_hex',
                    data_type=PropertyDefinition.DataType.STRING,
                )])])
```

Note: This example looks very similar to defining a struct property, but the
fields in a schema may be optional.

In order to store a lightbulb in global state, we can define a data structure
with several fixed fields, namely identifiers, as well as the set of dynamic
property values that will conform to the schema:

```
message Lightbulb {
    // The lightbulb identifier
    string id = 1;

    // the production facility organization id
    string production_org = 2;

    // the properties that conform to the Lightbulb schema
    repeated PropertyValue properties = 3;
} 
```

A `Lightbulb` smart contract would then be responsible for validating the
`properties` field against the Lightbulb schema at run-time.

An `Lightbulb` entry, with all optional properties, would then look like the
following:

```
Lightbulb(
    id="12344",
    production_org='philipsbulbfactory',
    properties=[
        PropertyValue(
            name="size",
            data_type=PropertyDefinition.DataType.NUMBER,
            number_value=10,
        ),
        PropertyValue(
            name="bulb_type"
            data_type=PropertyDefinition.DataType.ENUM,
            enum_value=2,  # LED
        ),
        PropertyValue(
            name="energy_rating",
            data_type=PropertyDefinition.DataType.NUMBER,
            number_value=89,
        ),
        PropertyValue(
            name="color",
            data_type=PropertyDefinition.DataType.STRUCT,
            struct_values=[
                PropertyValue(
                    name="name",
                    data_type=PropertyDefinition.DataType.STRING,
                    string_value="White",
                ),
                PropertyValue(
                    name="rgb_hex",
                    data_type=PropertyDefinition.DataType.STRING,
                    string_value="000000",
                ),
           ]),
    ])
```

## Addressing

### Referencing Schemas

Schemas are uniquely referenced by their unique schema name. For example:

```
    get_schema(schema_name)
    set_schema(schema_name, schema)
```

### Schema Addressing in the Merkle-Radix State System

Grid Schemas are stored under the Grid namespace 621dee. For each schema, the
address is formed by concatenating the namespace, the special policy namespace
of 01, and the first 62 characters of the SHA-256 hash of the schema name.

For example, the address of the "Lightbulb" schema defined in the example above
is (in Python): 

```
    "621dee" + "01" + hashlib.sha512("Lightbulb").encode("utf-8")).hexdigest()[:62]
```

To avoid hash collisions, schemas must be stored in a `SchemaList`.

## Transactions

In order to add Schemas to state, a transaction must be used.  The following
transactions and their execution rules are designed for the Hyperledger Sawtooth
platform and may differ for other transaction execution platforms.

### Transaction Header

The header for the transactions will include the following:

- `family_name`: `"grid_schema"`
- `family_version`: `"1.0"`
- `namespaces`: `[ "621dee" ]`

### Payloads and Execution Rules

#### SchemaPayload

SchemaPayload contains an action enum and the associated action payload.  This
allows for the action payload to be dispatched to the appropriate logic.

Only the defined actions are available and only one action payload should be
defined in the SchemaPayload.

```
message SchemaPayload {
    enum Actions {
        UNSET_ACTION = 0;
        SCHEMA_CREATE = 1;
        SCHEMA_UPDATE = 2;
    }

    Action action = 1;

    SchemaCreateAction schema_create = 2;
    SchemaUpdateAction schema_update = 3
}
```

#### SchemaCreateAction

SchemaCreateAction adds a new Schema to state. 

```
message SchemaCreateAction {
    string schema_name = 1;
    string description = 2;
    repeated PropertyDefinition properties = 10;
}
```

The action is validated according to the following rules:

- If a Schema already exists with this name or the name is an empty string, the
  transaction is invalid.
- If the property list is empty, the transaction is invalid.
- The signer of the transaction must be an agent in Pike state and must belong
  to an organization in Pike state, otherwise the transaction is invalid.
- The agent must have the permission `can_create_schema` for the organization,
  otherwise the transaction is invalid.  

The schema is created with the provided fields, in addition to the Pike
organization ID as the `owner_id`. The schema is then stored in state.

The inputs for SchemaCreateAction must include:

- Address of the Agent submitting the transaction
- Address of the Schema
  
The outputs for SchemaCreateAction must include:

- Address of the Schema  

#### SchemaUpdateAction

SchemaUpdateAction updates a Schema to state. This update only adds new
Properties to the Schema.

```
message SchemaUpdateAction {
    string schema_name = 1;
    repeated PropertyDefinition properties = 2;
}
```

The action is validated according to the following rules:

- If a Schema does not exist, the transaction is invalid.
- If the property list is empty, the transaction is invalid.
- If one of the new properties has the same name as a property already defined
  in the schema, the  transaction is invalid.
- The signer of the transaction must be an agent in the Pike state and must
  belong to an organization in Pike state, otherwise the transaction is invalid.
- The signer of the transaction must belong to the same organization matching
  the `owner` of the schema, otherwise the transaction is invalid.
- The agent must have the permission `can_update_schema` for the organization,
  otherwise the transaction is invalid.

The inputs for SchemaUpdateAction must include:

- Address of the Agent submitting the transaction
- Address of the Schema  

The outputs for SchemaCreateAction must include:

- Address of the Schema  

# Drawbacks
[drawbacks]: #drawbacks

In this RFC, validating a collection of property values in a data structure
against a Schema is left to the smart contract implementer. This may result in
inconsistent validation rules, particularly in the cases where the natural
default value provided by protobuf deserialization may be ambiguous (for
example, the enum case).

The dynamic nature of the property values also require a query for the schema in
order to validate the state.  This incurs an additional cost when creating and
updating a collection of property values. 

# Rationale and alternatives
[alternatives]: #alternatives

This design provides a reusable pattern for arbitrary properties in data
structures defined in Grid smart contracts. Providing a common language across
all data structures built with the schemas will allow for the creation of a
common set of tools, such as REST APIs and Sawtooth state delta export.

Alternatively, the reusable components could be implemented at the message
level, but this provides little in the way of pre-defined primitives.  They
would be too broadly defined as protobuf primitive values.  It also moves the
schema out as an external definition, which would make dynamic applications (for
example, common UI components) more difficult.  The applications would need to
inspect the message format itself, in order to provide details about the data
structures.

# Prior Art
[prior-art]: #prior-art

These types are adapted from the original [Sawtooth Supply Chain Transaction
Family
Specification](https://github.com/hyperledger/sawtooth-supply-chain/blob/master/docs/source/family_specification.rst)
and the expanded data types RFC, [Sawtooth RFC
0013](https://github.com/hyperledger/sawtooth-rfcs/blob/master/text/0013-supply-chain-expand-data-types.md).

# Unresolved questions
[unresolved]: #unresolved-questions

It would be nice to provide validation functions to the Grid SDK that would
validate that the data structure matches a schema. This would make the use of
schemas easier on a smart contract developer. The implementation for such
functions are outside the scope of this RFC.

As described in this RFC, complex data types, like Lat/Long, could be
represented as structs.  In the case of Lat/Long, it is considered common enough
to be promoted to a first-class data type.  How and when other structs are
promoted to a first-class data type remains to be determined.

Schemas currently have no version stored in state.  Given that the schemas are
only created and updated in an additive fashion, a new version of a schema is
equivalent to creating a new schema with a new name.  For example,
`my_schema-1.0` could be replaced by `my_schema-1.1`.  This does not answer the
question of how structs verified by different schema versions are migrated, nor
whether or not this is a strong enough pattern for versioning.
