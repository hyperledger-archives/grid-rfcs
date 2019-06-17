- Feature Name: gs1_subteam
- Start Date: 2019-05-04
- RFC PR:
- Hyperledger Grid Issue:

# Summary
[summary]: #summary

Creates a new GS1 subteam to help shepherd RFCs related to the GS1 standards
through the RFC process. Topics appropriate for this subteam include any
related to the GS1 standards. This team is reponsible for ensuring that Grid's
GS1 support is consistent with the GS1 standards and the intent behind the
those standards.

# Motivation
[motivation]: #motivation

[Grid RFC 0008](https://github.com/hyperledger/grid-rfcs/blob/master/text/0008-grid-governance.md)
introduced an explicit governance model for Hyperledger Grid, including
a framework for creation of subteams. With the introduction of GS1-related
features into Grid, it is desirable to have a subteam assembled to provide
guidance and input on GS1-related topics and to serve as an additional voting
entity for GS1-related RFCs.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

The GS1 subteam is responsible for discussion, review, and decision making on
topics related to GS1 standards support, including RFCs.

When overlap occurs between the GS1 subteam and Core subteam, such as when
voting on and RFC which is GS1-related and highly technical, members of both
subteams should make the decision as a larger unified subteam.

As with the other subteams detailed in the governance RFC, the specific
decisions the GS1 subteam will be tasked with include:

- Shepherding applicable RFCs
- Coming to consensus on whether to accept or reject RFCs
- Setting policy on what sort of changes require an RFC
- Coming to consensus on new members for the subteam
- Reviewing PRs

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

In order to achieve the desired level of expertise in the GS1 subject matter
while keeping it grounded within the Grid architecture, it is important that
the GS1 subteam contain a mix of both GS1 experts and Grid maintainers.

Therefore, the initial membership of the GS1 subteam is made up of the
following members:

- @davececchi, Dave Cecchi, team lead
- TBD

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

The initial team members should be determined before this RFC goes to FCP.
