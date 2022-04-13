---
layout: default
title: SBI
parent: Boot
grand_parent: Kernel

---

## Supervisor Binary Interface

The supervisor (S-Mode) can have access to a higher abstraction of system interfaces like ACPI. A kernel supervisor can make these requests to its SEE via `ecall` as defined by SBI.
