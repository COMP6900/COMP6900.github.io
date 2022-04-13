---
layout: default
title: Neutron API
parent: Neutron
---

## Userspace Overview

Most things can be built with `rust-std` or bare neutron service suite with rei. However apps that are graphical in nature should use `arckit` which bundles `arcwm`, general wrapper classes and an object persistence library for storing and manipulating data.

When building executables/libraries with rust for `target_os=neutron_arc64`:

```rust
use neutron_arc::*;

// 

```

## Kernel Overview

When writing kernel level drivers or kernel modules, one should use `neutron_api`.
