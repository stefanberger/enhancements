<!--
**Note:** When your enhancement is complete, all of these comment blocks should be removed.

To get started with this template:

- [ ] **Create an issue in keylime/enhancements**
  When filing an enhancement tracking issue, please ensure to complete all
  fields in that template.  One of the fields asks for a link to the enhancement.  You
  can leave that blank until this enhancement is made a pull request, and then
  go back to the enhancement and add the link.
- [ ] **Make a copy of this template.**
 name it `NNNN-short-descriptive-title`, where `NNNN` is the issue number (with no
  leading-zero padding) assigned to your enhancement above.
- [ ] **Fill out this file as best you can.**
  At minimum, you should fill in the "Summary", and "Motivation" sections.
  These should be easy if you've preflighted the idea of the enhancement with the
  appropriate SIG(s).
- [ ] **Merge early and iterate.**
  Avoid getting hung up on specific details and instead aim to get the goals of
  the enhancement clarified and merged quickly.  The best way to do this is to just
  start with the high-level sections and fill out details incrementally in
  subsequent PRs.
-->
# enhancement-48: Incremental Attestation for IMA

<!--
This is the title of your enhancement.  Keep it short, simple, and descriptive.  A good
title can help communicate what the enhancement is and should be considered as part of
any review.
-->



<!--
A table of contents is helpful for quickly jumping to sections of a enhancement and for
highlighting any additional information provided beyond the standard enhancement
template.
-->

<!-- toc -->
- [Release Signoff Checklist](#release-signoff-checklist)
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Proposal](#proposal)
  - [User Stories (optional)](#user-stories-optional)
    - [Story 1](#story-1)
    - [Story 2](#story-2)
  - [Notes/Constraints/Caveats (optional)](#notesconstraintscaveats-optional)
  - [Risks and Mitigations](#risks-and-mitigations)
- [Design Details](#design-details)
  - [Test Plan](#test-plan)
  - [Upgrade / Downgrade Strategy](#upgrade--downgrade-strategy)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
- [Infrastructure Needed (optional)](#infrastructure-needed-optional)
<!-- /toc -->

## Release Signoff Checklist

<!--
**ACTION REQUIRED:** In order to merge code into a release, there must be an
issue in [keylime/enhancements] referencing this enhancement and targeting a release**.

For enhancements that make changes to code or processes/procedures in core
Keylime i.e., [keylime/keylime], we require the following Release
Signoff checklist to be completed.

Check these off as they are completed for the Release Team to track. These
checklist items _must_ be updated for the enhancement to be released.
-->

- [ ] Enhancement issue in release milestone, which links to pull request in [keylime/enhancements]
- [ ] Core members have approved the issue with the label `implementable`
- [ ] Design details are appropriately documented
- [ ] Test plan is in place
- [ ] User-facing documentation has been created in [keylime/keylime-docs]

<!--
**Note:** This checklist is iterative and should be reviewed and updated every time this enhancement is being considered for a milestone.
-->

## Summary

<!--
This section is incredibly important for producing high quality user-focused
documentation such as release notes or a development roadmap.  It should be
possible to collect this information before implementation begins in order to
avoid requiring implementers to split their attention between writing release
notes and implementing the feature itself. Reviewers
should help to ensure that the tone and content of the `Summary` section is
useful for a wide audience.

A good summary is probably at least a paragraph in length.
-->

The IMA event logs can grow to multiple thousands of log entries with each
entry being ~100 bytes or even larger when IMA appraisal is being used on
the system. With keylime doing an attestation every second, Mbytes of
network bandwith would be consumed for the transfer of the log and many
CPU cycles be comsumed by the verifier to verify each log. This enhancement
attempts to address this issue by introducing incemental IMA attestation
where the verifier requests only the end of the IMA log using the next entry
number to receive from the IMA log following the previous attestation step.
The verifier would then resume the verification of the log where it left off
the previous time and maintain the state of IMA's PCR 10.

## Motivation

<!--
This section is for explicitly listing the motivation, goals and non-goals of
this enhancement.  Describe why the change is important and the benefits to users.
-->

The motivation of this enhancement is the realization that bandwitdh
consumption as well as the CPU cycle consumption can be reduced and the
scalability of the keylime verifier be increased when only the differences
in the IMA logs from the last attestation verification are transferred.

### Goals

<!--
List the specific goals of the enhancement.  What is it trying to achieve?  How will we
know that this has succeeded?
-->

The goal of this enhancement is to reduce network bandwidth consumption as well as
the verifier's CPU cycle consumption.

### Non-Goals

<!--
What is out of scope for this enhancement?  Listing non-goals helps to focus discussion
and make progress.
-->

## Proposal

<!--
This is where we get down to the specifics of what the proposal actually is.
This should have enough detail that reviewers can understand exactly what
you're proposing, but should not include things like API designs or
implementation.  The "Design Details" section below is for the real
nitty-gritty.
-->

The verifier will need to maintain per-agent state that reflects the state
of the IMA PCR from the last attestation along with the next-to-request
IMA log entry. The verifier needs to be extended to return the
IMA log starting at a specific entry in the IMA log. There are no changes
required to the quote.

This proposal assumes that affinity of an agent with a verifier is
maintained throughout the attestion. This allows a specific verifier
to maintain the state of and agent in memory (IMA PCR and next-to-request
IMA log entry). The Keylime verifier protocol seems to support this
assumption since a verifier initates the protocol for attestation by
talking to an agent.

To maintain backwards compatibility with older verifiers, the updated
agent will return the entire IMA log when the verifier's request does
not contain the number of the next IMA entry to return, which indicates
and 'old' verifier.

Similiary, to maintain backwards compatibility with older agents, a updated
verifier has to be able to deal with the situation where the agent did
not respond with the partial log but the entire IMA log. This situation
is indicated by the agent not returning the number of the first IMA log
entry it returned, thus the verifier will assume the log starts at
the first entry.

From the above it can be seen that the benefits of reduction of network
bandwith consumption and CPU cycle consumption are only going to seen
when updated verifiers and updated agents are being used.

Upon restart of the verifier, it will request the IMA log from the
first entry (entire log) when resuming communication with a particular
agent. Therefore, startup of the IMA verification will be as costly
as it is before this enhancement.

The per-agent state of the PCRs and the next-to-request IMA log entry
will be maintained in the verifier's memory and accessible using the
unique ID of an agent (indexed by agent Id).

### User Stories (optional)

<!--
Detail the things that people will be able to do if this enhancement is implemented.
Include as much detail as possible so that people can understand the "how" of
the system.  The goal here is to make this feel real for users without getting
bogged down.
-->

#### Story 1

#### Story 2

### Notes/Constraints/Caveats (optional)

<!--
What are the caveats to the proposal?
What are some important details that didn't come across above.
Go in to as much detail as necessary here.
This might be a good place to talk about core concepts and how they relate.
-->

### Risks and Mitigations

<!--
What are the risks of this proposal and how do we mitigate.  Think broadly.
For example, consider both security and how this will impact the larger
enhancement ecosystem.

How will security be reviewed and by whom?
-->

## Design Details

<!--
This section should contain enough information that the specifics of your
change are understandable.  This may include API specs (though not always
required) or even code snippets.  If there's any ambiguity about HOW your
proposal will be implemented, this is the place to discuss them.
-->

### Test Plan

<!--
**Note:** *Not required until targeted at a release.*

Consider the following in developing a test plan for this enhancement:
- Will there be e2e and integration tests, in addition to unit tests?
- How will it be tested in isolation vs with other components?

No need to outline all of the test cases, just the general strategy.  Anything
that would count as tricky in the implementation and anything particularly
challenging to test should be called out.

All code is expected to have adequate tests (eventually with coverage
expectations).
-->

The PR implementing this proposal will add
- unit tests for newly added functions
- integration tests fro testing incremental attestation

### Upgrade / Downgrade Strategy

<!--
If applicable, how will the component be upgraded and downgraded? Make sure
this is in the test plan.

Consider the following in developing an upgrade/downgrade strategy for this enhancement
-->

### Dependencie requirements

<!--
If your new change requires new dependencies, please outline and demonstrate that your selected dependency 
is well maintained and packaged in Keylime's supported Operating Systems (currently Debian Stable
and as of time writing Fedora 32/33). 

During code implementation you will also be expected to add the package to CI , the keylime ansible role and 
keylimes main installer (`keylime/installers.sh`).

If the package is not available in the supported Operated systems, the PR will not be merged into master. 

Adding the package in `requirements.txt` is not sufficent for master which is where we tag releases from. 

You may however be able to work within an experimental branch until a package is made available. If this is
the case, please outline it in this enhancement.

-->

## Drawbacks

<!--
Why should this enhancement _not_ be implemented?
-->

## Alternatives

<!--
What other approaches did you consider and why did you rule them out?  These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.
-->

None known.

## Infrastructure Needed (optional)

<!--
Use this section if you need things infrastructure related specific to your enhancement.  Examples include a
new subproject, repos requested, github webhook, changes to CI (travis).
-->
