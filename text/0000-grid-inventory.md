- Feature Name: (fill me in with a unique identifier, my_awesome_feature)
- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- Hyperledger Grid Issue: (leave this empty)

# Summary
[summary]: #summary

This RFC proposes a generic and extensible framework for a Hyperledger Grid
Inventory component, and is inspired by GS1’s EDI Inventory Report - Business
Message Standard. GS1 is a widely used data standard in enterprises and, given
that positioning and familiarity, Grid support feels natural. Additional
implementations of Grid Inventory may derive from or extend this
implementation.

Grid Inventory leverages other extensible Hyperledger Grid components
including: Grid Product, and Grid Location. Grid Inventory aims to provide
current inventory quantity & status visibility for each Grid Product and Grid
Location defined within the Grid Inventory smart contract between parties, as
well as a summary of relevant inventory events that occurred within a
time-range.  It includes transactions for generating summary inventory report
information from external systems of record and recording the results in state.

Grid Inventory also aims to serve as a foundation for future potential Grid
capabilities that may require inventory related information, such as purchasing
and logistics.

# Motivation
[motivation]: #motivation

The Grid Inventory implementation is designed to enable shared visibility of
summary inventory information for any defined Grid Product and Grid Location
between participants.  Management of inventory levels is a common concept
within supply chain solutions and is a key primary business activity related
to all purchases of physical goods from suppliers, as well as all sales of
physical goods to customers.  Management of inventory levels is typically
coordinated very manually and very independently between parties across siloed
systems and processes.  This siloed and manual coordination of inventory
between parties can lead to substantial pain-points, including:
inefficiencies, waste, and poor coordination of inventory levels across
the supply chain.

This Grid Inventory concept aims to solve these issues by offering a common
industry solution for vertically integrating inventory information between
parties existing systems of record.

The Grid Inventory design addresses several use cases, including:

- View Current Inventory Status Data: Viewing current summary inventory
  quantities and inventory statuses between trading partners for defined Grid
  Products and Grid Locations.

- View Historical Inventory Activity Data: Viewing historical inventory
  activity between trading partners for defined Grid Products and Grid
  Locations.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

TBD - based on discussion.  We are exploring designs based on:

 - GS1’s EDI Inventory Report - Business Message Standard.
 - GS1's X12 EDI Standards.
 - GS1's EPCIS Standard.
 - Other standards as relevant.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

TBD - based on discussion.

# Drawbacks
[drawbacks]: #drawbacks

By leveraging GS1’s EDI Inventory Report standards, only a summary view of
inventory information can be visible between parties.  Visibility of batch/lot
or serialized product instance information will not be visible in inventory
using this initial Grid Inventory design.  In addition, in order for this
solution to offer maximum value, it will likely require behavioral changes by
end-users who are transacting in existing systems to perform inventory stock
adjustment or movement transactions more real-time and more frequently.  Often,
these production reporting transactions are done not in real-time, for example
at the end of each shift, week, or even in some cases at month-end.
More frequent, or ideally real-time submission of transactions can help ensure
the physical and digital versions of the supply chain remain in sync.

# Rationale and alternatives
[alternatives]: #alternatives

A design for Grid Inventory to offer inventory quantity, status and activity
visibility between parties feels like a natural next step for leveraging Grid
Product and Grid Location.  Designing this in alignment with GS1’as EDI
Inventory Report standard also feels natural given Grid’s alignment
with other GS1 standards. GS1’s EPCIS (Electronic Product Code
Information Services) standard was also considered for this design, however
implementing a design based on EPCIS would largely be too large of a scope
for the initial Grid Inventory RFC.  Future Grid Inventory related RFC’s
could consider leveraging EPCIS concepts, which would enable event visibility
at a more granular Batch/Lot or Serialized instance level.


# Prior art
[prior-art]: #prior-art

- Grid Product
- Grid Location
- GS1 EDI XML Standards 3.0 Inventory Report


# Unresolved questions
[unresolved]: #unresolved-questions

- There may be additional Inventory status values, as well as inventory
  activity type values within GS1’s implementation guide for this EDI standard.
  This design considered all that were shown in available examples of the
  standard, however a discussion with GS1 should take place to ensure none
  are missing prior to moving to a final comment period.

- Future extensions of this RFC can be considered such as supporting batch/lot
  information, or concepts from GS1’s EPCIS design.

- The scope of this RFC has been limited to retrieving
  summary inventory information from external systems and sharing in-state.
  Future RFC’s can be considered for transactions to update external systems
  of record, including: Quantity Adjustments, Movement Transactions, and
  Transformation Events for supporting traceability use-cases.
