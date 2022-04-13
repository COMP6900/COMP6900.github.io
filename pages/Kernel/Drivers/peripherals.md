---
layout: default
title: Peripheral Drivers
parent: Drivers
grand_parent: Kernel

---

Peripheral drivers usually go through USB or some other generic port. On PI and other embedded devices, prob GPIO.
So the first thing is to ensure the USB port controller works properly. Then when something is plugged in, that should send an interrupt to the BIOS code. The kernel can then query that code and intercept it to do complex things.

## Mouse Drivers
You need a way to store changes in (x, y) on each clock edge. When stationary, driver doesnt have to update its state. 

### Observer pattern
If using a caller prompted observer pattern then that should simplify a lot of it too. So on every tick, its the firmware that generates the interrupt rather than the driver querying the firmware.

## Keyboard Drivers
Even simpler than mouse drivers. You only need to check press and release. Then send those calls to the keyboard daemon and have the OS DE and apps handle the rest.

## Headphone Drivers
I actually dunno that much about sound stuff. I'll come back to this later.
