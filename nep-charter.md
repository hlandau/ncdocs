Namecoin Extension Proposals
----------------------------

The Namecoin project may from time to time choose to publish formal
specifications detailing the operation of the Namecoin system or other
technologies relating to it. These specifications enable interoperability by
providing guidance for new implementations.

Each Namecoin Extension Proposal (NEP) shall be assigned an ordinal number.

The process is inspired by the IETF/RFC Editor process for publishing RFCs. The
process shall be as follows:

  - The Namecoin project creates an official NEP repository for storing all NEPs.

  - A person publishes a draft specification in a repository on GitHub.
    They should not assign the specification a number, as this is done only
    once the specification becomes a NEP by approval.

    The specification must have a title and list the authors.

    It should specify the licence on the document. The licence should be very
    liberal; for example, it should be Creative Commons licenced or released
    into the public domain.

    The use of the Markdown format is suggested but other formats will be
    accepted.

  - The proposer notifies the Namecoin project, and other interested parties,
    by any means. The specification is thereby brought to the attention
    of the Namecoin community and discussed. The primary avenues for discussion
    will be the Namecoin forum and the #namecoin IRC channel.

    A forum for the discussion of draft specifications will be created on the
    Namecoin forum. A thread should be created in this forum only when a
    specification has been published. It should incorporate a link to that
    specification on GitHub and briefly introduce the specification.

    All interested parties are then welcome to comment on the specification.

  - The proposer considers all feedback and revises their specification.
    They publish a notice to this effect in the thread and on IRC.
    The process continues until all issues with the specification have been
    worked out.

  - A decision is made by the Namecoin project members to approve or reject
    the specification. [TBD: consensus criteria]. If the decision is approved,
    it is allocated a NEP number. Usually this number shall be one greater than
    the previous NEP. The specification is copied into the NEP repository,
    renamed to have a filename of NEP-####.ext where #### is the 0-padded NEP
    number and .ext is the appropriate file extension.

    At this time the document should be edited to reflect the fact that it is a
    NEP, and to make any necessary editorial changes. Technically substantive
    changes must not be made.

    The index of NEPs is updated to list the new NEP and its status.
    The initial status is Current.

  - Once a specification has become a NEP, that NEP cannot be changed. Changes
    can be made only by creating a new NEP by the process described above.

A NEP has one of the following statuses:

  - Current: means that the NEP has not been superceded or deprecated.
  
  - Superceded: means that the NEP has been superceded by a new NEP.
    The NEP superceding it should be listed.

  - Deprecated: means that the NEP has been deprecated without being
    replaced with a new NEP.

NEPs can be classified into the following categories:

  - Standards Track: A Standards Track NEP describes any change that affects
    most or all Namecoin implementations, such as a change to the network
    protocol, a change in block or transaction validity rules, or any change or
    addition that affects the interoperability of applications using Namecoin.

  - Informational: An Informational NEP describes a Namecoin design issue, or
    provides general guielines or information to the Namecoin community, but
    does not propose a new feature. Informational NEPs do not necessarily
    represent a Namecoin community consensus or recommendation, so users and
    implementors are free to ignore Informational NEPs.

  - Process: A Process NEP describes a process surrounding Namecoin, or
    proposes a change to (or an event in) a process. Process NEPs are
    like Standards Track NEPs but apply to areas other than the Namecoin
    protocol itself. They may propose an implementation, but not to
    Namecoin's codebase; they often require community consensus; unlike
    Informational NEPs, they are more than recommendations, and users
    are typically not free to ignore them. Examples include procedures,
    guidelines, changes to the decision-making process, and changes to
    the tools or environment used in Namecoin development. Any meta-NEP
    is also considered a process NEP.

When constructing a specification, you should generally inspire the structure
of your document by that of RFCs. Invoke RFC 2119 when appropriate.

This document is a proposed Process specification for the NEP process.
If approved, it will become the first NEP.

