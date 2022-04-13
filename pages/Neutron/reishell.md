---
layout: default
title: Rei Shell
parent: Neutron
---

## Overview

Rei shell is a shell application with a builtin gui as well as a default vga text buffer output.

On a default build, one can expect:

- `reis` to be the default terminal emulator, stored in `/packages/reis`
- `rei service` to be online and accepting socket connections via ssh

## Commands

Rei shell (reis) is an interpreter for rei with direct access to Neutron services and functionalities such as pipes, builtin commands and autocompletion.

Unlike standard reic, reis is interpreted and is compiled with NeutronAPI directly. 
