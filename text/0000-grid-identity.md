- Feature Name: grid-identity
- Start Date: 2020-12-08
- RFC PR: (leave this empty)
- Hyperledger Grid Issue: (leave this empty)

# Summary
[summary]: #summary

Grid Pike v2 is a role-based access control system for managing permissions
within Grid and specifically within smart contracts. Agents, which are
identified by their cryptographic public keys, are provided with permissions to
act on behalf of organizations. Grid Pike v2 is an evolution of the Pike v1
identity system used in Grid and Sawtooth.

# Motivation
[motivation]: #motivation

Pike v1, the existing identity system, lacks a few key features which are
necessary to fully capture delegation of permissions desired in more complex
systems (such as the upcoming Grid Purchase Order capability). The desired
capabilities are:

  - Organizations should have the ability to define organization-specific roles
    which are constructed from the permissions hard-coded into smart contracts.

  - Organizations should be able to delegate specific roles to other
    organizations to fulfill.

Much of the rest of the design remains the same.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

From a smart contract perspective, permission checks only involve two
components: a public key and the name of the smart-contract-defined permission.
All authorization checks thus can be written as:

  ```
  if (!has_permission(state, public_key, permission_name)) {
      // return invalid transaction status
  }
  ```

The public key is usually obtained from the transaction's signer’s public key.
Therefore, the authorization check determines whether the signer of the
transaction has the permission required. If the check fails, the transaction
will be considered invalid.

Grid Pike v2 defines the data structures and logic for the `has_permission()`
function, which is provided to the smart contract by the Grid SDK. The
implementation of `has_permission()` makes various calls to obtain portions of
state and thus requires a handle to state be passed into the function.

## Understanding permission checking in Pike

The current Pike implementation relies on organizations, agents, and roles:

![Pike Diagram](https://github.com/Cargill/grid-rfcs/blob/identity/images/grid-pike/pike1.svg)

In Pike, when `has_permission()` is evaluated, an agent is looked up via the
public key. The public key is the agent’s primary key and the agent is stored in
state using an address derived from the public key. The agent has a list of
roles, which are strings. The `has_permission()` function returns true if the
`permission_name` is in the list of roles and false if it is not.

Each Pike agent must be part of an organization. For many smart contracts in
Grid, the records created by smart contracts also include the ID of the
organization that they are associated with. For instance, a product record
includes the Pike organization ID of the organization that owns the product
record. In order to ensure that agents can't create unauthorized records on
behalf of an organization they are not a part of, smart contracts have to check
the organization ID in the record matches up with the agent's organization ID.
If not, the transaction is considered invalid.

This pattern is effective in preventing agents from acting on behalf of other
organizations. However, there may be cases where an organization needs to grant
certain permissions to another organization. With Pike this would not be
possible without specific workaround in each smart contract. This could lead to
inconsistency and sub-standard implementations across Grid. It is imperative
that Grid Pike v2 solves this problem.

## Redefining Roles

In Grid Pike v2 , roles are not directly used as permissions. Instead, a role is
defined as a list of permissions. The smart contracts check for permissions.
This level of indirection allows organizations to define higher-order roles. The
resulting state looks like:

![Grid Pike v2 Diagram](https://github.com/Cargill/grid-rfcs/blob/identity/images/grid-pike/pike2.svg)

The Agent has one or more organization-specific roles. The `has_permission()`
function determines the roles assigned to the agent and returns `true` if one of
the roles contains `permission_name` or `false` otherwise.

## Delegation of Roles Between Organizations

In order to support delegation between organizations, a role has fields called
`allowed_organizations` and `inherit_from`.

The `allowed_organizations` field is a list of organizations other than the
defining organization who can use the permissions granted in the role. This
allows organizations to delegate roles to each other.

The `inherit_from` field is a list containing references to other roles. If a
role has `inherit_from` defined, the permissions of the role must be a subset
of the union of all permissions defined by its `inherit_from`. An organization
may delegate a role to another organization containing a list of many
permissions. The delegatee organization may have separate functions that need
only a subset of the permissions in the delegated role. The `inherit_from` field
allows the delagatee organization to define roles that inherit from the
delegated role in order to be more granular about agent permissions.

To illustrate this concept, an example is provided below:

### Delegation Example

For this example, there are four organizations participating on a network:
Alpha, Beta, Gamma, and Delta. Alpha Company manages a set of tanks armed to the
teeth with ballistic conference t-shirts. They have hired Beta Company and
Gamma Company to drive the tanks.

The data model for the t-shirt tank may look something like this:

```yaml
- tank_id: 0001
  tank_owner: alpha
  turret_angle_degrees: 90
  t-shirts_remaining: 15
  is_driving: false
  decommissioned: false
```

The smart contract for operating tanks (named "tankops") would also define
several permissions:

  - `tankops::can-drive` allows agents to submit a transaction to update the `is_driving`
    boolean.
  - `tankops::can-turn-turret` allows agents to update the angle of the turret.
  - `tankops::can-fire` allows agents to fire the t-shirt cannon, decrementing the
    count of remaining t-shirts.
  - `tankops::can-decommission` allows agents to decommission the tank.

#### Alpha Compay

Alpha Company doesn't have any drivers, but they do have inspectors that are in
charge of decommissioning tanks which are no longer fit for service. To do this,
an admin of Alpha Company submits a Grid Pike v2 transaction to create the
Inspector role.

```yaml
- name: alpha.Inspector
  permissions:
    - tankops::can-decommission
  allowed_organizations: []
  inherit_from: []
  active: true
```

Alpha Company admins can then assign this role to agents to give them permission
to decommission t-shirt tanks. With the `allowed_organizations` field blank, the
smart contract will assume that only Alpha agents can decommission Alpha tanks.

Alpha Company also needs to define a role for drivers:

```yaml
- name: alpha.Drivers
  permissions:
    - tankops::can-drive
    - tankops::can-turn-turret
    - tankops::can-fire
  allowed_organizations:
    - beta
    - gamma
  inherit_from: []
  active: true
```

This role gives Beta Company and Gamma Company access to the permissions listed
in the alpha.Drivers role. For their agents to actually use these permissions,
they must redefine a role specific to their company which inherits permissions
from the alpha.Drivers role.

#### Beta Company

Beta Company is a rather small organization, and they expect their drivers to be
proficient in the full operation of a t-shirt tank. Therefore, it will define
a role with all of the necessary permissions:

```yaml
- name: beta.Drivers
  permissions:
    - tankops::can-drive
    - tankops::can-turn-turret
    - tankops::can-fire
  allowed_organizations: []
  inherit_from:
    - alpha.Drivers
  active: true
```

#### Gamma Company

The Gamma Company is a large organization with highly specialized agents. They
don't want to give all of their agents permission to do everything on a tank as
they are typically trained in only one of the three functions. To enforce this,
they define three different roles:

```yaml
- name: gamma.Navigator
  permissions:
    - tankops::can-drive
  allowed_organizations: []
  inherit_from:
    - alpha.Drivers
  active: true
```

```yaml
- name: gamma.Aimer
  permissions:
    - tankops::can-turn-turret
  allowed_organizations: []
  inherit_from:
    - alpha.Drivers
  active: true
```

```yaml
- name: gamma.Blaster
  permissions:
    - tankops::can-fire
  allowed_organizations: []
  inherit_from:
    - alpha.Drivers
  active: true
```

While most Gamma Company agents are specialists, they also have a few elite
t-shirt tank operators who are skilled in each function. The Gamma Company
defines a role which allows them to step in at a moment's notice if the agents
under their command are not performing their functions with sufficient finesse:

```yaml
- name: gamma.TankCommander
  permissions:
    - tankops::can-drive
    - tankops::can-turn-turret
    - tankops::can-fire
  allowed_organizations: []
  inherit_from:
    - alpha.Drivers
  active: true
```

#### Delta Company

After hearing of the great prowess of Beta Company tank drivers, Delta Company
(a competitor of Alpha Company) makes an offer to Beta Company, hiring them to
drive Delta tanks. The Beta Company has also been training their drivers to
inspect tanks and now offers a decommissioning service as well, which Delta
Company wishes to purchase. Delta Company defines the following role:

```yaml
- name: delta.TankOperator
  permissions:
    - tankops::can-drive
    - tankops::can-turn-turret
    - tankops::can-fire
    - tankops::can-decommission
  allowed_organizations:
    - beta
  inherit_from: []
  active: true
```

To allow its agents to start operating and inspecting the Delta tanks, the Beta
Company admin updates the `beta.Drivers` role to also include the
`tankops::can-decommission` permission and add `delta.TankOperator` as a super role.

```yaml
- name: beta.Drivers
  permissions:
    - tankops::can-drive
    - tankops::can-turn-turret
    - tankops::can-fire
    - tankops::can-decommission
  allowed_organizations: []
  inherit_from:
    - alpha.Drivers
    - delta.TankOperator
  active: true
```

This updated role allows Beta agents to drive, fire, and turn the turret on both
Alpha and Delta tanks. However, it only allows agents to decommission Delta
tanks, as the `alpha.Drivers` role does not include the
`tankops::can-decommission` permission.

Several weeks later, the Beta Company receives a phone call from Alpha Company
lawyers. They heard about the deal with Delta Company and cite a clause in their
contract which disallows drivers of Alpha tanks to also drive tanks from another
t-shirt tank management company. After the appropriate reprimands are dealt,
the Beta Company deactivates their previous `beta.Drivers` role and creates
two new roles, one for each tank company.

```yaml
- name: beta.Drivers
  permissions:
    - tankops::can-drive
    - tankops::can-turn-turret
    - tankops::can-fire
    - tankops::can-decommission
  allowed_organizations: []
  inherit_from:
    - alpha.Drivers
    - delta.TankOperator
  active: false
```

```yaml
- name: beta.AlphaDrivers
  permissions:
    - tankops::can-drive
    - tankops::can-turn-turret
    - tankops::can-fire
  allowed_organizations: []
  inherit_from:
    - alpha.Drivers
  active: true
```

```yaml
- name: beta.DeltaDrivers
  permissions:
    - tankops::can-drive
    - tankops::can-turn-turret
    - tankops::can-fire
    - tankops::can-decommission
  allowed_organizations: []
  inherit_from:
    - delta.TankOperator
  active: true
```

After this embarassing incident, the t-shirt tank consortium sets to work on a
new version of the tank operation smart contract to better handle exclusivity
agreements.

## Alternate IDs

In several Grid smart contracts, it is necessary to record an additional ID to
identify an organization in a certain context. For example, in Grid Product, the
organization must have the proper GS1 company prefix in order to interact with
products with correlating GTINs. In Grid Pike, this is handled through the use
of the metadata field.

In Grid Pike v2, we further define this functionality through a new concept
called "Alternate IDs". This is implemented through a field called
`alternate_ids` on the `Organization` message and a new message called
`AlternateIDIndexEntry`. The `alternate_ids` field allows users and smart
contracts to easily fetch a list of all of the alternate IDs associated with an
organization. The AlternateIDIndexEntry message serves as an index to fetch a
Grid Pike organization from an externally known ID, and also ensures that no
organization can have an alternate ID that is already assigned.

When an organization is created, we check the list of alternate IDs supplied in
the transaction to ensure that none of the alternate IDs are already associated
with an existing organization. If not, we create a new `AlternateIDIndexEntry`
in state for each alternate ID. If that organization is updated, we check the
list of alternate IDs supplied with the transaction against the present record.
Any missing alternate IDs will be deleted from the index (the
`AlternateIDIndexEntry` for that alternate ID is deleted) and any new IDs will
be checked and added.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## State

Agents, organizations, and roles are stored in state using protobuf message
format. These messages are defined as follows:

```protobuf
message Agent {
  string org_id = 1;
  string public_key = 2;
  bool active = 3;
  repeated string roles = 4;
  repeated KeyValueEntry metadata = 5;
}

message AgentList {
    repeated Agent agents = 1;
}

message Organization {
  string org_id = 1;
  string name = 2;
  repeated string locations = 3;
  repeated AlternateID alternate_ids = 4;
  repeated KeyValueEntry metadata = 5;
}

message OrganizationList {
  repeated Organization organizations = 1;
}

message Role {
  string org_id = 1;
  string name = 2;
  string description = 3;
  repeated string permissions = 4;
  repeated string allowed_organizations = 5;
  repeated string inherit_from = 6;
}

message RoleList {
  repeated Role roles = 1;
}

message AlternateIDIndexEntry {
  string id_type = 1;
  string id = 2;
  string grid_identity_id = 3;
}

// Helpers
message KeyValueEntry {
  string key = 1;
  string value = 2;
}

message AlternateID {
  string id_type = 1;
  string id = 2;
}
```

## Transactions

There is one `PikePayload` message which accounts for all Pike transactions.
This message has an enum to indicate if the transaction is a creation, update,
or delete action for each of the three core data types in Pike v2.

```protobuf
message PikePayload {
  enum Action {
    ACTION_UNSET = 0;
    CREATE_AGENT = 1;
    UPDATE_AGENT = 2;
    DELETE_AGENT = 3;
    CREATE_ORGANIZATION = 4;
    UPDATE_ORGANIZATION = 5;
    DELETE_ORGANIZATION = 6;
    CREATE_ROLE = 7;
    UPDATE_ROLE = 8;
    DELETE_ROLE = 9;
  }

  Action action = 1;
  CreateAgentAction create_agent = 2;
  UpdateAgentAction update_agent = 3;
  DeleteAgentAction delete_agent = 4;
  CreateOrganizationAction create_organization = 5;
  UpdateOrganizationAction update_organization = 6;
  DeleteOrganizationAction delete_organization = 7;
  CreateRoleAction create_role = 8;
  UpdateRoleAction update_role = 9;
  DeleteRoleAction delete_role = 10;
}

message CreateAgentAction {
  string org_id = 1;
  string public_key = 2;
  bool active = 3;
  repeated string roles = 4;
  repeated KeyValueEntry metadata = 5;
}

message UpdateAgentAction {
  string org_id = 1;
  string public_key = 2;
  bool active = 3;
  repeated string roles = 4;
  repeated KeyValueEntry metadata = 5;
}

message DeleteAgentAction {
  string org_id = 1;
  string public_key = 2;
}

message CreateOrganizationAction {
  string id = 1;
  string name = 2;
  repeated AlternateID alternate_ids = 4;
  repeated KeyValueEntry metadata = 5;
}

message UpdateOrganizationAction {
  string id = 1;
  string name = 2;
  repeated string locations = 3;
  repeated AlternateID alternate_ids = 4;
  repeated KeyValueEntry metadata = 5;
}

message DeleteOrganizationAction {
  string id = 1;
}

message CreateRoleAction {
  string org_id = 1;
  string name = 2;
  string description = 3;
  repeated string permissions = 4;
  repeated string allowed_organizations = 5;
  repeated string inherit_from = 6;
  bool active = 7;
}

message UpdateRoleAction {
  string org_id = 1;
  string name = 2;
  string description = 3;
  repeated string permissions = 4;
  repeated string allowed_organizations = 5;
  repeated string inherit_from = 6;
  bool active = 7;
}

message DeleteRoleAction {
   string org_id = 1;
   string name = 2;
}
```

## Naming Conventions

Roles in Grid Pike must adhere to a naming convention for easy identification
and uniformity. The permissions hard-coded into smart contracts should also
follow a naming convention. The naming conventions for these strings are as
follows:

Permissions: `<smart contract name>::<permission>`, for example the name of a
permission to update agents for the "identity" smart contract would be
`pike::can-update-agents`.

Roles: Roles should not have a "." character in their name. Clients (Such as UI,
REST API responses, and CLI tools) should use the following convention when
displaying a role name: `<Role.org_id>.<Role.name>`. This is to avoid any
confusion in the name of a role which may be reused across organizations. For
instance, the name of an "Admin" of the org with ID "alpha" should be displayed
as `alpha.Admin`. Note that the role name stored in state state is just "Admin".

## Addressing

In order to uniquely locate agents, organizations, and roles in the Merkle-Radix
state system, an address must be constructed which identifies the storage
location each the Grid Pike object.

All Grid addresses are prefixed by the 6-hex-character namespace prefix
“621dee”. Pike objects are further prefixed under the Grid namespace with
the reserved enumeration of “05”. The remaining 62 characters depend on the
object type as shown below:

### Agent State

The specific namespace prefix within Grid Pike for Agent state is
`621dee0500`, which is the general Grid Pike namespace `621dee05`
concatenated with 00. The remaining 60 characters are the first 60 characters of
the hash of the agent’s public key.

### Organization State

The specific namespace prefix within Grid Pike for Organization state is
`621dee0501`, which is the general Grid Pike namespace `621dee05`
concatenated with 01. The remaining 60 characters are the first 60 characters of
the hash of the organization's ID.

### Role State

The specific namespace prefix within Grid Pike for Role state is
`621dee0502`, which is the general Grid Pike namespace `621dee05`
concatenated with 02. The next 60 characters are the first 60 characters of
the hash of the role's org ID and the role's name, separated by a period:
`<Role.org_id>.<Role.name>`

### AlternateIDIndexEntry State

The specific namespace prefix within Grid Pike for the alternate ID index is
`621dee0503`, which is the general Grid Pike namespace `621dee05`
concatenated with 03. The next 60 characters are the first 60 characters of
the hash of the alternate ID. Note that this addess does not take the Grid
Pike org ID into account since the alternate ID must be unique.

## Permissions

Each smart contract that uses Grid Pike (including Grid Pike itself)
must define a set of permissions used for validation. The Grid Pike smart
contract defines 7 permissions, allowing agents to create, update, and delete
Grid Pike objects:

  - pike::can-create-agents
  - pike::can-update-agents
  - pike::can-delete-agents
  - pike::can-update-organization
  - pike::can-create-roles
  - pike::can-update-roles
  - pike::can-delete-roles

## Initialization

Due to the chicken-and-egg problem inherent in role-based permissioning systems,
Grid Pike v2 needs to define specific logic to initialize state for an
organization.

In Pike v1, the signing key of a transaction that creates a new organization
must not be established as an agent's public key already. The organization and
first agent associated with the new organization are created in one atomic step,
and the new agent is given the `admin` permission, which allows them to assign
permissions to other agents.

For Grid Pike v2, we will use a similar strategy. The transaction to create a
new organization will produce three objects: an agent, and organization, and an
"admin" role. An example of these objects is shown below:

### Initial Agent

```yaml
- org_id: alpha
  public_key: 000000000000000000000000000000000000000000000000000000000000000000
  active: true
  roles:
    - alpha.Admin
  metadata: []
```

### Initial Organization

```yaml
- org_id: alpha
  name: AlphaCompany
  locations: []
  alternate_ids: []
  metadata: []
```

### Initial Role

```yaml
- name: alpha.Admin
  permissions:
    - pike::can-create-agents
    - pike::can-update-agents
    - pike::can-delete-agents
    - pike::can-update-organization
    - pike::can-create-roles
    - pike::can-update-roles
    - pike::can-delete-roles
  allowed_organizations: []
  inherit_from: []
```

The <org_name>::Admin role has some special-case logic around it to avoid
several critical issues:
  - The Admin role can never be updated or deleted. This is the simplest way to
    ensure that we don't remove the ability to execute Grid Pike
    functionality for an organization.
  - The Admin role can only be added or removed from another agent with the
    Admin role. This way we can never remove the last admin.

# Drawbacks
[drawbacks]: #drawbacks

Since this RFC doesn't remove any existing functionality, there are few
drawbacks. However, existing smart contracts using Pike would need to be
reworked to use Grid Pike v2.

# Rationale and alternatives
[alternatives]: #alternatives

This design of Grid Pike v2 allows for powerful role-based permissioning and
delegation. It also maintains patterns established with Pike v1 while adding
some functionality that is specifically useful for Grid.

Another option for the implementation of this functionality would be to frame
it as a new smart contract. Grid Pike v2 implements a large enough change to
permissioning that it could reasonably be considered a new smart contract.
However, seeing as Pike v2 is essentially a superset including all Pike v1
functionality, this could be confusing. This would also make Pike a deprecated
feature instead of something we keep iterating upon.

The name "Pike" for the identity solution in Grid has generated some confusion
in the past. Other smart contracts in Grid have a descriptive name, where "Pike"
doesn't have any intrinsic meaning with regard to its functionality. We may want
to consider renaming it as part of this effort.

Earlier versions of Pike included a smart permissions capability which is not
discussed here. This capability may have to be redesigned to fit in with the
role system of Grid Pike v2 if it is needed in the future.

# Prior art
[prior-art]: #prior-art

[Grid Pike](https://grid.hyperledger.org/docs/0.1/pike_smart_contract_specification.html)
# Unresolved questions
[unresolved]: #unresolved-questions

- None
