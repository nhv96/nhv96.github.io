---
layout: post
title:  "Errors Wrap And Unwrap"
date:   2024-11-30
category: til
---
# Errors wrap and unwrap
Should use `%w` instead of `%s` or `%v`, so that `errors.Unwrap` can unwrap the message and compare it using `errors.Is`.
