---
layout: default
title: Terraformer
parent: Neutron
has_children: true
---

## Terraformer3D Overview

A simple unity/godot like game engine for building 3D games with rust-wgpu and blender. All types of models can be stored in `.blend` and it should be the format's responsibility to make it more efficient.

- specialisation 1: for making first/third person views. View distance medium and configurable.
- specialisation 2: for making top down 3D views of relatively small maps at a time. Or a mid-large sized map covered by fog.

## General Things

- wpgu is an abstraction over the default API, e.g. vulkan

## WGPU

Rust [wgpu](https://wgpu.rs/) itself tries to mimick vulkan's interface. But in reality it translates those wgpu calls to whatever API the host uses.

- [WebGPU](https://gpuweb.github.io/gpuweb/) is an interface specification meant for rendering things in a web environment/browser
- A [tutorial](https://sotrh.github.io/learn-wgpu/beginner/tutorial1-window/#boring-i-know) on wgpu
