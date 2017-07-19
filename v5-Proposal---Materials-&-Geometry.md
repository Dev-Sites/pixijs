<!-- --- title: Proposal: Materials & Geometry -->

## Problem

The current underlying structure of PixiJS is that each feature (Sprites, Graphics, TilingSprite, ParticleContainer, etc) implements its own "backend". This is in part due to the natural growth of PixiJS throughout the years. Unfortunately it causes two major problems today: increased maintenance burden for changes, difficult low-level integration outside of the library.

### Increased Maintenance

Generally over the years new features have been added and existing features have been expanded by the addition of more code to the base. This often results in features developing independently of one another. While this is not inherently a problem, it does give rise to a maintenance issue where adding a common feature to be shared across all existing features can be difficult. Since each individual feature has its own backend it can time consuming to add new functionality to different backend, or to track down in which subsystem a bug resides.

### Complex Low-Level Integration

Having features implementing their own methods of rendering and process makes it difficult as a library author to integrate with the PixiJS pipeline. Mostly because the reality is that there is no real, codified, pipeline. There are no common building blocks that all features are built off of, that can easily be used to build external or additional features. This means each library author wanting to extend pixi basically reinvents the wheel each time they extend. We can work to expose multiple "tiers" of interfaces that people can use to integrate with PixiJS.

Current methods of extending PixiJS with a new feature often involve writing your own `ObjectRenderer`. This can be very cumbersome and complex, depending on the feature. Having to rewrite ~500 lines of `SpriteRenderer` in order to have an object that renders in some custom way is too much.

## Solution

**_ToDo / WIP_**

In order to solve these two problems, I propose we attack the maintenance issue by unifying how our own features work. We can do this by creating a few layers of API that each expose a level of interface that different users will find acceptable.

From the lowest, to the highest level interface I propose the following:

1. _GL Tier_ - Utilities and wrappers that directly interact with WebGL APIs. These simplify WebGL interactions and keep WebGL semantics.
2. _Core Tier_ - Higher level interactions with WebGL that lose GL semantics. Uses abstract objects that work to perform potentially complex WebGL tasks.
3. _Feature Tier_ - High level objects that use the core tier to implement complex scenes and rendering mechanics.

I expect that library authors will mostly be using technology in the _Core Tier_, and end-users will often be using tech in the _Feature Tier_.

A more detailed explanation of each tier, and modules that exist within them, can be found below:

### _GL Tier_ (`@pixi/gl`)

### _Core Tier_ (`@pixi/core`)

### _Feature Tier_ (`@pixi/sprite`, `@pixi/graphics`, `@pixi/text`, etc)




--------

# INCOMPLETE, TEMPLATE FOLLOWS



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