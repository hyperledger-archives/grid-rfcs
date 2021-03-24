- Feature Name: Grid Purchase Order
- Start Date: 2020-02-25
- RFC PR: 25
- Hyperledger Grid Issue:

# Table of Contents

- [Summary](0025-purchase-order.md#summary)
- [Motivation](0025-purchase-order.md#motivation)
- [Guide-level Explanation](0025-purchase-order.md#guide-level-explanation)
- [Reference-level Explanation](0025-purchase-order.md#reference-level-explanation)
- [Drawbacks](0025-purchase-order.md#drawbacks)
- [Rationale and Alternatives](0025-purchase-order.md#rationale-and-alternatives)
- [Prior Art](0025-purchase-order.md#prior-art)
- [Unresolved Questions](0025-purchase-order.md#unresolved-questions)

# Summary
[summary]: #summary

Communicating Purchase Order details via email, EDI or other traditional methods
creates uncertainty for both buyers and sellers today because they lack a shared
view of the order’s status and contents. Siloed views of order data can lead to
misalignment and result in undesirable outcomes such as a customer dispute.
Hyperledger Grid Purchase Order aims to address this challenge by providing a
mechanism for trade partners to collaborate on the creation and modification of
a purchase order, all while enjoying a shared view into the state of the order.

The design allows a buyer to order a specified quantity of goods and services
from a seller for a single shipment to a single location. The business use case
that inspired this design is that of a vendor-managed purchasing relationship.
However, the design was broadened to also support a more traditional procurement
relationship between a buyer and a seller. As a reviewer of this document, we
request that you comment on how this design does/does not support your purchasing
needs.

Grid Purchase Order leverages other Hyperledger Grid components including Grid
Identity, Grid Workflow, Grid Product, and Grid Location. In the near-term, this
business capability is focused on solving a need to better communicate and
collaborate on purchasing information. Looking to the future, it sets the stage
for further integration, both upstream and downstream, with order fulfillment
and settlement business processes.

# Motivation
[motivation]: #motivation

The Grid Purchase Order implementation is designed to share purchasing
information between trade partners. The purchase of goods and services is a
primary activity within the supply chain and the communication of order
information occurs in various ways today: by phone, email, SMS, eCommerce
marketplaces, Electronic Data Interchange (EDI), etc. Both the manual
coordination and automated, point-to-point sharing of purchasing information
between trade partners present challenges within day-to-day supply chain
operations. Pain points include: poor data accuracy stemming from manual data
entry errors, discrepancies between systems which impact receiving or result in
financial claims, costs related to administrative/clerical time, and management
of custom integrations.

The Grid Purchase Order concept aims to address these pain points by offering a
common industry solution for sharing purchase order information between trade
partners on Grid. Trade partners have the option to integrate this Grid
component with existing systems of record.

Expected outcomes include:

- Improved cost efficiency. An organization’s financial results can benefit
  from a reduction in administrative time spent manually inputting data,
  reconciling transactional differences with trade partners, performing
  credit/debit adjustments, addressing receiving problems, and handling product
  returns.
- Improved transaction accuracy. Automated sharing of purchasing data can reduce
errors that stem from manual data entry.

- Increased speed. Large volumes of data can be quickly communicated between
organizations, leading to faster response times, improved buying decisions and
production planning, greater customer satisfaction, and visibility into order
status.

- Improved productivity. Less time spent comparing documents and resolving discrepancies
means team members can focus on move value-add business opportunities.

# Guide-level Explanation
[guide-level-explanation]: #guide-level-explanation

A purchase order is a business transaction that specifies the details of goods
or services a Buyer ordered from a Seller under agreed upon conditions. The
transaction contains information such as the trade items ordered (with
associated quantity and price), shipping details, payment terms, etc.

In the simplest business scenario, a purchase order is issued by a buyer,
confirmed in full by a seller, and fulfilled with no further collaboration
necessary. In more involved business scenarios – where product is out of stock,
or requested delivery dates must be adjusted – trade partners interact to reach
agreement and modify purchase order records. This RFC aims to offer a solution
to collaboration needs – both simple and complex -- by defining a default
Purchase Order design. The design leverages Grid Workflow, which requires the
use of a state transition model, to define the states through which a purchase
order may flow. The design also leverages Grid Identity to configure role-based
access control. The content that follows describes the default design.

## Actions

This design introduces four smart contract actions that, with the appropriate
permissions, a user of the system may perform. An action represents a change
to the shared state of a purchase order.

- Create PO. Allows for the creation of a new purchase order record.
- Update PO. Allows for the modification of an existing purchase order record.
- Create Version. Allows for the creation of a new version of a purchase order.
- Update Version. Allows for the modification of an existing purchase order
version.

## Workflow

The design introduces three sub-workflows that each play a role in managing the
shared view of a purchase order. The Purchase Order Sub-Workflow manages the
business process states in which a purchase order record may exist. The System
of Record Version Sub-Workflow governs how actions taken on one or many versions
of a purchase order influence the shared state of the overall record. The
Collaborative Version Sub-Workflow also governs actions taken on versions, but
caters specifically to trade partners who use Grid as their purchase order
system of record.

These sub-workflows represent an out-of-the-box pattern for purchase order
collaboration. Alternative sub-workflows may be configured by trade partners
who need to support a different view of the business process. Partners could
configure different workflow states, constraints, and/or permissions to achieve
the goals of their business relationship.

Each sub-workflow is comprised of a series of states which define how a purchase
order evolves from start to finish. Some workflows progress in a linear fashion;
other workflows accommodate progression and regression based on the actions
taken by trade partners. Each workflow state contains a list of constraints –
think business rules – that must be met before a purchase order can enter that
state. In addition, each workflow state contains a list of permissions that
outline what actions may be taken, by whom, at that stage in a purchase order’s
life.

### Purchase Order Sub-Workflow

![Purchase Order Sub-Workflow](../images/0025-purchase-order-rfc/po_sub_workflow.png)

The Purchase Order Sub-Workflow defines the states through which a purchase
order record may progress. A record is like a container that can hold multiple
versions of a purchase order, both historical and active in nature. When we
define a state at the record level, we give meaning to the overall progression
of the purchase order. This summary state is informed by actions taken on
underlying versions of the purchase order.

The sub-workflow consists of the following states: Issued, Confirmed, and
Closed. Two business rules (also known as constraints) dictate when a purchase
order may proceed to the next state.

- Accepted. A boolean that must exist on an object. If the Accepted constraint
field on a purchase order version is set to True, then this field is set to
True. Only one version of a purchase order may have this workflow constraint
set to True.

- Closed. A boolean that must exist on an object. If the purchase order record
has been made final, then this field is set to “True” and further changes to
the contents of the purchase order are prohibited.

Beginning state. A buyer is able to create a purchase order record alongside
the creation of a purchase order version. A seller is able to create a
purchase order version. Versions enter the Purchase Order System of Record
Version Sub-Workflow at different points depending on permission assignments
(more on permissions later).

Issued state\*. When one or more versions have been created and no version
fulfills the Accepted constraint, then the purchase order record resides in
Issued state. During this state, buyers and sellers are permitted to create
new versions and update existing versions in line with their respective workflow
permissions. A seller contains special permission to transition a purchase order
into Confirmed state by satisfying the Accepted constraint.

\*Note: Some trade partners operate with a high level of trust and do not
require confirmation of purchase order contents. In this scenario, a system
agent could be created and granted permission to auto-transition a purchase
order from Issued state to Confirmed state, altogether bypassing the Version
Sub-Workflows introduced below.

Confirmed state. An open purchase order enters Confirmed state when both the
buyer and seller have fully accepted its contents. It is assumed a separate,
but related, sales order fulfillment process is underway. During this state,
both a buyer and seller are permitted to request changes to the purchase order
by issuing a new version. A seller is also permitted to finalize a purchase
order by transitioning it into Closed state. Note: If an Accepted version is
cancelled, then the record returns to Issued state.

Closed state. A purchase order was made final; it can no longer be changed. To
transition to this state, the Closed constraint must be set to True. Once in
Closed state, no permissions are available to any persona. In practice,
organizations apply different business logic to determine when a purchase order
is closed. Some orgs may accept changes up until one hour before product ships,
others may prohibit changes after a 48 hour lead time has elapsed, etc. To
maintain flexibility, orgs may use the Grid Integration Component to manage
the rules that control this constraint.

The Purchase Order Sub-Workflow makes the following assumptions.

- Business process states regarding the shipment and/or billing of an order
belong to the sales order entity, not the purchase order entity.

### Purchase Order System of Record Version Sub-Workflow

![PO Record of version Sub-Workflow](../images/0025-purchase-order-rfc/po_sor_sub_workflow_1_of_2.png)

The Purchase Order System of Record Version Sub-Workflow defines how trade
partners may collaborate on versions of a purchase order. Actions taken within
this workflow inform valid transitions between business process states. More
specifically, acceptance of a purchase order in this version workflow enables
the transition of a purchase order record from Issued state to Confirmed state
within the Purchase Order Sub-Workflow described above.

This workflow is designed with external systems in mind. Given the system of
record is a buyer’s ERP system, when a seller introduces modifications to a
record, then changes are treated as guidance until the buyer issues an updated
version. This rule is enforced via permissions at the two starting positions
within the sub-workflow. A buyer is permitted to propose a purchase order
version; a buyer or seller is permitted to initiate a draft for collaboration.

Two constraints control the movement of a version from one state to another.

- Complete. A boolean that must exist on an object. If all mandatory data
elements have a value defined, then this field is set to True.

- Accepted. A boolean that must exist on an object. If both trade partners have
agreed to the contents in full, then this field is set to True. Only one version
may be accepted at a time.

Beginning state (pre-Proposed). A buyer has permission to create a purchase
order version and propose it to the seller.

Proposed state. A buyer submitted a version for review and approval by the
seller. A buyer has permission to update the proposal before the seller takes
action or can cancel the version in favor of a better one. A seller may reject,
accept or modify the version to the degree their granular update permissions
allow. Note: A seller may rely on ERP business logic to determine the
appropriate transition from Proposed state. This logic, such as checking
contractual agreements or inventory availability, lives outside of the scope of
Grid at this time.

Rejected state. A version was rejected in full. The buyer and seller can take no
further action on the version. In the case a seller rejected with a reason and
the buyer wants to alter the order based on the seller’s feedback, the buyer
may issue a new version.

Accepted state. A version was reviewed and accepted in full. A separate sales
order entity (out of scope of Grid Purchase Order) is created within the
seller’s system. The version is represented within the Confirmed state within
the Purchase Order Sub-Workflow and takes the place of a previously accepted
purchase order (if applicable). A buyer is permitted to cancel an accepted
purchase order so long as a goods receipt or invoice has yet to be generated.
This logic lives outside the scope of Grid at this time and is a concern of
the Grid Integration component. A seller may take no further action on this
version.

Obsolete state. A version was either cancelled or replaced by a more recently
Accepted version. The buyer and seller can take no further action on the
version.

Modified state. A version is partially confirmed (often referred to as
‘accepted with changes’) and awaiting further approval. At this time, a seller
generally creates a related sales order in their system of record. A buyer may
move the version to Obsolete and issue a new version based on the seller’s
guidance. A seller may update the suggested modifications so long as the buyer
has yet to take action. An editor may convert the version into a draft for
further collaboration.

The second half of the System of Record Version Sub-Workflow provides trade
partners a shared editing capability where a version may be co-created. The
flow represents an “offer” process of sorts and benefits non-buying
organizations who, as part of their business relationship, suggest purchase
order details to a buying organization to inspire the issuance of a purchase
order. A new constraint is introduced to control the movement of a version
from one state to another.

- Draft. A boolean that must exist on an object. If the purchase order version
is a draft that exists for collaboration purposes, then this field must be set
to True.

![PO Record of version Sub-Workflow](../images/0025-purchase-order-rfc/po_sor_sub_workflow_2_of_2.png)

Beginning state. An organization with po.draft permissions created a version
for collaboration purposes.

Editable state. The contents of the version are being populated by one or more
organizations. Any organization has permission to submit the draft for review
or close it in favor of a better version.

Review state. An organization has proposed the version for review by another
organization. Another organization has permission to approve the version by
moving it to Composed state. The organization may also decline the version by
adding a rejection reason (optional) or move it back to Editable state for
further modification.

Composed state. Both organizations are in agreement on the contents of the
version. The buying organization retains the decision to issue a formal
purchase order based on the drafted contents.

Declined state. An organization has declined the version content. An
organization may move it back to Editable state for further modification or
choose to cancel the version.

Cancelled state. A version is no longer relevant; no further edits may be
introduced to the version.

### Purchase Order Collaborative Sub-Workflow

![PO Collaborative Sub-Workflow](../images/0025-purchase-order-rfc/po_collaborative_sub_workflow.png)

The Collaborative Sub-Workflow offers trade partners a means to manage
purchase order records exclusively on Grid. Without the need to maintain data
integrity with external systems of record, more actions are available to trade
partners. This workflow illustrates a potential long-term vision for this
business capability on Grid. A detailed explanation has been omitted for
brevity.

## Workflow State Permissions

As seen in the above diagrams, this RFC defines a set of permissions at each
state in a sub-workflow. Permissions are assigned to an alias, such as po.buyer,
and dictate what actions a user with said permissions may take at that stage of
a purchase order’s life. An organization may craft roles within Grid Identity by
assigning one or more aliases to a user. This permissions design is not
exhaustive; additional aliases could be introduced in the future.

When creating a purchase order version, mandatory data elements must have a value
defined. Similarly, when updating a purchase order, a subset of fields cannot be
edited. These requirements are hard-coded and enforced by smart contract logic.
Should a source system dictate additional data elements that must remain
unchanged, validation may be introduced through the integration component.

## Transactions

Purchase Orders are managed by submitting transactions to Hyperledger Grid,
which will process them with the Grid Purchase Order smart contract. The
following transactions are supported:

- Create_PO. Create a purchase order record and store it in state.
- Update_PO. Update the properties of a purchase order already in state.
- Create_Version. Create a version of a purchase order and store it in state.
- Update_Version. Update the properties of a purchase order version already
in state.

## Examples

### General Business Scenario
In this example, two organizations are participating on a network: Fred’s Farm
and The General Store. Fred’s Farm represents a buyer who purchases animal
feed and other on-farm equipment from a seller, The General Store. The
organizations choose to define roles as follows.

```
Name: FredsFarm.Admin
Permissions:
    - po.buyer
    - allowed _organizations: [ ]
    - inherit_from: [ ]

Name: TheGeneralStore.Admin 
Permissions:
    - po.sellerconfirm
    - po.sellermodify
Allowed _organizations: [ ]
Inherit_from: [ ]
```

On June 1, 2018, Fred’s Farm issues a purchase order (PO8865) to The General
Store. The order contains two products:
- 50 tons of bulk dairy feed to be dropped off at Bin 2 on the farm.
- 25 bags of specialty feed.

Pricing is included for each line item. Fred’s Farm requests a delivery date of
June 4, provides instructions to the driver for how to access the farm gate,
and expects to receive an order confirmation from The General Store.

The General Store reviews the purchase order and sees Fred’s Farm used a
historical price for the specialty feed. The General Store requests modification
to the purchase order, citing a need to update the price for the second line
item from $10 per bag to $12 per bag.

Fred’s Farm reviews the suggested modifications and agrees with the updated
price. The buyer closes the initial version and issues a new version containing
the price modifications. The General Store accepts the new version and
fulfillment begins. Two days later, after their 48-hour lead time rule expired,
The General Store communicates the purchase order is final and no further
changes may be accepted.

![Example 1](../images/0025-purchase-order-rfc/po_example_1_of_2.png)

### Vendor-Managed Purchasing Scenario

In this example, The General Store manages the supply of animal feed on behalf
of Fred’s Farm and suggests purchase orders to Fred’s Farm based on demand
forecasts. Fred’s Farm has granted The General Store the ability to draft
purchase orders without the need for Fred’s Farm to review the contents (a
system agent auto-accepts the purchase order in the Review state). Formally
issued purchase orders will then be issued by Fred’s Farm system of record,
based on composed contents.

On August 1, 2018, The General Store submits a draft purchase order to Fred’s
Farm based on projected inventory needs. The proposal calls for 35 bags of
horse feed to be delivered to the farm on August 5. Per their business
agreement, Fred’s Farm uses the draft contents to issue a formal purchase order
from the system of record. The next day, The General Store submits a new draft
that contains a modified delivery date (inventory forecasts changed based on
on-farm feed consumption). Fred’s Farm again loads the updates into their system
of record and issues an updated purchase order.

![Example 2](../images/0025-purchase-order-rfc/po_example_2_of_2.png)

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## State

### PurchaseOrder

Purchase order state is composed of several objects with the primary object
being the `PurchaseOrder`. `PurchaseOrder` contains the following fields

- `uid` - Unique identifier for Purchase Order
- `workflow_status` - Workflow status. Values are defined by the smart contract
workflows
- `is_closed` - True if purchase order was closed, false otherwise
- `accepted_version_id` - The ID of the purchase order that has been accepted
- `versions` - A list of different purchase order versions
- `created_at` - When the document was created in seconds since January 1, 1970

```
message PurchaseOrder {
  required string uid = 1;
  required string workflow_status = 2;
  repeated PurchaseOrderVersion versions = 3;
  repeated uint64 created_at = 4;
}
```

### PurchaseOrderVersion

`PurchaseOrderVersion` represents a new version of a purchase order.
`PurchaseOrderVersion`s move through the version workflow if `is_draft` is
false, or through the draft workflow if `is_draft` is true. Its status within
those workflows is indicated by `workflow_status`. A `PurchaseOrderVersion` is
identified by its `version_id` which is globally unique. `revisions` is a list
of revisions to version, and `current_revision_number` is the identifier of the
most recent revision.

```
message PurchaseOrderVersion {
  required string version_id = 1;
  required string workflow_status = 2;
  required boolean is_draft = 3;
  required uint64 current_revision_number = 4;
  repeated PurchaseOrderRevision revisions = 5;
}
```

### PurchaseOrderRevision

`PurchaseOrderRevision` holds the editable fields of a purhcase order, the
time the revision was created, and the public key of the submitter.

```
message PurchaseOrderRevision {
  required uint64 revision_number = 1;
  required string submitter = 2;
  required uint64 created_at = 3;

  string order_identification = 3;
  required string order_type_code = 4;
  required TransactionalPartyType buyer = 5;
  required TransactionalPartyType seller = 6;
  uint64 creation_date_time = 7;
  uint64 last_update_date_time = 8;
  string revision_number = 9;
  required OrderLogisticalInformation order_logistical_information = 10;
  string additional_order_instruction = 11;
  AmountType total_monetary_amount_excluding_taxes = 12;
  AmountType total_monetary_amount_including_taxes = 13;
  AmountType total_tax_amount = 14;
  TransactionalPartyType bill_to = 15;
  TransactionalPartyType pickup_from = 16;
  TransactionalPartyType customs_brocker = 17;
  string contract_identification = 18;
  AmountTypeExchangeRateInformation currency_exchange_rate_information = 19;
  string incoterms_code = 20;
  string delivery_instructions = 21;
  repeated ValuePair avp_list = 22;
  string extension = 23;
  OrderResponseReasonCode order_response_reason_code = 24;
  uint64 amended_date_time_value = 25;
  repeated PaymentTerm payment_term = 26;
  repeated LineItem line_items = 27;
}
```

### AlternativeID

`AlternativeID` are the mechanism the smart contract will use to allow
purchase orders to be created and edited without having to specify a
purchase order number at creation. This is identical to the mechanism
outlined in the [Pike 2 RFC](https://github.com/hyperledger/grid-rfcs/pull/23).

The `id_type` is used the specify the field that will be used as an alternate ID,
and the `id` is the `uid` of the purchase order.

```
message AlternateID {
  string id_type = 1;
  string id = 2;
}
```

### Location Addressing in the Merkle-Radix State System

In order to uniquely locate Purchase Orders in the Merkle-Radix state system,
an address must be constructed which identifies the storage location of the
Purchase Order representation.

All Grid addresses are prefixed by the 6-hex-character namespace prefix
“621dee”. `PurchaseOrder` and `AlternativeID` are further prefixed under
the Grid namespace with reserved enumeration of `06`.

The remaining 62 characters of a `PurchaseOrder` address is calculated by taking
the first 60 characters of a SHA512 hash of its `uid` and concatenating it with
the prefix `00`.

```
“621dee” + “06” + “00” Sha512(uid)[:60]
```

The remaining 62 characters of a `AlternateID` address is calculated by taking
the first 60 characters of a SHA512 hash of its corresponding purchase order's
`uid` and concatenating it with the prefix `01`.

```
“621dee” + “06” + “01” + Sha512(uid)[:60]
```

### Sub Messages

Detailed definitions of fields and sub fields. These fields are subset
of the [GS1 order spec](https://www.gs1.org/sites/default/files/docs/EDI/ecom-xml/functional-user-guide/3_4/HTML/O/a1.htm?https://www.gs1.org/sites/default/files/docs/EDI/ecom-xml/functional-user-guide/3_4/HTML/O/a5.htm).

```
message LineItem {
  required string line_item_number = 1;
  required QuantityType requested_quantity = 2;
  QuantityType confirmed_quantity = 3;
  required TransactionalTradeItem transactional_trade_item = 4;
  string lineItemActionCode = 5;
  AmountType net_amount = 6;
  AmountType net_price = 7;
  AmountType total_monetary_amount_excluding_taxes = 8;
  AmountType total_monetary_amount_including_taxes = 9;
  AmountType item_price_base_quantity = 10;
  repeated ValuePair avp_list = 11;
  string extension = 12;
  string substituteInformation = 13;
  string lineItemChangeIndicator = 14;
  string orderResponseReasonCode = 15;
  string additionalOrderLineItemInstruction = 16;

}

message OrderLogisticalInformation {
  string commodity_type_code = 1;
  string shipment_split_method_code = 2;
  TransactionalPartyType ship_from = 3;
  TransactionalPartyType ship_to = 4;
  TransactionalPartyType inventory_location = 5;
  TransactionalPartyType ultimate_consignee = 6;
  TransactionalPartyType intermediate_delivery_party = 7;
  OrderLogisticalDateInformation order_logistical_date_information = 8;
}

message OrderLogisticalDateInformation {
  DateRange requested_delivery_date_range = 1;
  DateRange requested_ship_date_range = 2;
  DateRange requested_delivery_date_range_at_ultimate_consignee = 3;
  uint64 requested_ship_date_time = 4;
  uint64 requested_pick_up_date_time = 5;
  uint64 requested_delivery_date_time_at_ultimate_consignee = 6;
}

message TransactionalTradeItem {
  string gtin = 1;
  repeated string additional_trade_item_identification = 2;
  QuantityType trade_item_quantity = 3;
  string trade_item_description = 4;
  string product_variant_identifier = 5;
  string item_type_code = 6;
  string trade_item_data_owner = 7;
  string butter_fat_reference = 8;
  TransactionalItemData transactional_item_data = 9;
  repeated ColourType colour = 10;
  repeated SizeType size = 11;
  TradeItemClassification trade_item_classification = 12;
  repeated ValuePair avp_list = 13;
}

message TradeItemClassification {
  string gpc_category_code = 1;
  repeated string additional_trade_item_classification_code = 2;
  string gpc_category_name = 3;
  GpcAttribute gpc_attribute = 4;
}

message GpcAttribute {
  string gpc_attribute_type_code = 1;
  string gpc_attribute_value_code = 2;
  string gpc_attribute_type_name = 3;
  string gpc_attribute_value_name = 4;
}

message TransactionalItemData {
  uint64 available_for_sale_date = 1;
  string batch_number = 2;
  string best_before_date = 3;
  string country_of_origin = 4;
  string item_expiration_date = 5;
  string lot_number = 6;
  string packaging_date = 7;
  string production_date = 8;
  QuantityType product_quality_indication = 9;
  string sell_by_date = 10;
  repeated string serial_number = 11;
  string shelf_life = 12;
  QuantityType trade_item_quantity = 13;
  bool item_in_contact_with_food_product = 14;
  repeated UnitMeasurementType transactional_item_weight = 15;
  repeated UnitMeasurementType transactional_item_volume = 16;
  repeated NumberRange serial_number_range = 17;
  repeated DimensionType transactional_item_dimensions = 18;
  TransactionalItemLogisticUnitInformation \
     transactional_item_logistic_unit_information = 19;
  TransactionalItemDataCarrierAndIdentification \
    transactional_item_data_carrier_and_identification = 20;
  TradeItemWaste trade_item_waste = 21;
  TransactionalItemOrganicInformation \
    transactional_item_organic_information = 22;
  repeated ValuePair avp_ist = 23; 
}

message TransactionalItemOrganicInformation {
  bool is_trade_item_organic = 1;
  OrganicCertification organic_certification = 2;
}

message OrganicCertification {
  string item_certification_agency = 1;
  string item_certification_standard = 2;
  string item_certification_value = 3;
}

message TradeItemWaste {
  string waste_identification = 1;
  repeated string type_of_waste = 2;
}

message TransactionalItemDataCarrierAndIdentification {
  string gs1_transactional_item_identification_key = 1;
  string data_carrier = 2;
}

message TransactionalItemLogisticUnitInformation {
  uint64 number_of_layers = 1;
  uint64 number_of_units_per_layer = 2;
  uint64 number_of_units_per_pallet = 3;
  string packaging_terms = 4;
  string package_type_code = 5;
  uint64 maximum_stacking_factor = 6;
  string returnable_package_transport_cost_payment = 7;
  DimensionType dimensions_of_logistic_unit = 8;
}

message NumberRange {
  string min_value = 1;
  string max_value = 2;
}

message DimensionType {
  double height = 1;
  double width = 2;
  double depth = 3;
}

message SizeType {
  string descriptive_size  = 1;
  string size_code = 2;
}

message AmountType {
  string currency_code = 1;
  uint64 amount = 2;
}

message MeasurementType {
  string measurement_unit_code = 1;
  uint64 amount = 2;
}

message UnitMeasurementType  {
  string measurment_type = 1;
  string value = 2;
}

message QuantityType {
  string measurement_code = 1;
  uint64 amount = 2;
}

message ColourType {
  string colour_code = 1;
  string colour_description = 2;
}

message DateRange {
  uint64 begin = 1;
  uint64 end = 2;
}

message AmountTypeExchangeRateInformation {
  string currency_conversion_from_code = 1;
  string currency_conversion_to_code = 2;
  double exchange_rate = 3;
  uint64 exchange_rate_date_time = 4;
}

message ValuePair {
  string attributeName = 1;
  string qualifierCodeName = 2;
  string qualifierCodeList = 3;
  string qualifierCodeListVersion = 4;
  string value = 5;
}

message TransactionalPartyType {
  string gln = 1;
  repeated string additionalPartyIdentification = 2;
  Address address = 3;
  Contact contact = 4;
}

message Address {
  string city = 1;
  string cityCode = 2;
  string countryCode = 3;
  string countyCode = 4;
  string crossStreet = 5;
  string currencyOfPartyCode = 6;
  string languageOfThePartyCode = 7;
  string name = 8;
  string pOBoxNumber = 9;
  string postalCode = 10;
  string provinceCode = 11;
  string state = 12;
  string streetAddressOne = 13;
  string streetAddressTwo = 14;
  string streetAddressThree = 15;
  GeographicalCoordinates geographicalCoordinates = 16;
}

message GeographicalCoordinates {
  string latitude = 1;
  string longitude = 2;
}

message Contact {
  string contactTypeCode = 1;
  string personName = 2;
  string departmentName = 3;
  string jobTitle = 4;
  repeated string responsibility = 5;
  repeated CommunicationChannel communicationChannel = 6;
  repeated CommunicationChannel afterHoursCommunicationChannel = 7;
}

message CommunicationChannel {
  string communicationChannelCode = 1;
  string communicationValue = 2;
  string communicationChannelName = 3;
}

message PaymentTerm {
  string paymentTermsEventCode = 1;
  string paymentTermsTypeCode = 2;
  uint64 proximoCutOffDay = 3;
  uint64 netPaymentDue = 4;
  InstallmentDue installmentDue = 5;
  repeated PaymentTermsDiscount paymentTermsDiscount = 6;
  repeated PaymentMethod paymentMethod = 7;
  repeated SEPAReference sEPAReference = 8;
}

message SEPAReference {
  string transactionalReferenceTypeCode = 1;
  string transactionalReferenceValue = 2;
  uint64 transactionalReferenceDateTime = 3;
}

message PaymentMethod {
  string paymentMethodCode = 1;
  string paymentMethodIdentification = 2;
  string automatedClearingHousePaymentFormat = 3;
}

message PaymentTermsDiscount {
  string discountType = 1;
  AmountType discountAmount = 2;
  double discountPercent = 3;
  TimePeriodDue paymentTimePeriod = 4;
  string discountDescription = 5;
}

message InstallmentDue {
  uint64 percentDue = 1;
  uint64 paymentTimePeriod = 2;
}

message TimePeriodDue {
  string timeMeasurementUnitCode = 1;
  uint64 amount = 2;
}
```

## Transaction Payload and Execution

### PurchaseOrderPayload Transaction

`PurchaseOrderPayload` contains an action `enum` and the associated action
payload. This allows for the action payload to be dispatched to the appropriate
logic. Only the defined actions are available and only one action payload should
be defined in the `PurchaseOrderPayload`. `PurchaseOrderPayload` contains the 
following required fields:
- `action` - Action enum, indicating the payload type
- `org_id` - The Pike organization that is sending the payload
- `public_key` - The public key of a Pike agent that is sending the payload
- `timestamp` - Time the payload was created

```
message PurchaseOrderPayload {
  enum Action {
    UNSET_ACTION = 0;
    CREATE_PO = 1;
    CREATE_VERSION = 2;
    UPDATE_VERSION = 3;
  }
  required Action action = 1;
  required string org_id = 2;
  required string public_key = 3;
  required uint64 timestamp = 4;

  CreatePurchaseOrderPayload create_po_payload = 5;
  CreateVersionPayload create_version_payload = 6;
  UpdateVersionPayload update_version_payload = 7;
}


message CreatePurchaseOrderPayload {
  required uint64 created_at = 1;
  PurchaseOrderRevision revision = 2;
}

message CreateVersionPayload {
  boolean is_draft = 1;
  PurchaseOrderRevision revision = 2;
}

message UpdateVersionPayload {
  PurchaseOrderRevision revision = 1;
}
```

### Create Purchase Order Payload

`CreatePurchaseOrderPayload` add a new purchase order to state. An optional
`PurchaseOrderVersion` can be included in the payload representing the initial
version of the purchase order.

Validation Requirements:
- The `org_id` must exist in Pike for it be a valid transaction
- The `public_key` must belong to a Pike agent that is a part of the
organization designated by `org_id`
- The Pike agent must have the permission `can-create-po` 
- All fields marked `required` in the `CreatePurchaseOrderPayload` must be supplied

Inputs:

Address of Pike `cad11d`
Address of Grid Purchase Order `621dee05`

Outputs:

Address of Grid Purchase Order `621dee05`

### Create Version Payload

`CreateVersionPayload` creates a new `PurchaseOrderVersion`.

Validation Requirements
- The `org_id` must exist in Pike for it be a valid transaction 
- The `public_key` must belong to a Pike agent that is a part of the organization designated by `org_id` 
- The Pike agent must have the permission `can-create-po-version`
- All fields marked `required` in the `CreateVersionPayload` must be supplied

Inputs:

Address of Pike `cad11d`
Address of Grid Purchase Order `621dee05`

Outputs:

Address of Grid Purchase Order `621dee05`

### Update Version Payload

`UpdateVersionPayload` updates an existing `PurchaseOrderVersion`
with a new revision.

Validation Requirements:
- The `org_id` must exist in Pike for it be a valid transaction 
- The `public_key` must belong to a Pike agent that is a part of the organization designated by `org_id` 
- All fields marked `required` in the `UpdateVersionPayload` must be supplied

Inputs:

Address of Pike `cad11d`
Address of Grid Purchase Order `621dee05`

Outputs:

Address of Grid Purchase Order `621dee05`

# Drawbacks
[drawbacks]: #drawbacks

- Grid Purchase Order forms an opinion on default Purchase Order workflow.
We researched industry standards in an effort to uncover a standard business
process for managing purchase order transactions across trade partners. We did
not uncover a standard workflow so we designed one in its absence (keeping a
design principle of flexibility in mind). The design aims to be generic where
possible to accommodate the integration of business-specific logic.

- Grid Purchase Order hardcodes data elements rather than utilizing Grid Schema.
This design assumes purchase order data elements are well known and can be
available for trade partners to use/not use as they wish. By hardcoding the list
of data elements, we know what fields are available and can drive business logic
from them (use of Schema makes this more difficult). However, if it is important
for trade partners to restrict the use of some of these well-known fields, then
additional mapping complexity may be introduced. Note: It is easier to
transition from a hardcode approach to use of Grid Schema than vice-versa.

- Grid Purchase Order defines a single data model. This design aims to
semantically represent core data elements found across various specifications
by using GS1’s Order and Order Response Business Message Standards (XML 3.4).
The XML standards recognize lower adoption today than other legacy EDI message
standards used in North America (ANSI) and internationally (EANCOM, EDIFACT).
However, the standards offer a unified view of data that bridges geographies.
Note: Data mapping and translation becomes a concern of the integration component.

# Rationale and Alternatives
[alternatives]: #alternatives

Why is this design the best in the space of possible designs?

- Trade partners may collaborate on a shared version of the truth. This design
aims to modernize legacy EDI communication methods by allowing organizations to
more effectively partner on the creation and modification of purchase orders. It
offers a path to co-create versions and present a shared view of the status of a
purchase order.

- Organizations have flexibility in assigning permissions and roles. Through the
use of Grid Identity, trade partners may configure and assign permissions within
the Purchase Order workflows to reflect the distribution of roles within their
business partnership. An organization can delegate a role to another
organization, split permissions across multiple internal roles, etc.

- The data model is extensible. This RFC is focused on capturing the data
elements that are essential to constructing a complete purchase order
transaction. This initial set of data fields may be enhanced by future RFCs to
include additional data structures such allowance charges, related documents,
subline item information, delivery schedules, etc.

What other designs have been considered and what is the rationale for not
choosing them?

- Request for Quotation and Quote. When exploring how to provide a vendor with
the ability to propose purchase orders to a buyer, we considered Quote-related
business processes as a potential pathway to achieve our goal. We found this
introduced transactional capabilities that exceed the scope of the business
scenario in focus. These processes could live within Grid in the future as a
predecessor to Grid Purchase Order.

- Replenishment Request and Proposal. We also considered Replenishment-related
business processes when exploring how to provide a vendor with the ability to
propose purchase order details to a buyer. This standard contains too many gaps
when compared against our goal. Core purchasing data elements are omitted and
unnecessary transactions were introduced.

- Design that accommodates multiple shipments. We found a variety of industry
standards noted a best practice to avoid the creation of purchase orders that
contain multiple shipments. Per one GS1 standard, “incorporating multiple
ship-to locations creates potential processing problems for the receiver, ASN
problems for the buyer, billing issues for the supplier, and payable issues for
the buyer.” This RFC encourages adherence to this best practice. A future RFC
could outline support for this business scenario as a design enhancement.

What is the impact of not doing this?

- Data will continue to be controlled independently. This distributed design
offers a paradigm shift where the master record is represented on Grid for
shared visibility. Establishing a shared source of truth will help improve
order-to-invoice matching and reduce claims/disputes related to misaligned data.

- Continued inefficiencies stemming from manual tasks. Collaboration across
trade partners on the proposal and modification of purchase orders often occurs
offline by way of phone calls, emails, etc. This design offers a mechanism to
collaborate in a more systematic way.

# Prior Art
[prior-art]: #prior-art

- [Grid Product RFC](https://github.com/hyperledger/grid-rfcs/blob/master/text/0005-product.md)
- [Grid Location RFC](https://github.com/hyperledger/grid-rfcs/blob/master/text/0020-location.md)
- [Grid Identity RFC](https://github.com/hyperledger/grid-rfcs/pull/23)
- [Grid Workflow RFC](https://github.com/hyperledger/grid-rfcs/pull/24)

# Unresolved Questions
[unresolved]: #unresolved-questions

## State Transition Model

- Do the default Purchase Order states as described in this RFC resonate with
the community and play nicely with existing business processes?

- Both parties have equal permissions when collaborating on a draft purchase
order. How will we enforce segregation of duties (e.g. when one party
transitions the order into Review state then only the other party may take action).

## Data Model & Permissions

- Must we define smart contract permissions separately from workflow state
permissions?

- Trade partners need the ability to define which data elements may be used
within a purchase order transaction to ensure compatibility with external
application constraints. Where should we control field definition: within Grid
or outside of Grid?

- Is it important to define a Data Type for data elements (e.g. numeric, text,
structure)?

- This design omits some data elements that appear specific to the transmission
of an EDI message such as DocumentStatusCode, ContentOwner, Sender,
HeaderVersion, etc. Do these fields need to be represented within Grid?

- Are there other data elements we should include in this initial design
(e.g. allowance charges)?

- Should we provide a supplemental artifact to the Community that explains
which data elements each granular permission controls?

## Data storage in state

- Do we want to store purchase orders as XML files or consume XML files as
part of the smart contract payloads?
