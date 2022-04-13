---
layout: default
title: Vulkan
parent: Graphics
grand_parent: Drivers
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

# Objects

Each vulkan object is prefixed by `Vk`. E.g. `VkSampler`. Instead of state machine semantics like `glTexture2D` we use local objects.

- A device requires a physical device, i.e. it subclasses it. A descriptor set requries a descriptor pool
- A phyiscal device has a queue family and memory heap

## Command Buffer

A `VkCommandBuffer` requires an event, pipeline, render pass, descriptor set and pipeline layout. This means a single command needs those things to be validated and placed into the device queue

- the command is submitted to the queue using `QueueSubmit()`. It is fenced and has a semaphore to ensure no data races entering the queue

## Descriptor Set

A `VkDescriptorSet` is one of two key things we need before submitting a command. We need to query and manipulate the GPU memory with `DeviceMemory`. In Device Memory, we bind a `Buffer` to it and create a `BufferView`.

- A buffer can be a vertex buffer, a uniform buffer, a framebuffer. Usually we def want a vertex buffer if we want to draw anything. We have to bind it with an `Image` object to actually create the memory for it

Next we create a `VkSwapchainKHR` to queue our rendered frames for the GPU to output to the display. We queue `VkImage` objects. And those objects have an associated `VkImageView`.

- if we want to apply a texture, we need a sampler. Create a `VkSampler` object with the view objects

With the 2 `View` objects, we can combine them with a `Sampler`, `DescriptorPool` and `DescriptorSetLayout` to create a full `DescriptorSet` object.

- we then combine the descriptor set object with a `PipelineLayout` to bind the descriptor set to be copied into the command buffer. The `PipelineLayout` is important for creating a valid pipeline (i.e. 5/6-stage pipeline) with shaders and stuff so the GPU knows what to do

## Framebuffer

We have to bind a `VkFramebuffer` object with a `VkRenderPass` object to call the `CmdBeginRenderPass` after specifying the descriptor set. This allows the rendered texture to be imaged onto the Framebuffer for manual clearing or post processing.

- The `RenderPass` object same pipeline so vulkan knows which cycle to output to the framebuffer

## Query

Next we can call `CmdBeginQuery` to tell Vulkan to start specifying our final query. We also need a `VkQueryPool` object to create a `Query` object. If we want we can hook onto the query's functions to specify how to begin the query

- We could also just use the default
- But its great to use for specifying stuff like `Occlusion`, Timestamping, etc. Good for things like depth pre/post shading tests

## Binding the Pipeline

A `VkPipeline` object can be created from a `PipelineLayout` object from earlier and the `RenderPass` object. We then combine it with newly created `ShaderModule` and `PipelineCache` to create the final `Pipeline` object.

- Now we can `CmdBindPipeline` to actually bind the final pipeline specification of a shader program. The shader program was compiled from the user specified vertex -> geometry -> fragment code
- Note: `PipelineCache` is optionally but recommended as it can really reduce VRAM usage by telling the GPU to cache the pipeline. Then save the cache to disk for later use as well

## Wait for Completion

Then we make an async call `CmdWaitEvents` and wait for the GPU to process the vertex information (Descriptor Sets) through its shader program.

- A `VkFence` is the signal to the program that the execution of the task is completed
- `Semaphore` can control accesss to VRAM resources like vertices. We dont want two different queues (usually 'kernels') to have a data race on modifying the same vertices
- An `Event` can be used to listen to specific GPU events. Note, not the same as input events, would use the CPU and something like GLFW for it
