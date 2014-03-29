---
layout: post
title: Understanding Millwheel
tags: millwheel
date:   2014-03-29 10:00:00
---

I am reading Google's interesting paper on the [millwheel][millwheel] system.
Below is a digest.

- One time input to the system: computation graph, application code for nodes
- Ongoing stuff handled by the system: persistent state, streaming input

The programming model provides:

- short term persistent storage
- long term persistent storage
- low watermark timestamp
- logical time
- out-of-order record processing
- exactly-once semantics for record processing.

The persistent store is per-key per computation. So if there are 4 nodes in the
computation DAG, there will be 4 "tables" with rows keyed on the key in the
input record. Timers are based on low watermarks.  The unique id which is used
for ensuring exactly once semantics is assigned by the system and not by the
user (not in the input). In a strong production, output data, state, timers --
all are snapshotted in a single per-key per-computation atomic write. In a weak
production, output data is sent forth optimistically assuming the receiver can
ACK it quickly.  A write sequencer is assigned to a lexicographic key interval
of each computation. A sequencer ensures that older pending writes dont
overwrite results of newer writes. Also the low watermarks are persisted for
data consistency.

[millwheel]: http://research.google.com/pubs/pub41378.html
