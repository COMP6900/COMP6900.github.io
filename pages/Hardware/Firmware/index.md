---
layout: default
title: Firmware
has_children: true
parent: Hardware
---

## Firmware = Lowest Level Software

Usually written in C/Rust/Assembly. You need really tight control over what your doing. Includes bit manipulation and register and line control by the clock cycle. Posedge and Negedge. Need to program the microcontroller by the clock cycle so yes.

## Motherboard Firmware

Prob the place with the most code. Largest ROM. Can be upgraded. Usually shouldnt though unless you got some security patches or maybe some efficiency updates. Usually no new features.

## RAM Controller

Prob some small amount of code that checks the stuff. Can prob just be hardcoded into the controller hardware logic in VHDL/PCB.

## Graphics Card Firmware

Usually nothing, mostly in drivers and compiler code.

## USB Controller

Prob some small stuff too.

## Peripherals

Mouse, keyboard, modem, etc. These things will prob have some firmware that can be upgraded. Either better use of features, new software/GUI features, etc. Usually downloaded quite fast, except modem which is kinda like a whole computer.

## Chair?

Apparently your chair can have a microcontroller in it with dynamic software ROM execution. In this case, idk, RGB gaming chair.
