- Feature Name: Location
- Start Date: 2020-07-29
- RFC PR: 
- Hyperledger Grid Issue: 

# Summary
[summary]: #summary

This RFC proposes a generic and extensible framework for a Hyperledger Grid 
Location entity as well as a more specific - and fully encapsulated - GS1 
compliant Location. GS1 is a widely used data standard in enterprises and, given 
that positioning and familiarity, Grid support feels natural. Additional 
implementations of Location for specialized industries or use cases may derive 
from or extend this implementation.

This Grid Location proposal is specific to physical locations within a supply 
chain. The GS1 General Specification defines a physical location as a site (an area,
a structure or group of structures) or an area within the site where something 
was, is, or will be located. For example: A label attached to a loading dock or 
to a shelf location in a warehouse. Legal entities, functions and digital 
locations are out of scope for our initial design and should be addressed in 
supplemental RFC(s).

# Motivation
[motivation]: #motivation

The Grid Location implementation is designed for sharing location master data 
between participants. Location is a near universal concept within supply chain 
solutions and would naturally be one of the highest areas of re-use across Grid 
applications. The identification of where a physical location exists (and thus 
where a business transaction occurs) is a foundational element that will enable 
a wide host of distributed supply chain and commerce solutions. The design will 
address use cases such as:

- the inclusion of Location within business transactions (e.g. track and trace 
events, inventory and warehouse management, point of sale, network optimization).
- the enrichment of UX experiences by the inclusion of additional Location 
attribution (e.g. name, address, location type).


# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

## Entities
[entities]: #entities

A location has an identifier, a namespace, an owner, and one or more properties. 
Properties are described in the “_Defined GS1 Properties_” section of the RFC.

|Master Data Common Name|Attribute Name|Description|Example|
|-----------------------|--------------|-----------|-------|
|Location Identifier|location_id|A location is referenced using a location_id (key attribute). For GS1 locations, the location_id is a Global Location Number which is part of GS1 specifications.|"0099474000005"|
|Location Namespace|location_namespace|This RFC defines a single location_namespace for GS1; a location in the GS1 location_namespace is called a GS1 location. Note that this design supports extending additional location namespaces in the future.|"01"|
|Owner|owner/org_id|The identifier of the organization responsible for maintaining the location. For GS1 locations, org_id is the Pike organization which claims ownership of the GS1 company prefix (via the "gs1_company_prefixes" field in the Pike records). The GLN will always start with the company prefix. The smart contract will check that the org claiming ownership lists the matching prefix in their Pike org.||


## Transactions
[transactions]: #transactions

In order to add Locations to state, a transaction must be used. The following 
transactions and their execution rules are designed for use with Hyperledger
Sawtooth's Sabre smart contract engine running via Hyperledger Transact, which
covers all currently implemented DLT backends in Grid.

Locations are managed by submitting transactions to Hyperledger Grid, which will
process them with the Grid Location smart contract. The following transactions 
are supported:
- LocationCreate - create a Location and store it in state.
- LocationUpdate - update (replace) the properties of a Location already in state.
- LocationDelete - remove a Location from state.
 
_*Disclaimer: LocationDelete is a potentially hazardous operation and needs to 
be done with care._


## Permissions
[permissions]: #permissions

Creation of GS1 locations is restricted to agents acting on behalf of the 
organization that is associated with the GS1 company prefix encoded in the GLN. 
Like Grid Product, an organization must have the GS1 company prefix in the 
“gs1_company_prefixes” metadata field for its Pike organization. When a 
location is created, its owning organization is stored with the location in an 
“owner” field.

Updates to locations are restricted to agents acting on behalf of the 
organization stored in the location’s owner field. Only property fields can be 
updated. They are updated by providing a complete and updated list of the 
properties, and overwrites the entire list of property values. Location_id, 
location_namespace, and owner fields are immutable.

Deletion of locations is restricted to agents acting on behalf of the 
organization in the location’s owner field. A setting will turn off deletion 
entirely, with the intent that it could be enabled/disabled by a Grid 
administrator. Disabling delete is useful because external systems may have 
references to these locations and deleting them could leave dangling references.


## GLN Validation
[gln validation]: #glnvalidation

GLN validation can reduce the opportunity for invalid data to enter the system.
At a minimum, when GLNs are entered in clients and later processed in smart
contracts, they will need to conform to the GLN format and the check digit
should be correct. See [check digit calculator](https://www.gs1us.org/tools/check-digit-calculator "check digit calculator").



# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## State
[state]: #state

### Location Representation

The primary object stored in state is “Location”, which consists of a 
location_id (GS1 GLN), an owner (org_id compatible with Pike), and a list of 
property name/value pairs. The properties available are defined by the Grid 
Location Schema Transaction Family and are restricted to the fields and rules of
the GS1 Location Property Schema. Transactions which are responsible for setting
location state values must ensure that the properties conform with the 
requirements of the GS1 Location Property Schema.

    message Location { 
        enum LocationNamespace { 
            UNSET_NAMESPACE = 0; 
            GS1 = 1; 
        }

        // What type of location is this (GS1)
        LocationNamespace location_namespace = 1; 

        // location_id is the GLN for GS1-namespace locations
        string location_id = 2;

        // Who owns this location (pike organization id)
        string owner = 3;

        // Properties about this location
        repeated PropertyValue properties = 4; 
    }

The GS1 GLN is an identifier (location_id) used to identify entities or 
locations. A GLN is the data transmitted from a data carrier (barcode, rfid, 
etc.) scan and is made up of a company prefix, location reference, and check 
digit.

### Referencing Locations

Locations are uniquely referenced by their location_id and location_namespace.
For GS1, locations are referenced by the GLN identifier.

**Note: For the remainder of this document, “address” will refer to a 
Grid/Transact address as opposed to a Street Address.**

### Location Addressing in the Merkle-Radix State System

In order to uniquely locate GS1 locations in the Merkle-Radix state system, an 
address must be constructed which identifies the storage location of the 
Location representation.

All Grid addresses are prefixed by the 6-hex-character namespace prefix “621dee”.
Locations are further prefixed under the Grid namespace with reserved 
enumerations of “04” (“00”, “01”, “02” and “03” being reserved for other 
purposes) indicating “Locations” and an additional “01” indicating “GS1 
Locations”.

Therefore, all addresses starting with “621dee” + “04” are Grid locations, and 
more specifically, all addresses starting with “621dee” + “04” + “01” are Grid 
GS1 Locations identified by a GLN.

The GLN format consists of a 13-digit “numeric string” that follows a similar 
structure to GTIN-13. The GLN structure first includes a GS1 Company Prefix 
which is 7-10 digits in length and is assigned by a GS1 Member Organization. A 
Location Reference number follows which is 2-5 digits in length and is 
allocated by the company to a location or party. Lastly, a single digit Check 
Digit is calculated and applied according to a GS1 algorithm. See
[GLN Data Format](https://encrypted-tbn0.gstatic.com/images?q=tbn%3AANd9GcSFo3J5QULgkCWyIH3yeddRPx1H5aQusLRRq47y1n2FqTshGxs%3Ax-raw-image%3A%2F%2F%2Ff4fc819afc288cee1948ca0d40d76c3bcd56a6d270fcfeeac80fd25784eb6071&usqp=CAU "GLN Data Format").

After the 10-hex-characters that are consumed by the grid namespace prefix, the 
location, and GS1 prefixes, there are 60 hex characters remaining in the address.
The 13 digits of the GLN can be left padded with 45-hex-character zeroes and 
right padded with 2-hex-character zeroes to accommodate potential future storage
associated with the GS1 Location representation, for example:

    “621dee” + “04” + “01” + “000000000000000000000000000000000000000000000” +
    13-character “numeric string” location_id + “00” // location_id == GLN

A full GS1 Location address (for example purposes) would therefore be:

    “621dee0401000000000000000000000000000000000000000000000123456789012800”


## Transaction Payload and Execution
[transaction payload and execution]: "transactionpayloadandexecution"

### Location PayLoad Transaction

LocationPayload contains an action enum and the associated action payload. This 
allows for the action payload to be dispatched to the appropriate logic.

Only the defined actions are available and only one action payload should be 
defined in the LocationPayload.
    
        message LocationPayload {        
        enum Actions {
            UNSET_ACTION = 0;
            LOCATION_CREATE = 1;
            LOCATION_UPDATE = 2;
            LOCATION_DELETE = 3;
        }

        Action action = 1;

        // Approximately when transaction was submitted, as a Unix UTC timestamp
        uint64 timestamp = 2;

        LocationCreateAction location_create = 3;
        LocationUpdateAction location_update = 4;
        LocationDeleteAction location_delete = 5;
    }

### LocationCreateAction

LocationCreateAction adds a new location to state. The transaction should be 
submitted by an agent, which is identified by its signing key, acting on behalf 
of the organization that corresponds to the owner in the create transaction. 
(Organizations and agents are defined by the Pike smart contract.)

Validation requirements:
- If a location with location_id already exists the transaction is invalid.
- The signer of the transaction must be an agent in the Pike state and must 
belong to an organization in Pike state, otherwise the transaction is invalid.
- The agent must have the permission can_create_location for the organization, 
otherwise the transaction is invalid.
- If the location_namespace is GS1, the organization must contain a GS1 Company 
Prefix in its metadata (gs1_company_prefixes), and the prefix must match the 
company prefix in the location_id, which is a GLN if GS1, otherwise the 
transaction is invalid.
- The properties must be valid for the location_namespace. For example, if the
location is GS1 location, its properties must only contain properties that are
included in the GS1 Schema. If it includes a property not in the GS1 Schema the
transaction is invalid.

If all requirements are met, the transaction will be accepted and the location
will be created in state.

The inputs for LocationCreateAction must include:
- Grid address of the Agent submitting the transaction
- Grid address of the Organization the Location is being created for
- Grid address of the Location Namespace Schema the location’s properties must 
match
- Grid address of the Location to be created

The outputs for LocationCreateAction must include:
- Grid address of the Location created

### LocationUpdateAction

LocationUpdateAction updates an existing location in state. The transaction 
should be submitted by an agent, identified by its signing key, acting on behalf
of an organization that corresponds to the owner in the location being updated.
(Organizations and agents are defined by the Pike smart contract.)

Validation requirements:
- If a location with location_id does not exist the transaction is invalid.
- The signer of the transaction must be an agent in the Pike state and must 
belong to an organization in Pike state, otherwise the transaction is invalid.
- The owner in the location must match the organization that the agent belongs 
to, otherwise the transaction is invalid.
- The agent must have the permission can_update_location for the organization, 
otherwise the transaction is invalid.
- The new properties must be valid for the location_namespace. For example, if 
the location is a GS1 location, its properties must only contain properties that
are included in the GS1 Schema. If it includes a property not in the GS1 Schema 
the transaction is invalid.

The properties in the location will be swapped for the new properties and the 
updated location will be set in state.

The inputs for LocationUpdateAction must include:
- Grid address of the Agent submitting the transaction
- Grid address of the Organization the Location is being updated for
- Grid address of the Location Namespace Schema the location’s properties must
match
- Grid address of the Location to be updated

The outputs for LocationUpdateAction must include:
- Grid address of the Location updated

Note: This RFC handles updates to Location master data (e.g. a Location without 
a defined inactivationDate is updated to have a defined inactivationDate). The 
logic that consumes Location master data will decide what records are valid for 
its use case. For example: A Location with a defined inactivationDate cannot be 
used in new transactions but can still be used when referencing earlier 
transactions. In this example, consuming logic will decide which Locations are 
included in a transaction.

### LocationDeleteAction

LocationDeleteAction removes an existing location from state. The transaction 
should be submitted by an agent, identified by its signing key, acting on behalf
of the organization that corresponds to the owner in the location being updated.
(Organizations and agents are defined by the Pike smart contract.)

If the Grid setting grid.location.allow_delete is set to false, this transaction
is invalid. The default value for grid.location.allow_delete is true. This 
setting is stored using the Sawtooth Settings smart contract, more information
can be found here.

Validation requirements:
- If a location with location_id does not exist the transaction is invalid.
- The signer of the transaction must be an agent in the Pike state and must 
belong to an organization in Pike state, otherwise the transaction is invalid.
- The owner in the location must match the organization that the agent belongs 
to, otherwise the transaction is invalid.
- The agent must have the permission “can_delete_location” for the organization
otherwise the transaction is invalid.

The inputs for LocationDeleteAction must include:
- Grid address of the Agent submitting the transaction
- Grid address of the Organization the Location is being deleted for Grid 
address of the Location to be deleted

The outputs for LocationDeleteAction must include:
- Grid address of the Location to be deleted

### Defined GS1 Properties

This Location RFC defines required and optional attributes for GS1 Locations 
(see table below). This implementation does not include attribution for specific
industries, financial/tax account information, a physical location extension, or
digital locations and should be implemented with supplemental RFC(s).

**_REQUIRED FIELDS_**
|GS1 Common Name|GS1 Attribute Name|Description|Example|Data Type|Min|Max|
|---------------|------------------|-----------|-------|----|---|---|
|Location Name 1|locationName|The name of the facility being described.|"Sunny Fresh Foods"|STRING|1|80|
|Description|locationDescription|Free text, 178 characters.|"A Cargill production facility dedicated to serving high-quality egg products across various markets."|STRING| | |
|Location Type|locationType|Multiple types allowed. All Suppliers: Org Entity, Order From, Remit To, Ship To. Healthcare Providers: Bill To, Deliver To, Order By, Order From, Org Entity, Paid By, Recall, Remit To, Ship From, Ship To.|"Ship From"|ENUM|7|48|
|Address Line 1|addressLine1|The primary street address for your location. The USPS address is validated if Country = United States.| "206 W 4th Street"|STRING|1|80|
|City|city|Name of the city of your location. The USPS address is validated if Country = United States.|"Monticello"|STRING|1|35|
|State or Region|stateOrRegion|The state, province, or region using the standard two-letter abbreviation specified in ISO 3166-2:1998 country subdivision code [16].|"MN"|STRING|1|3|
|Postal Code|postalCode|The ZIP or other postal code.|"55362-8524"|STRING|1|10|
|Country|country|Country of your location. Spell out, do not use abbreviations.|"United States"|STRING|2|80|
|Latitude, Longitude|latitude, longitude| |"44.986656, -93.258133"|LAT_LONG| | |
|Contact Name|contactName| |"Jane Doe"|STRING| | |
|Contact Email|contactEmail| |"jane_doe@example.com"|STRING| | |
|Contact Phone|contactPhone|The location's primary phone number. Best practice is individual with assignment duty.|"937-435-3870"|STRING|1|30|
|Create Date|createDate|Date this location becomes active.|"06/01/2015"|DATETIME| | |

**_OPTIONAL FIELDS_**
|GS1 Common Name|GS1 Attribute Name|Description|Example|Data Type|Min|Max|
|---------------|------------------|-----------|-------|----|---|---|
|Location Name 2|locationName2|A secondary facility name.|"Cargill Incorporated"|STRING|0|80|
|Address Line 2|addressLine2|Any secondary information such as Suite, Floor, etc. The USPS address is validated if Country = United States.|"Suite 204"|STRING|0|80|
|Address Line 3|addressLine3|Additional descriptive information that is not verified through the USPS data base. Best practice is to use AddressLine3 when there are multiple locations using the same USPS address. Examples: Billing office, cardiology lab, backroom, etc.| |STRING| | |
|Inactivation Date|inactivationDate|Date this location is no longer used by the information provider.|"01/15/2020"|DATETIME| | |
|Parent Location GLN|parentLocation|Used to describe a location hierarchy. Needed for every GLN except the top-level location, which does not have a parent location.|"0653114000000"|NUMBER|13|13|
|Industry Sector|industrySector|Select one option: General, CPG, Healthcare, Foodservice.|"Foodservice"|ENUM| | |
|Supply Chain Role|role|Available options are based on the selected Industry Sector for this GLN. General: Manufacturer, Solutions Provider, Undefined. CPG: Manufacturer, Solutions Provider, Undefined. Healthcare: Distributor, Provider, Supplier, Undefined. Foodservice: 3rd Party, Warehouse, Distributor, Independent Operator, Manufacturer, Operator.|"Manufacturer"|ENUM| | |
|Information Provider GLN|informationProviderGLN|The entity providing this information. Usually points to the primary business GLN listed in the spreadsheet or database.|GS1 US, Inc. "0614141000005"|NUMBER| | |
|GDSN GLN Type|GDSNGLNType|Multiple types allowed. Options include: Brand Owner GLN, Manufacturer GLN, Recipient Provider GLN, Source Provider GLN, Information Provider GLN.|"Brand Owner GLN"|ENUM|15|34|
|Replaced GLN|replaced GLN|The GLN assigned to this location previously, if any.|"1234567890128"|NUMBER|13|13|


# Drawbacks
[drawbacks]: #drawbacks

Many of the approaches taken here are similar or the same as the approach 
taken with Grid Product. Thus, many of the drawbacks mentioned in the Grid 
Product RFC that haven't yet been addressed will apply to Grid Location as well.
This RFC intentionally re-uses the current structure where possible as 
cross-cutting enhancements would be better in separate RFCs.

This implementation does not account for transfer-of-ownership scenarios 
(mergers, acquisitions), and should be implemented as a supplemental RFC at a 
later time.


# Rationale and alternatives
[alternatives]: #alternatives

Starting with capturing a base location allows us to later extend the definition
in the future, addressing items such as:
- financial account information
- tax registration numbers (VAT number)
- delivery requirements or restrictions
- facility specification (operating hours, time zone)
- digital locations (network address, Production/Test/Development)

Above we use the terminology LocationNamespace, which is used in the same manner
as in the Product RFC, where it is called ProductNamespace. The current Product 
codebase, however, calls this ProductType. Because “locationType” is a GS1 field,
this RFC remains consistent with the use in the Product RFC, with the assumption
that the Product codebase should be changed to match the original RFC in this respect.

Note from GS1: GLNs and GTINs are formed from an organization’s GS1 Company 
Prefix which means the numbering sequence is identical. As such, GLNs and GTINs
should be maintained in separated database records.


# Prior art
[prior-art]: #prior-art

[Product RFC](https://github.com/hyperledger/grid-rfcs/blob/master/text/0005-product.md "Product RFC")


# Unresolved questions
[unresolved]: #unresolved-questions

- GS1 attribution details regarding required/optional attributes, field type and
minimum/maximum field values were initially sourced from the _GS1 US Data Hub | 
Location User Guide_. We are seeking validation GS1 experts on attribute details
that exist within this document as well as input on missing attributes and/or 
details.
- Does GS1 assign one or more usage types (legal entity, function, physical 
location, digital location) to a given Location? If yes, what field is used and
how do required/optional attribution differ for each usage type? See [GS1 Usage 
Types](https://www.gs1.org/1/glnrules/en/guideline/230 "GS1 Usage Types").
