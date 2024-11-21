---
layout: post
title:  "Retrying Commands In Linux"
date:   2024-11-21
category: til
---
# Retrying some commands in Linux

```bash
until ssh root@mynewvm; do sleep 10; done
```

Only works in Linux.
