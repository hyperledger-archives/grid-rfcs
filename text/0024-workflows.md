- Feature Name: Workflow
- Start Date: 2020-01-25
- RFC PR: 24
- Hyperledger Grid Issue: (leave this empty)

# Summary
[summary]: #summary

Grid Workflow provides a framework for modeling business workflows as state
transitions within smart contracts. Workflows are defined by their states,
constraints, and permissions.

# Motivation
[motivation]: #motivation

The rules governing what actions different entities are allowed to perform and
when they are allowed to perform them within smart contracts can be complex and
nuanced. The Grid Workflow module offers a framework for encapsulating that
complexity and allows for those rules to become decoupled from the smart
contract logic. Long-term, we envision runtime-defined workflows, thus offering
significant process flexibility within Grid. Grid Workflow may also make
implementing and maintaining smart contracts easier by clearly separating the
business flow conceptually.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

Smart contracts use Grid Workflows by creating a `Workflow` object. It creates
the `Workflow` object either explicitly by hard-coding the workflow definition
in code or by loading the workflow definition from state (a future smart
contract may be provided to make storing workflows a generalized Grid
capability, but it is out-of-scope for this RFC).

The `Workflow` contains a list of `SubWorkflow` objects representing different
smaller workflows that make up the overall workflow. Each `SubWorkflow` contains
a `WorkflowState` that describes the possible state transitions the
`SubWorkflow` can make, the constraints that need to be met to make said
transitions, and a list of `PermissionAlias` objects that represent the
permissions that are required by the acting entity to initiate a transition.
Finally, a `PermissionAlias` object is an object that represents a list of
permissions and the transitions those permissions are allowed to make. These
permissions are initially defined in the Pike Smart Contract. For example, a
permission alias called `po.seller`, meant to represent the seller listed on a
purchase order, is an alias for the permissions to create, and update a purchase
order, and can move the purchase order into the confirmed state.

```
let alias = PermissionAlias {
   name: “po.seller”.to_string(),
   permissions: vec![“update_po”.to_string(), “update_version”.to_string()],
   transitions: vec![“confirmed”.to_string()]
}
```

Below is an example of how to instantiate a simple purchase order workflow.

```
fn get_po_workflow() -> Workflow {
    let mut permission = PermissionAlias::new("po.seller");
    permission.add_permission("po.create");
    permission.add_permission("po.update");
    permission.add_transition("confirm");

    let state = WorkflowStateBuilder::new("issued")
        .add_constraint("active=None")
        .add_transition("issued")
        .add_transition("confirm")
        .add_permission_alias(permission)
        .build();

    let subworkflow = SubWorkflowBuilder::new("po")
        .add_state(state)
        .add_starting_state("issued")
        .add_starting_state("proposed")
        .build();

    let workflow = Workflow::new(vec![subworkflow]);
}
```

Below is an example outlining this process for a hypothetical smart contract designed to process purchase orders.

```
fn apply(
    &self,
    request: &TpProcessRequest,
    context: &mut dyn TransactionContext,
) -> Result<(), ApplyError> {
   let current_state = context.get_state_entry(PO_STATE_ADDRESS);
   let desired_state = request.get_state();
   let workflow = get_po_workflow();
   let subworkflow = workflow.subworkflow("po");
   let state = subworkflow.state(current_state);
   let pike_perms = context.get_state_entry(PIKE_ADDRESS);
   let permissions = state.expand_permissions(pike_perms);

   let can_transition = state.can_transition(
       desired_state, pike_permissions);
   if (!can_transition) {
       InvalidTransaction
   }
}
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## Workflow

A workflow consists of a list of sub workflows

```
pub struct Workflow {
    subworkflow: Vec<SubWorkflow>,
}

impl Workflow {
    pub fn new(subworkflow: Vec<SubWorkflow>) -> Self {
        Self { subworkflow }
    }

    pub fn subworkflow(&self, name: &str) -> Option<SubWorkflow> {
        for sub_wf in &self.subworkflow {
            if sub_wf.name() == name {
                return Some(sub_wf.clone());
            }
        }

        None
    }
}
```

## SubWorkflow

SubWorkflow has a name, a list of WorkflowState and a list of starting states

```
#[derive(Clone)]
pub struct SubWorkflow {
    name: String,
    states: Vec<WorkflowState>,
    starting_states: Vec<String>,
}

impl SubWorkflow {
    pub fn name(&self) -> &str {
        &self.name
    }

    pub fn state(&self, name: &str) -> Option<WorkflowState> {
        for state in &self.states {
            if state.name() == name {
                return Some(state.clone());
            }
        }

        None
    }

    pub fn starting_states(&self) -> &[String] {
        &self.starting_states
    }
}
```

## WorkflowState

A WorkflowState has a name, a list of constraints, a list of permission
aliases, and a list of transitions that can be made from that state. It has two
methods `can_transition` and `expand_permission`. `can_transition` returns true
if an entity can execute a transition to a given state if it has a given set of
Pike permissions. `expand_permissions` returns a list of all the permissions
that are stored under a given `PermissionAlias`
 
```
#[derive(Clone)]
pub struct WorkflowState {
    name: String,
    constraints: Vec<String>,
    permission_aliases: Vec<PermissionAlias>,
    transitions: Vec<String>,
}

impl WorkflowState {
    pub fn can_transition(&self, new_state: String, pike_permissions: Vec<String>) -> bool {
        if !self.transitions.contains(&new_state) {
            return false;
        }

        for perm in pike_permissions {
            for alias in &self.permission_aliases {
                if alias.name() == perm && alias.transitions.contains(&new_state) {
                    return true;
                }
            }
        }

        false
    }

    pub fn expand_permissions(&self, names: &[String]) -> Vec<String> {
        let mut perms = Vec::new();

        for name in names {
            for alias in &self.permission_aliases {
                if alias.name() == name {
                    perms.append(&mut alias.permissions().to_vec());
                }
            }
        }

        perms
    }
}
```

## PermissionAlias

`PermissionAlias` is an alias that houses multiple permissions. It has a name, a
list of actions that live under the alias, and a list of transitions the alias
can perform.

```
#[derive(Clone, Default)]
pub struct PermissionAlias {
    name: String,
    permissions: Vec<String>,
    transitions: Vec<String>,
}

impl PermissionAlias {
    pub fn new(name: &str) -> Self {
        Self {
            name: name.to_string(),
            permissions: vec![],
            transitions: vec![],
        }
    }

    pub fn add_permission(&mut self, permission: &str) {
        self.permissions.push(permission.to_string());
    }

    pub fn add_transition(&mut self, transition: &str) {
        self.transitions.push(transition.to_string());
    }

    pub fn name(&self) -> &str {
        &self.name
    }

    pub fn permissions(&self) -> &[String] {
        &self.permissions
    }

    pub fn transitions(&self) -> &[String] {
        &self.transitions
    }
}

```

Below is an example of a permission alias with the name `po.seller` that has
permission to perform two actions, update a purchase order and update its
version number, and the ability to perform one transition, move the purchase
order to it’s confirmed state.

```
let alias = PermissionAlias {
   name: “po.seller”.to_string(),
   permissions: vec![“update_po”.to_string(), “update_version”.to_string()],
   transitions: vec![“confirmed”.to_string()]
}
```

# Drawbacks
[drawbacks]: #drawbacks

While the implementation of Workflow is intended to be fairly lightweight, there
may be smart contracts whose permissioning rules are simple enough to be
reasonably represented directly in the smart contract. In such cases, Workflow
may introduce unnecessary complexity to those smart contracts.

# Rationale and alternatives
[alternatives]: #alternatives

The goal of Grid Workflow is to make permissioning and state code for smart
contracts easier to write and maintain by offering a flexible framework for
implementing it. The alternative to this would be what is currently done for
smart contracts, which is to have permissioning code be written ad hoc and be
unique and custom to each smart contract that is written.

# Unresolved questions
[unresolved]: #unresolved-questions

- Should this workflow design support rules automatic approval? Rules that
determine who will be assigned to approve transactions? Rules for escalating a
workflow that has been waiting for approval for a long time?
