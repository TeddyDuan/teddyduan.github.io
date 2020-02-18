---
layout: post
title: setTimeout&setInterval&倒计时实现
date: 2019.11.12
---

> setTimeout/setInterval 中的 delay 参数，指的是将 fn 加入到宏任务执行队列的时间，而非 fn 具体执行的时间。

### setInterval

1. 根据 MDN 文档, delay 的最小间隔值为 4ms; 有些文章里的 10ms 可能与具体浏览器实现有关。
2. 当宏任务队列中已有 fn 且 fn 未得到执行，则当次 fn 不会被加入到队列中。
   - 导致某些间隔被忽略
   - 两次间隔之间的实际执行时间可能比预期小(JavaScript 高级程序设计 p611)

### setTimeout

1. 根据 MDN 文档，delay 最小间隔值为 4ms;
2. setTimeout 模拟 setInterval

   ```javascript
   setTimeout(function() {
     setTimeout(arguments.callee, 1000);
   }, 1000);
   ```

### setTimeout 倒计时

1.  倒计时总的毫秒值应该从服务端获取，不应该依赖客户端时间。
2.  客户端时间的差值可以作为倒计时参考。

    ```javascript
    function doCountDown() {
      const cd = 60000; // 实际场景中，cd应该是从服务器传过来的时间间隔。
      countDown(cd, 100);
    }

    function countDown(cd, interval) {
      const target = document.getElementById('countDown');

      const start = new Date().getTime();

      // 计数器用于计算timeout执行时的偏差；计基础时间start, 当前fn执行时间为now;
      // offset = now -start;
      // 若完全无延时，则 offset === count * interval;
      // 计os = offset - count * interval;
      // 若os超出了interval, 则setTimeout的delay值置为0(实际为4)
      // 否则setTimeout的delay值置为interval-os;
      let count = 0;
      const timer = function() {
        const now = new Date().getTime();
        const offset = now - start;
        if (offset >= cd) {
          target.innerText = '倒计时结束';
          clearTimeout(timer.toId);
        } else {
          target.innerText = `还剩${ms2Time(cd - offset)}`;

          const toOffset = offset - ++count * interval;
          const next = toOffset > interval ? 0 : interval - toOffset;
          timer.toId = setTimeout(timer, next);
        }
      };

      timer.toId = setTimeout(timer, interval);
    }

    function ms2Time(ms) {
      let val = ms;
      const sss = String(Math.floor(val % 1000)).padStart(3, '0');

      val = Math.floor(val / 1000);
      const s = Math.floor(val % 60);

      val = Math.floor(val / 60);
      const m = Math.floor(val % 60);

      val = Math.floor(val / 60);
      const h = Math.floor(val % 24);

      val = Math.floor(val / 24);
      return `${val}天 ${h}时 ${m}分 ${s}秒:${sss}`;
    }
    ```

    <html lang="en">
      <head>
        <meta charset="UTF-8" />
        <title>Document</title>
        <style>
          body {
            width: 100%;
            height: 100%;
            margin: 0;
          }

          body > div {
            margin: 10px 20px;
          }

          .btn {
            margin: 0px 10px;
            height: 30px;
          }
        </style>

      </head>

      <body>
        <div>
          <button class="btn" onclick="doCountDown()">doCountDown</button>
        </div>
        <div id="countDown">...</div>

        <script>
          function doCountDown() {
            const cd = 60000; // 实际场景中，cd应该是从服务器传过来的时间间隔。
            countDown(cd, 100);
          }

          function countDown(cd, interval) {
            const target = document.getElementById('countDown');

            const start = new Date().getTime();

            let count = 0;
            const timer = function() {
              const now = new Date().getTime();
              const offset = now - start;
              if (offset >= cd) {
                target.innerText = '倒计时结束';
                clearTimeout(timer.toId);
              } else {
                target.innerText = `还剩${ms2Time(cd - offset)}`;

                const toOffset = offset - ++count * interval;
                const next = toOffset > interval ? 0 : interval - toOffset;
                timer.toId = setTimeout(timer, next);
              }
            };

            timer.toId = setTimeout(timer, interval);
          }

          function ms2Time(ms) {
            let val = ms;
            const sss = String(Math.floor(val % 1000)).padStart(3, '0');

            val = Math.floor(val / 1000);
            const s = Math.floor(val % 60);

            val = Math.floor(val / 60);
            const m = Math.floor(val % 60);

            val = Math.floor(val / 60);
            const h = Math.floor(val % 24);

            val = Math.floor(val / 24);
            return `${val}天 ${h}时 ${m}分 ${s}秒:${sss}`;
          }
        </script>

      </body>

    </html>
