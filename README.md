# MIPs: MEM Improvement Proposals

MIP stands for MEM Improvement Proposal. A MIP is a design document providing information to the MEM community, or describing a new feature for MEM or its processes or environment. The MIP should provide a concise technical specification of the feature and a rationale for the feature.

We intend MIPs to be the primary mechanisms for proposing new features, for collecting community input on an issue, and for documenting the design decisions that have gone into MEM. The MIP author is responsible for building consensus within the community and documenting dissenting opinions.

Because the MIPs are maintained as text files in a versioned repository, their revision history is the historical record of the feature proposal.

## Structure

* Header - RFC 822 style headers containing metadata about the MIP, including the MIP number, a short descriptive title (limited to a maximum of 44 characters), and a description (limited to a maximum of 140 characters).
* Abstract - Abstract is a multi-sentence (short paragraph) technical summary. This should be a very terse and human-readable version of the specification section. Someone should be able to read only the abstract to get the gist of what this specification does.
* Motivation (optional) - A motivation section is critical for MIPs that want to change the MEM protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the MIP solves. This section may be omitted if the motivation is evident.
* Specification - The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations.
* Rationale - The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale should discuss important objections or concerns raised during the discussion around the MIP.
* Backwards Compatibility (optional) - All MIPs that introduce backward incompatibilities must include a section describing these incompatibilities and their consequences. The MIP must explain how the author proposes to deal with these incompatibilities. This section may be omitted if the proposal does not introduce any backward incompatibilities, but this section must be included if backward incompatibilities exist.
* Reference Implementation (optional) - An optional section that contains a reference/example implementation that people can use to assist in understanding or implementing this specification. This section may be omitted for all MIPs.

## Template

The markdown MIP template can be found [here](https://github.com/decentldotland/MIPs/blob/main/template.md)

## MIP Workflow

The MIP process begins with a new idea for MEM. Each potential MIP must have a champion -- someone who writes the MIP using the style and format described below, shepherds the discussions in the appropriate forums, and attempts to build community consensus around the idea. The MIP champion (a.k.a. Author) should first attempt to ascertain whether the idea is MIP-able. Posting in the Discord or a MIP GitHub issue is the best way to go about this.

Discussing an idea publicly before going as far as writing a MIP is meant to save both the potential author and the wider community time. It also helps to make sure the idea is applicable to the entire community and not just the author.

Once the champion has asked the MEM community as to whether an idea has any chance of acceptance, a draft MIP should be presented as a PR to the MIP repo. This gives the author a chance to flesh out the draft MIP to make it properly formatted, of high quality, and to address additional concerns about the proposal.

The MIP editors assign MIP numbers based on the original pull request number and change the status of MIP issues.

Approving editors will assign the MIP a number, give it the status "Draft", and add it to the MIPs git repository. The MIP editor will not unreasonably deny a MIP. For a MIP to be accepted it must meet certain minimum criteria. It must be a clear and complete description of the proposed enhancement. The enhancement must represent a net improvement. The proposed implementation, if applicable, must be solid and must not complicate the protocol unduly.

The MIP author may update the Draft as necessary in the git repository. Updates to drafts may also be submitted by the author as pull requests.

Once a MIP has been accepted, the reference implementation must be completed. When the reference implementation is complete and accepted by the community, the status will be changed to "Final".

## History

This document was derived heavily from Bitcoin’s BIP-0001 written by Amir Taaki which in turn was derived from Python’s PEP-0001. In many places, text was simply copied and modified.

## Copyright
Copyright and related rights waived via [CC0](./LICENSE.md).

