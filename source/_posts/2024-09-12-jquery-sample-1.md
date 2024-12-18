---
title: 简单的鼠标滚轮滑动事件 (jQuery)
date: 2024-09-12 17:30:30 +0800
tags: [html, jQuery]
---

**核心：**

`addEventListener("wheel", (event) => {});`

**注释：**

使用 `event.preventDefault();` 来阻断鼠标原本的滚动功能。

**仓库：**

https://github.com/LazyKnightX/samples/tree/main/samples/html/html-001

**参考：**

* [MDN - Element: wheel event](https://developer.mozilla.org/en-US/docs/Web/API/Element/wheel_event)

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <p id="info">
        page: 1
    </p>
    <script src="http://libs.baidu.com/jquery/2.0.3/jquery.min.js"></script>
    <script src="./script.js"></script>
</body>
</html>
```

**script.js**

```js
let c = 1;

function next() {
    c += 1;
    $('#info').html(`page: ${c}`);
}

function prev() {
    c -= 1;
    $('#info').html(`page: ${c}`);
}

addEventListener("wheel", (event) => {
    console.log(event);
    if (event.deltaY > 0) {
        next()
    }
    else {
        prev()
    }
});
```