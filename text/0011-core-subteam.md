- Feature Name: core_subteam
- Start Date: 2019-05-04
- RFC PR: [hyperledger/grid-rfcs#11](https://github.com/hyperledger/grid-rfcs/pull/11)
- Hyperledger Grid Issue:

# Summary
[summary]: #summary

Create a new Grid Core subteam to shepherd technical RFCs through the RFC
process, and make technical governance decisions; technical topics include
those related to the design and architecture of Grid. This team is responsible
for ensuring long-term architectural consistency with the existing Grid code
and overall project direction adopted by the Grid Root team.

The Grid Core subteam shall be the default subteam for technical decisions in
absence of another appropriate subteam.

# Motivation
[motivation]: #motivation

[Grid RFC 0008](https://github.com/hyperledger/grid-rfcs/blob/master/text/0008-grid-governance.md)
introduced an explicit governance model for Hyperledger Grid, including
a framework for creation of subteams.  The RFC process generates many
technical RFCs (some are currently pending) which are appropriate for the
Grid Core subteam to shepherd and approve.

In order to ensure that we always have a functional technical decision making
process, the Grid Core subteam will serve as the default subteam for technical
decisions.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

The Grid Core subteam is Responsible for the design, architecture, performance,
stability, and security of the core Grid platform.

The Grid Core subteam exists for the purpose of technical review and decision
making, in particular those which are in the form of an RFC.

Non-technical decisions should be left to the Grid Root subteam or other
subteam as appropriate.  For example, as new components are added to Grid, the
Grid Core subteam may act in a capacity of technical review of the potentially
incoming components. However, the decision on whether to add the component may
be left with the Grid Root subteam as a wider project decision. Thus it is
possible that RFCs may require approval from both the Grid Root and Grid Core
subteams, if that is deemed appropriate.

As with the other subteams detailed in the governance RFC, the specific
decisions the Grid Core subteam will be tasked with include:

- Shepherding applicable RFCs
- Coming to consensus on whether to accept or reject RFCs
- Setting policy on what sort of changes require an RFC
- Coming to consensus on new members for the subteam
- Reviewing PRs

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

In order to achieve the long-term architectural consistency, membership in the
Grid Core team should consist of individuals who have a fairly long history
with the project. For the subteam to be able to properly evaluate RFCs or
decisions with the proper context, the subteam membership should consist of
members who have a working knowledge of the existing code and/or overall
existing architecture.

Therefore, the initial membership of the Grid Core subteam is made up of Grid
Root team members who are Grid maintainers and/or are have a history of
providing input into Grid technical issues:

- @vaporos, Shawn Amundson, team lead
- @peterschwarz, Peter Schwarz
- @jsmitchell, James Mitchell
- @dplumb94, Darian Plumb
- @desktophero, Jason Walker
- @agunde406, Andrea Gunderson
- @adeebahmed, Adeeb Ahmed

While members of this subteam have decision making capability, it is critical
to the health of the project that the wider community is involved in
discussions and that the subteam members represent the wider community to
the extent possible.

# Drawbacks
[drawbacks]: #drawbacks

None.

# Rationale and alternatives
[alternatives]: #alternatives

No alternatives have been discussed.

# Prior art
[prior-art]: #prior-art

This is based on the governance model laid out by
[Grid RFC 0008](https://github.com/hyperledger/grid-rfcs/blob/master/text/0008-grid-governance.md),
[Sawtooth RFC 0006](https://github.com/hyperledger/sawtooth-rfcs/blob/master/text/0006-sawtooth-governance.md),
[Sawtooth RFC 0024](https://github.com/hyperledger/sawtooth-rfcs/blob/master/text/0024-core-subteam.md),
and
[Rust team](https://github.com/rust-lang/rfcs/blob/master/text/1683-docs-team.md).

# Unresolved questions
[unresolved]: #unresolved-questions

None.
