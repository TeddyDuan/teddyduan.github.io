---
layout: post
title: 节流(debounce)与防抖(throttling)
date: 2019.11.12
---

> <a href="../debounce.html" target="__blank">防抖: debounce</a>

```javascript
const debounce = function(fn, time) {
  let timeout;

  return function() {
    const self = this,
      args = arguments;

    const tocb = function() {
      timeout = null;
      fn.apply(self, args);
    };

    clearTimeout(timeout);
    timeout = setTimeout(tocb, time);
  };
};
```

> <a href="../throttling.html" target="__blank">节流: throttling</a>

```javascript
const throttling = function(fn, time, interval) {
  let timeout,
    start = new Date();

  return function() {
    const self = this,
      args = arguments;

    const now = new Date();

    const tocb = function() {
      start = now;
      fn.apply(self, args);
    };

    if (now - start > interval) {
      tocb();
    }
  };
};
```

> [参考文章:【前端性能】高性能滚动 scroll 及页面渲染优化](https://www.cnblogs.com/coco1s/p/5499469.html)

> [返回]({{site.baseurl}}/前端优化总结)
