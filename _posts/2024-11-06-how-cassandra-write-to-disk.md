---
layout: post
title:  "How Cassandra Write To Disk"
date:   2024-11-06
category: til
---
# How it write to disk
1. write to a log (disk)
2. memtable (memory)
3. ack the write
4. periodically flush the memory to disk
5. write is just an append operation
