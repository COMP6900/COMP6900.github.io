---
layout: default
title: Vulkan
parent: Terraformer
grand_parent: Neutron
---

## Overview

Vulkan is a C/asm graphics API. Libraries can be built using vulkan like opengl's GLAD, GLFW, etc. GLFW actually supports vulkan.

![](/assets/img/games/hardware_overview.png)

### Is Vulkan superior to OpenGL?

If your code can take advantage of the lower level vulkan constructs to produce finer grained control over the gpu, then you prob will see quite some performance gains.

But if your code kinda sucks then you prob wont see much of a difference. Once again, code quality and principles like proper organisation, KISS and good threading management is everything.

### Differences vs OpenGL

- Object-based. Not a global state machine that you tell to make changes
- State is per object and stored in local command buffers in VRAM. Unlike Opengl where we link state with a global context like glTexture
- Multithreaded gpu code support. In Opengl you cant write multithreaded shaders/code directly, only CPU multithreading
- Allows control over VRAM as if it were like RAM. Virtual paging and synchronising access. Opengl manages all of it for you if you want
- BUT: No builtin error checking code like OpenGL. You have to hook onto the validation layer for debugging

## Pipeline

The graphics pipeline is a 6-stage pipeline of assembling vertices into shapes (usually triangles), rasterising the triangle edge vectors into pixels, and coloring in those pixels with the correct color either linearly interpolated or more commonly, through a texture. Some post-processing effects like TAA, bloom, etc. can also be applied on the previous frame in another pipeline cycle after the first cycle has produced a full frame.

## Game Loop

Like an arduino program, a game should have a 'loop' that continuously executes. This is called 'Loop/clock-based semantics' and is useful when we need to continuously tell multiple components like the CPU and GPU to do things whenever something changes.

The entire computer itself operates on a clock based system where we synchronise each hardware device to do its next queued action at the same time. This allows us to properly sync device accesses/reads and modifies in a predictable manner.

In a game, we think the same way. Say we have 10 different NPCs in a town with scenary in the background like trees, particles, the sky.

- Each NPC would have to do its default animation, e.g. standing still, waving when the player comes near. Maybe walk their default route and walk around things that may be colliding with them (A*)
- The stuff in the background will prob be unaffected by much. IT would just be looping over its default animation again and again
- You, the player has to move around, and watch your step to not bump into things.

To synchronise all these animations (local transformations) and object translations (movement of NPCS and the player) so they all happen at the same time and do not conflict with each other, e.g. clipping through objects and NPCS, we need to keep track of all the 'live' entities' positions and bounding boxes at every 'tick'.

- Each tick we can: prioritise the player's input movements. If they would collide with the bounding box of anything else, we can move them out of the way or stop them before collision
- For other objects like NPCs, we prioritise based on any arbitrary order and do the same thing
