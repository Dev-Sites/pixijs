This document exists as a guide for writing proposals for additions and/or changes to PixiJS. While
not all changes will require a proposal, larger changes should be peer reviewed by the team and the
community before being implemented. This helps to build consensus on an implementation before work
starts. It can also be an opportunity to fully develop and improve on ideas by including the entire
community in the construction.

To ensure that proposals follow a similar structure and contain all the information necessary for
implementing a proposed solution, some guidelines are written below.

## Title

Each proposal should be titled `<version> Proposal: <title>`, where `<version>` is the major version
that the proposal is for and `<title>` is the user-defined title. For example: `v5 Proposal: Particles`.

## Template

Below is a quick markdown template useful when creating a new proposal. Each of the sections are
described in detail, please read them before writing your document.

```markdown
## Problem

A good design proposal starts with a problem statement. What are users, developers, or contributors
having difficulty with? What common task do we need to make easier? Who is affected? Starting with
several examples that show where the library isn't meeting the needs of programmers will help make
clear what problem is being fixed. Be sure to mention what (if any) the current workarounds are.

Provide code examples of scenarios that are difficult to handle currently, and their workarounds
(if any).

## Solution

Outline the proposed solution to the stated problem.

### API

In this section describe the proposed API. It should describe any new classes or plugins that should
be added to support this solution. Describe the philosophy behind the API, and how it solves the
problem statement.

Answer questions like:

- What does the new API look like?
- How does this API solve the problems mentioned above?
- What parts of the library are affected?
    * Is it a new plugin?
    * Changes to an existing plugin?

### Examples

Provide examples of using the solution. Examples are key to understanding how the solution is effective.
Readers should be able to directly compare examples in this section to the problem statements code
examples.

## Implementation

In this section describe in as much detail as possible the implementation of the proposed solution.
Start with a general explanation of how any internal structures interact with the public API and
any concepts or patterns that will be used. Follow up by creating multiple sub-categories that
explain specific details of the implementation. Explicit details, even when they seem trivial, are
better than a miscommunication of intent later.

### Architecture

This section is useful to explain the overall architecture of the solution. Diagrams displaying
class interaction, data flow, usage-patterns, and other useful high-level information is strongly
encouraged. How does this code interact with existing systems? How does it interact with the
public API?

### Sample Code

Code samples are useful as necessary, but don't implement the feature here. Instead, ensure that
anyone maintainer reading your document has all the information necessary to write the feature.
Interfaces, and pseudo-code can be useful to convey ideas.

### References

Mathematical functions, reference materials, and source articles should be included here as necessary.
It is often helpful to see the mathematics behind the code for clarity.
```

**Attribution**

Some content in this guide was based upon TypeScript's [Writing Good Design Proposals][ts-wgdp] wiki article.

[ts-wgdp]: https://github.com/Microsoft/TypeScript/wiki/Writing-Good-Design-Proposals
