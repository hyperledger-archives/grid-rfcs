# Hyperledger Grid

Hyperledger Grid [has moved](https://github.com/hyperledger/toc/issues/82) to [End of life status](https://toc.hyperledger.org/governing-documents/project-lifecycle.html#end-of-life).

# Hyperledger Grid RFCs
[Hyperledger Grid RFCs]: #grid-rfcs

The grid-rfcs repository contains feature proposal RFCs (requests for comments)
for [Hyperledger Grid](https://grid.hyperledger.org).

This README file describes the process for submitting, reviewing, and approving
an RFC for the [grid](https://github.com/hyperledger/grid) repository. Use the
same process to propose changes to the RFC process in
[this repository](https://github.com/hyperledger/grid-rfcs).

## Table of Contents
[Table of Contents]: #table-of-contents

  - [Opening](#grid-rfcs)
  - [Introduction]
  - [When you need to follow this process]
  - [Before creating an RFC]
  - [What the process is]
  - [The RFC life cycle]
  - [Reviewing RFCs]
  - [Implementing an RFC]
  - [Help! This is all too informal!]
  - [License]


## Introduction
[Introduction]: #introduction

Many changes, including bug fixes and documentation improvements can be
implemented and reviewed via the normal GitHub pull request workflow.

Some changes, though, are "substantial"; we ask that these be put through a
bit of a design process and produce a consensus among the Hyperledger Grid
community and the Grid root team and relevant
[subteams](#subteam-specific-guidelines).

The "RFC" (request for comments) process is intended to provide a consistent
and controlled path for major changes to enter Hyperledger Grid, so that all
stakeholders can be confident about how Hyperledger Grid is evolving.

This process is intended to be substantially similar to the Rust RFC process,
customized as necessary for use with Hyperledger Grid. The `README.md` and
`0000-template.md` files were initially forked from the [Rust
RFCs](https://github.com/rust-lang/rfcs) repository.


## When you need to follow this process
[When you need to follow this process]: #when-you-need-to-follow-this-process

You need to follow this process if you intend to make "substantial" changes to
Hyperledger Grid or any of its sub-components, including (but not limited to)
[hyperledger/grid](https://github.com/hyperledger/grid) and the
RFC process itself. What constitutes a "substantial" change is evolving based
on community norms and varies depending
on what part of the ecosystem you are proposing to change, but may include the
following:

  - Architectural changes
  - Substantial changes to component interfaces
  - New features
  - Backward-incompatible changes
  - Changes that affect the security of communications or administration

Some changes do not require an RFC:

  - Rephrasing, reorganizing, refactoring, or otherwise "changing shape that
    does not change meaning".
  - Additions that strictly improve objective, numerical quality criteria
    (warning removal, speedup, better platform coverage, more parallelism, trap
    more errors, etc.)

If you submit a pull request to implement a new feature without going through
the RFC process, it may be closed with a polite request to submit an RFC first.


### Subteam-specific guidelines
[Subteam-specific guidelines]: #subteam-specific-guidelines

The Grid root team can choose to handle an RFC or delegate it to a subteam.
To propose creating a new subteam, create an RFC for the
[grid-rfcs](https://github.com/hyperledger/grid-rfcs) repository. The Grid root
team must approve the RFC. After the RFC is approved and merged, a
`subteam/{name}.md` file will list the subteam members and a link to the subteam
will be added to this section.

## Before creating an RFC
[Before creating an RFC]: #before-creating-an-rfc

A hastily proposed RFC can hurt its chances of acceptance. Low quality
proposals, proposals for previously-rejected changes, and those that don't fit
into the near-term roadmap may be quickly rejected, which can be demotivating
for the unprepared contributor. Laying some groundwork ahead of the RFC can
make the process smoother.

Although there is no single way to prepare for submitting an RFC, it is
generally a good idea to pursue feedback from other project developers
beforehand, to ascertain that the RFC may be desirable; having a consistent
impact on the project requires concerted effort toward consensus-building.

The most common preparations for writing and submitting an RFC include talking
the idea over on the Grid developer discussion forum, [#grid], and proposing
ideas to the [Hyperledger Grid mailing list].

As a rule of thumb, receiving encouraging feedback from long-standing project
developers, particularly maintainers or members of the relevant team, is a
good indication that the RFC is worth pursuing.


## What the process is
[What the process is]: #what-the-process-is

In short, to get a major feature added to Hyperledger Grid, you must start by
adding an RFC in markdown format as a pull request for this repository. The RFC
is discussed and changed as necessary. When approved, the RFC is merged into the
RFC repository. At that point, the RFC is "active" and may be implemented with
the goal of eventually adding the new feature to Hyperledger Grid.

  - Fork the [grid-rfcs repository](https://github.com/hyperledger/grid-rfcs).
  - Copy `0000-template.md` to `text/0000-my-feature.md` (where "my-feature" is
    descriptive). Don't assign an RFC number yet (leave `0000` in the name).
  - If the RFC has supporting images or diagram files, also create a folder
    called `0000-my-feature` and put them there.
  - Fill in the RFC. Put care into the details: RFCs that do not present
    convincing motivation, do not demonstrate understanding of the impact
    of the design, or are disingenuous about the drawbacks or alternatives tend
    to be poorly received.
  - Submit a pull request. As a pull request, the RFC will receive design
    feedback from the larger community, and the author should be prepared to
    revise it in response.
  - Build consensus and integrate feedback. RFCs that have broad support are
    much more likely to make progress than those that don't receive any
    comments. Feel free to reach out to the RFC assignee in particular to get
    help identifying stakeholders and obstacles.
  - The root team or relevant subteam will discuss the RFC pull request, as much
    as possible, in the comment thread of the pull request itself. Offline
    discussion will be summarized on the pull request comment thread.
  - RFCs rarely go through this process unchanged, especially as alternatives
    and drawbacks are shown. You can make edits, big and small, to the RFC to
    clarify or change the design, but make changes as new commits to the pull
    request and leave a comment on the pull request explaining your changes.
    Specifically, do not squash or rebase commits after they are visible on the
    pull request.
  - At some point, a member of the team will propose a "motion
    for final comment period" (FCP), along with a *disposition* for the RFC
    (merge, close, or postpone).
    - This step is taken when enough of the tradeoffs have been discussed that
    the team is in a position to make a decision. That does not require
    consensus amongst all participants in the RFC thread (which is usually
    impossible). However, the argument supporting the disposition on the RFC
    needs to have already been clearly articulated, and there should not be a
    strong consensus *against* that position outside of the team. Team
    members use their best judgment in taking this step, and the FCP itself
    ensures there is ample time and notification for stakeholders to push back
    if it is made prematurely.
    - For RFCs with lengthy discussion, the motion to FCP is usually preceded by
      a *summary comment* trying to lay out the current state of the discussion
      and major trade-offs/points of disagreement.
    - Before actually entering FCP, **all** members of the team must sign off;
    this is often the point at which some team members first review the RFC
    in full depth.
  - The FCP lasts one week, or seven calendar days. It is also advertised
    widely (e.g., in the [Hyperledger Grid mailing list]). This
    way, all stakeholders have a chance to lodge any final objections before
    a decision is reached.
  - In most cases, the FCP period is quiet, and the RFC is either merged or
    closed. However, sometimes substantial new arguments or ideas are raised,
    the FCP is canceled, and the RFC goes back into development mode.

## The RFC life cycle
[The RFC life cycle]: #the-rfc-life-cycle

Once an RFC becomes "active", then authors may implement it and submit the
change as a pull request to the appropriate Hyperledger Grid repo. Being
"active" is not a rubber stamp and, in particular, still does not mean the
change will ultimately be merged; it does mean that, in principle, all the major
stakeholders have agreed to the change and are amenable to merging it.

Furthermore, the fact that a given RFC has been accepted and is "active"
implies nothing about what priority is assigned to its implementation, nor does
it imply anything about whether a Hyperledger Grid developer has been assigned
the task of implementing the feature. While it is not *necessary* that the
author of the RFC also write the implementation, it is by far the most effective
way to see an RFC through to completion: authors should not expect that other
project developers will take on responsibility for implementing their accepted
feature.

Modifications to "active" RFCs can be done in follow-up pull requests. We
strive to write each RFC in a manner that it will reflect the final design of
the feature; but the nature of the process means that we cannot expect every
merged RFC to actually reflect what the end result will be at the time of the
next major release.

In general, once accepted, an RFC should not be substantially changed. Only very
minor changes should be submitted as amendments. More substantial changes
should be new RFCs, with a note added to the original RFC. Exactly what counts
as a "very minor change" is up to the root team or subteam to decide; check
[Subteam-specific guidelines] for more details.


## Reviewing RFCs
[Reviewing RFCs]: #reviewing-rfcs

While the RFC pull request is up, the root team or subteam may schedule meetings
with the author and/or relevant stakeholders to discuss the issues in greater
detail; in some cases, the topic may be discussed at a team meeting. In either
case, a summary from the meeting will be posted back to the RFC pull request.

The team makes final decisions about RFCs after the benefits and drawbacks
are well understood. These decisions can be made at any time, but the team
will regularly issue decisions. When a decision is made, the RFC pull request
will either be merged or closed. In either case, if the reasoning is not clear
from the discussion in thread, the team will add a comment describing the
rationale for the decision.


## Implementing an RFC
[Implementing an RFC]: #implementing-an-rfc

Some accepted RFCs represent vital changes that need to be implemented right
away. Other accepted RFCs can represent changes that can wait until some
arbitrary developer feels like doing the work. Every accepted RFC has an
associated issue tracking its implementation in the Hyperledger Grid
[JIRA issue tracker]; thus, that associated issue can be assigned a priority
via the triage process that the team uses for all issues related to
Hyperledger Grid.

The author of an RFC is not obligated to implement it. Of course, the RFC
author (like any other developer) is welcome to post an implementation for
review after the RFC has been accepted.

If you are interested in working on the implementation for an "active" RFC, but
cannot determine if someone else is already working on it, feel free to ask
(for example, by leaving a comment on the associated issue).


## Help! This is all too informal!
[Help! This is all too informal!]: #help-this-is-all-too-informal

The process is intended to be as lightweight as reasonable for the present
circumstances. As usual, we are trying to let the process be driven by
consensus and community norms, not impose more structure than necessary.

[#grid]: https://chat.hyperledger.org/channel/grid
[Hyperledger Grid mailing list]: https://lists.hyperledger.org/g/grid
[JIRA issue tracker]: https://jira.hyperledger.org/projects/GRID/issues
[RFC repository]: https://github.com/hyperledger/grid-rfcs


## License
[License]: #license

This repository is licensed under Apache License, Version 2.0
(<http://www.apache.org/licenses/LICENSE-2.0>).


### Contributions

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
licensed as above, without any additional terms or conditions.
