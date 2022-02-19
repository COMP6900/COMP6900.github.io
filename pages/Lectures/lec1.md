---
layout: default
title: Lecture 1
parent: Lectures
---
## Week 1, Lec 1
This is the greatest course, ever.

## Course Structure
Feburary 19th - May 11th
Weeks. In COMP6900 We do metric weeks. That means a week starts on day 1 and ends on day 10. A 'month' has 10 weeks. A semester has 1.5 months.
So that means a semester is exactly 150 days. Given 365 days a year, we have 2 semesters and 2-3 'hyper' courses each semester. A lot of the time it is very chill and very cool. No tutorials but rather online based question-answers in forums and on demand video/voice sessions with the dictator or an AI.

Week 1-3
- Asst1, upload to comp6900.github.io/asst1/22s1
- Hardware
- Kernel Basics
- Low level programming with ASM, C and Rust
    - Cross Compilation
- Introduction to Quanta, an OS that just works
- Introduction to Rei, a language that just works on Quanta
  - Pseudocode will be written in Rei

Week 4-7
- Asst2, upload to comp6900.github.io/asst2/22s1
- Introduction to the Neutron Kernel, the greatest kernel ever
- Implement threading and syscalls in Neutron, ASST2 Pt.1
- Why are modern kernels so bad?

Week 8-12
- Asst3, upload to comp6900.github.io/asst3/22s1
- Implement processes and ELF loader in Neutron, ASST3 Pt.1
- Implement userspace and graphics, ASST3 Pt.2

Week 13-15
- Finals week
- Tests knowledge on kernels, OS architecture, hardware in multiple choice and short answer questions in `rei`
- Revision material uploaded to comp6900.github.io/lectures/revision-22s1.html
- Final exam, done completely online in comp6900.github.io/final/22s1


## FAQ
Is this legit?
- Yes

Is the info good?
- As real as I can make it, yea. Well if theres some mistake I'll fix it so dw.

Does this count towards something?
- I'll give you a certificate or something if you pass. Need at least 50% to pass.
- To pass you have to show you understand what you're doing. Mostly assessment based with a 35% final exam.

Are there video lectures?
- Of course. Uploaded to https://www.youtube.com/channel/UC6qO4W9BRAPpUOUD43aDDvA. Or alternatively in the Videos section which embeds the videos with a custom HTML5 player.

## What is COMP6900?
Like a human body, a kernel can be thought of as an organism with different organs working together harmoniously to survive its environment. Its environment in this case is a messy ecosystem of code, apps, hardware and dumb users.

As a programmer, you need to know understand how a kernel survives its environment and mediates between stuff to achieve its goal.

## Middleman Analogy
Like a middleman, a kernel sits at a stand for any requests. When someone has a request, they go up to the middleman and asks for something. The middleman then sifts through their documents to find something that can satisfy their request. If it cant find an appropriate thing, then it will have to talk to other middleman on a lower level than it. And wait for them to reply back. And if they dont have it, they will have to talk to even lower middlemen.

So this is basically the bane of the kernel. Sit and wait until someone (an app) needs something (service). Then try to best fulfill it (daemons, drivers). If request is valid and can be done, then its only a matter of time. If it cant, then the asker must ask again at a later time or ask something else. The middleman (kernel) doesnt like being held by bad requests and time (IO) so it tries its best to mitigate it (timetable hueristics, clock algorithm, etc). A middleman wont be successful if he doesnt get any requests would he? So the busier he is, the better job he is doing (more CPU/resource utilisation). A middleman also has to protect his informats and the information itself so he doesnt get a bad rep. You dont want a middleman giving out info that could jeoprodise someone would you? So you ensure the buyer is reputable (paging, permissions) and the info itself is reputable (headers in tact, journalling, corrupt bits).

## RISCV64GC
In the risc world we deal with concepts like:
- RISC pipeline (vs "hyperthreading")
- ALU Load/Store instructions (vs Register-Memory)
- Extensions, for floats, atomics, hypervisor (virtualisation), etc.

### What the hell is RISCV64GC?
The answer is RV64IMAFDZicsrZifenceiC
- 64I: 64-bit integer extensions
- M: Multiplication
- A: Atomics
- F: Floating Point (32-bit)
- D: Floating Point (64-bit)
- Zicsr: Control & Status
- Zfencei: Extra Memory Fence/Protection
- C: Compressed Instructions (16-bit)

### Wait what is RISCV?
- An open source ISA.
- Much better than x86, objectively. I said so.
- Upside: Simple, elegant, modular, extensible.
- Upside: Less electricity goes into decoding the instructions (x86), so less power usage, though maybe more compiler work
- Upside: Easier assembly. Dont have to remember a billion different instructions and 2 different syntaxes. In fact you can use the assembly syntax I made [To be uploaded soon].
- Downside: More careful about cache since x86 programs tend to be smaller in size and can fit on CPU cache.

## RISC-V Instruction Types
The people at sifive seemed to realise the potential of hardware efficiency gains. So they came up with 6 different ways to encode a 32-bit riscv instruction.

### R Format
For arithmetic operations like `add`. Stuff that can be done in the ALU/FPU. <br/>

`<instruction> <dest_reg> <src_reg1> <src_reg2>` 

32-bit Instruction: <br/>
[ func 7 | src_reg2 5 | src_reg1 5 | func 3 | dest_reg 5 | opcode 7 ]

### I Format
For arithmetic operations with an immediate like `addi`. Also loading to an ALU register from memory

32-bit Instruction: <br/>
[ immediate_val 12 | src_reg 5 | func 3 | dest_reg 5 | opcode 7 ]

### S Format
Store operations from register to memory, stuff like `sw`.
- In RISC world, must load (I) a val from memory to a register, then apply arithmetic ops (R/I), then store the val back (S)

32-bit Instruction: <br/>
[ immediate_val[11..5] 7 | src_reg2 5 | src_reg1 5 | func 3 | immediate_val[4..0] 5 | opcode 7 ]

### SB Format
Branch to another address, updates the PC directly by specifying an offset instead of incrementing. Stuff like `beq`

TYPE 1: \
[ immediate_val[12..5] 7 | src_reg2 5 | src_reg1 5 | func 3 | immediate_val[4..1] 5 | opcode 7 ]

TYPE 2: \
[ immediate_val[10..5] 7 | src_reg2 5 | src_reg1 5 | func 3 | immediate_val[4..11] 5 | opcode 7 ]

### U Format
Arithmetic ops with upper immediates like `lui`
- In riscv, upper immediates are always 20bits, unlike *that* other arch

32-bit Instruction: \
[ immediate_val[31:12] 20 | dest_reg 5 | opcode 7 ]

### UJ Format
Jump to another address. Different to branch since you directly specify an address/offset rather than a relative offset. Also `link`. So `jal`

32-bit Instruction: \
[ immediate_val[31:12] 20 | dest_reg 5 | opcode 7 ]

- so `jal` stores the return address in `dest_reg` unlike branch
- Theres like at least 2 different types of instruction types for jal and jalr cause why not I guess

Do you have to remember these? No. But you will have to implement it and `autotest-6900` has to work so yea.
