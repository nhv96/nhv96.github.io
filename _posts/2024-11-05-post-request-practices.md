---
layout: post
title:  "Post Request Practices"
date:   2024-11-05
category: til
---
# Post request best practices
When making POST request, don't store sensitive data in request body, because it can be altered by somebody else, instead store it in the header.
```
POST /booking/ticket
Header: JWT token or session
Body: {
    ticketID
}
```
