---
title: HTTP fetch & post
date: 2025-1-12 09:19:23 +0800
tags: [http]
---

```javascript
fetch("https://id.twitch.tv/oauth2/token", {
  method: "POST",
  body: JSON.stringify({
  }),
  headers: {
    "Client-ID": "123",
    "Client-Secret": "456",
    "Grant-Type": "client_credentials",
    "Content-type": "application/json; charset=UTF-8"
  }
})
  .then((response) => response.json())
  .then((json) => console.log(json));
```
