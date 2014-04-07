---
layout: post
title: Some Interprocess communication patterns
date:   2014-04-07 12:00:00
tags: ipc
---

IPC patterns quick reference (from this [book][book]):

- Mutual exclusion (2 process, n process)
- Critical section (2 process, n process)
- Signaling (2 process, n process)
- Rendezvous (2 process, n process)
- Producer/Consumer
- Server/Client
- Database update/read

The fundamental means of communication between processes:

- Message passing
- Shared memory

[book]: http://www.cs.unm.edu/~crowley/osbook/begin.html
