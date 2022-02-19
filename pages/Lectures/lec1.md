---
layout: default
title: Lecture 1
parent: Lectures
---
## Week 1, Lec 1
This is the greatest course, ever.

## What is COMP6900?
Like a human body, a kernel can be thought of as an organism with different organs working together harmoniously to survive its environment. Its environment in this case is a messy ecosystem of code, apps, hardware and dumb users.

As a programmer, you need to know understand how a kernel survives its environment and mediates between stuff to achieve its goal.

## Middleman Analogy
Like a middleman, a kernel sits at a stand for any requests. When someone has a request, they go up to the middleman and asks for something. The middleman then sifts through their documents to find something that can satisfy their request. If it cant find an appropriate thing, then it will have to talk to other middleman on a lower level than it. And wait for them to reply back. And if they dont have it, they will have to talk to even lower middlemen.

So this is basically the bane of the kernel. Sit and wait until someone (an app) needs something (service). Then try to best fulfill it (daemons, drivers). If request is valid and can be done, then its only a matter of time. If it cant, then the asker must ask again at a later time or ask something else. The middleman (kernel) doesnt like being held by bad requests and time (IO) so it tries its best to mitigate it (timetable hueristics, clock algorithm, etc). A middleman wont be successful if he doesnt get any requests would he? So the busier he is, the better job he is doing (more CPU/resource utilisation). A middleman also has to protect his informats and the information itself so he doesnt get a bad rep. You dont want a middleman giving out info that could jeoprodise someone would you? So you ensure the buyer is reputable (paging, permissions) and the info itself is reputable (headers in tact, journalling, corrupt bits).
