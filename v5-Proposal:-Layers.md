### PAIN

This page is dedicated to pain that countless people had when developing Flash and PIXI apps.

The goal is to stay with our successful PIXI v4 API and stage architecture, and add many features at one time while making minimal changes. I suggest we don't make big changes in API that have less effect than this proposal.

## Problem

Imagine that we have a number of characters in the stage, every character has a body, shadow and a healthbar. There are no filters/masks.

To show those things correctly, in vanilla pixi we have to 
1. put body, shadow and bar in their "layer" container respectively
2. move all three sprites when character moves
3. remove all those sprites when character leaves the stage

Simple game can easily have 5-10 types of child objects, and managing them without special solution for layering is difficult.

## Requirements for solution

Let's assume that we added special `display component` to DisplayObject to solve the problem.

1. Renderer-level must not be affected, all major changes must happen in the stage.
2. Without component, stage and interaction behaviour must be the same as before. Except some corner cases that we can sacrifice (renderable=false, alpha=0).
3. Component must not affect `container.children` and their order. All manipulations must be done in special render lists. 
4. Performance: if there are N objects of K different type, and objects of each type are not supposed to be sorted with each other (or they are pre-sorted), total time per frame must be O(N + K log K), and not O (N log N). 10000 bunnies and 10000 of their shadows must not lead to sorting of 20000 elements.
5. Tree invariants: object rendering order is affected by position of that object in pixi stage tree and display component of nearest parent that has it. Thus, component may change order of the whole subtree, but it does not affect neighbour subtrees.
6. Filters/masks: Some rendering features require push/pop when entering/leaving a tree node, and that's why we cant just make everything flat. There must be a way to put object inside the filtered container in render-time.
7. There must be fully dynamic option (compute lists every frame), and there might be static option (compute lists at components change).

## Solution

Lets assign a number (z-order or z-index) to some of containers, and create special Stage container that travers all over the subtrees and "flattens" it in array, every frame. Let us call that method `updateDisplay`, it determines the "display order" of elements, and stores it somewhere.

It sorts that resulting array, thus it does not affect container structure. This approach is better than phaser's Group because it does work with any tree structure. This approach can also work with interaction if `processInteractive` returns the the maximum "display order" instead of just a flag whether or not something was found.

### Sorting optimization

Suppose we have 1000 characters, so, sorting 3000 objects every frame with a custom comparator will be a performance problem. Suppose characters already sorted between them somehow by game logic: their Z-coord in isometry is maintained by user logic, and it sorts only when they are moved in world logic. 

We have only 3 different values for z-index, and that's the way we can optimize: create multiple `DisplayGroup` that can be assigned to sprites, each DisplayGroup has z-index. Stage stores a separate array of elements for each displayGroup it founds in subtrees, thus we only sort 3 those groups and not 3000 elements. Elements that don't have assigned displayGroup are handled in default display group. Group is also inherited.

Stage sorts all groups by z-index (fast), performs sorting inside of each group by z-order (slow, can be maintained by game logic instead), and it doesn't go inside containers that have special flag (like ParticleContainer or any container with filters/masks).

I define "z-index" as a variable that has multiple values, element with bigger z-index is drawn above others, like in Web page. "z-order" is the distance to camera, its floating point and lesser element is drawn above others.

### Layers

The problem with display groups is that we cant move elements inside filters or masks, its not clear where exactly pushFilter and popFilter will be called, because we flattened a tree into a number of subtrees. So, instead of maintaining one array of used displayGroups, Stage can detect Layers which will render elements with corresponding group. Now the algorithm becomes clear: 

1. if element has a parentLayer, it will be rendered in that layer. 
2. if element has a parentGroup, stage will find layer that owns that group and element will be rendered in that layer.

- Why not remove displayGroups at all?
- DisplayGroups can be global constants, while layer exists for specific stage, and assigning parentLayer through dependency injection patterns is troublesome.

### Camera

Assume we have Stage with layer Camera and container World. Lets specify that `world.parentLayer = Camera` and add condition in camera's `renderWebGL` method that changes the current projectionMatrix. When we ask `renderer.render(camera)` it will actually render the world through a projection. Also, Camera can be placed anywhere in the world itself, we just have to make sure that `camera.parentLayer` is not camera itself.

Its good for API, because renderer will be as simple as before, we just pass it camera instead of the stage.

### Batching 3d

How can we have a smart batching in that case? Create a batcher container, assign `world.parentLayer=batcher`, and sort elements by their texture inside. Its very useful for 3d geometries that dont use alpha. It can be achieved without "Arrays.sort" too. The idea is that we don't complicate our low-level which is already over-complicated with multitexturing.

### Deferred.

Layers approach also allows to emulate deffered rendering in 2d engine by adding multiple layers and camera and using filters on them. We don't have to describe deferred rendering on low-level and that's awesome.

### Implementation:

Layers: https://github.com/pixijs/pixi-display/tree/layers
Camera: no implementation yet.

Demos: http://pixijs.github.io/examples/#/layers/lighting.js , http://pixijs.github.io/examples/#/layers/zorder.js

### Adopters

@bqvle, http://tankwars.io/

## Pros

Relatively simple implementation that solves multiple difficult problems. The plugin exists for a year, and people love it.

## Cons

I don't know of other renderers that use same approach, I believe this is something new, and there will be many questions like "why did you do that instead of copying other engines".

Extra recursion `updateDisplay` can give us a problem, but I dont see how can we make other layers/renderQueue implementation that will have better speed. However, it can be solved if we use better stage, I will make separate containers improvements proposal. Actually, I already solved that problem in new iteration, I'm working on documentation and tests at the moment :)