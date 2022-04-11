---
layout: default
title: Unikernels
parent: Kernel
---

## Idea of the Unikernel

Instead of having the kernel run as a separate 'process' and other apps and stuff be run in a different address space and interface with the kernel through MMIO/syscall ABIs, we can compile and link the kernel to the apps. I kinda like the idea, but I mean, it prob wouldnt be easy to install new things.

You would have to download a new app/update to disk. Then you have to recompile/link the kernel img and reboot it.
If there is a way to "hot reload" your OS quickly, that may be an idea.

### Solution to Unikernel Staticness

Or maybe keep the userspace running and just reload the backend somehow. Apparently it is possible to run a VM after you boot the unikernel img. Then you can download and run apps on the fly.

I dunno. I'll keep looking at this idea but for now its not really something that is simple or practical for something I want.
