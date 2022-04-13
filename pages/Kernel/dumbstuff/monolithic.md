---
layout: default
title: Monolithic Kernels
nav_order: 2
parent: Dumb Stuff
grand_parent: Kernel

---

Really, really bad idea. Actually not that bad but if you look at linux, then you know what I mean.

## Why is it so bad?

Because we are bloating the kernel. >50% of linux codebase are driver related code. What?
So you basically have something that barely works on new systems and APIs. Amazing.

## Features

### Place as many things in the kernel as possible

Lol

### Try to support old APIs, then bandage around it to support the new stuff

???

### Have an overly complex and messy API for userspace apps

[Yes, so easy to develop for I know](https://www.kernel.org/doc/htmldocs/kernel-api/)

### Inefficient communication between layers and systems

That linux network stack though...

![](https://web.archive.org/web/20170905131225if_/https://wiki.linuxfoundation.org/images/1/1c/Network_data_flow_through_kernel.png)
