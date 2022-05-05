---
layout: default
title: Lecture 7
parent: Lectures
---

## Overview

## Case Study: Fuchsia OS

Fuchsia claims to be built on 4 key principles:

1. Simpleness
2. Security
3. Updatability
4. Performance

### Fuchsia Model

In Fuchsia, "everything is a component" and it is the atomic unit of executable software.

Im guessing this comes from flutter, where you have components with state or no state and are rendered at each cycle.

Components communicate with their dependencies through FIDL. FIDL basically describes IPC protocols of components.

Components and their deps and executables / libraries are distributed through packages. A package is the unit of distribution in Fuchsia. Components only used shared (dynamic) libraries that are included in the same package as the component. Which means they do not use shared libraries elsewhere on the filesystem.

Why is this the case? Cause we want to allow multiple components in the same package to use the same shared library. And bound our resolution time in case of any transitive dependency closures due to inter-package deps.

Components are organised to keep critical deps in a package and service deps are resolved by runtime resolution. So kind of similar to a web services model.

### Fuchsia Components

[Here](https://fuchsia.dev/fuchsia-src/concepts/components/v2/introduction).

All software running on fuchsia have the minimalist of privileges before the user grants it to them. They dont even have the privilege of allocating memory, only when they try to, will the OS prompt the user to provide the requesting process permission to do that specific action.

### Fuchsia Syscalls

From an "everything is an object" model, we find that the concept of files in traditional UNIX land is replaced by a more generic concept of an "object":

- **object_get_child** - find the child of an object by its koid
- **object_get_info** - obtain information about an object
- **object_get_property** - read an object property
- **object_set_profile** - apply a profile to a thread
- **object_set_property** - modify an object property
- **object_signal** - set or clear the user signals on an object
- **object_signal_peer** - set or clear the user signals in the opposite end
- **object_wait_many** - wait for signals on multiple objects
- **object_wait_one** - wait for signals on one object
- **object_wait_async** - asynchronous notifications on signal change

### Fuchsia vs. UNIX

So instead of files you have kernel objects.

Instead of file descriptors you have "handles". Which seems to be mostly the same.

Other ideas like processes and threads still exist and require syscalls.

Communication syscalls like sockets, channels, streams, fifos also exist.
