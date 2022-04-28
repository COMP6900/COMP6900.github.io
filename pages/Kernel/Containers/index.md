---
layout: default
title: Containers
has_children: true
parent: Kernel
---

## Containers

Great for cross platform app development and deployment. Can then be managed by stuff like kubernetes across multiple hardware systems. Instead of managing VMs.

## FreeBSD Jails

Jails are a very cool thing and prob the first "container" framework ever.

Basically a jail has its own:

1. dir subtree
2. hostname
3. ip address
4. a command

Its dir subtree is the entry point of the jail when it starts up. The IP address is usually an existing network interface like `172.19.0.2:5000` which is local to the bridge. When the bridge is connected to an external network instead of just on the host, the bridge itself may be assigned something like `192.168.1.2:8000`. This would prob refer to the single machine which hosts a bunch of jails.
