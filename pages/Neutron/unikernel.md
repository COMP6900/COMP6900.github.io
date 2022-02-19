---
layout: default
title: Unikernels
nav_order: 2
parent: Neutron
---
Instead of having the kernel run as a separate 'process' and other apps and stuff be run in a different address space and interface with the kernel through MMIO/syscall ABIs, we can compile and link the kernel to the apps. I really like this idea, but I mean, it would be easy to install new things I guess.
You would have to download a new app/update to disk. Then you have to recompile/link the kernel img and reboot it.
If there is a way to "hot reload" your OS quickly, that may be an idea. Or maybe keep the userspace running and just reload the backend somehow. I dunno. I'll keep looking at this idea but for now its not really something that is simple or practical for something I want.
