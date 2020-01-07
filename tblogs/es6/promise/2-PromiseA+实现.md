---
layout: post
title: 源码实现Promise/A+规范
date: 2020.1.6
---

> 源码实现如下，在[此工程](https://github.com/TeddyDuan/ted-js/blob/master/_3pro/_4promise/TedPromise.js)维护。

```javascript
const { toString } = Object.prototype;
const isFunction = (fn) => toString.call(fn) === '[object Function]';
const isObject = (obj) => toString.call(obj) === '[object Object]';

const PENDING = Symbol('pending');
const FULFILLED = Symbol('fulfilled');
const REJECTED = Symbol('rejected');

/**
 * 1. resolve/reject用于更新promise的状态和值(value/reason), 且只能更新一次。
 * 2. 因为使用了resolve.bind/reject.bind, 所以不能写成箭头函数的形式。
 * 3. nextOnXXX在构造函数执行时不会调用，只会在then时调用，将当前promise的值传递给各后继promise.
 */
const resolve = function(value) {
  if (this.status === PENDING) {
    this.status = FULFILLED;
    this.value = value;

    // 当前宏任务中，可能出现当前promise0状态为pending的情况。
    // 此时多次调用then会压入promise0的栈，栈内各函数返回各自的promiseX(详见then方法实现)
    // 当promise0状态更新为fulfilled时，需执行后继promiseX，将promise0的值传递至相应promiseX中的onFulFilled.
    this.nextOnFulfilleds.forEach((fn) => fn());
  }
};

const reject = function(reason) {
  if (this.status === PENDING) {
    this.status = REJECTED;
    this.reason = reason;

    // 当前宏任务中，可能出现当前promise0状态为pending的情况。
    // 此时多次调用then会压入promise0的栈，栈内各函数返回各自的promiseX(详见then方法实现)
    // 当promise0状态更新为rejected时，需执行后继promiseX，将promise0的失败原因传递至相应promiseX中的onRejected.
    this.nextOnRejecteds.forEach((fn) => fn());
  }
};

/**
 * 将onFulfilled/onRejected的结果传递至promise2并更新其状态。
 * @param {*} promise2
 * @param {*} x
 * @param {*} resolve
 * @param {*} reject
 */
const resolvePromise = function(promise2, x, resolve, reject) {
  if (promise2 === x) {
    throw new TypeError('重复promise调用，死循环');
  }

  if (isFunction(x) || isObject(x)) {
    let called = false;
    try {
      const { then } = x;
      if (isFunction(then)) {
        /**
         * thenable的具体实现中, resolvePromise/rejectPromise可能出现多次调用的情况。
         * 1. 需要用called标记，保证所有的resolvePromise / rejectPromise调用里，真正得到执行的只有一次.
         * 2. thenable对象可能是一个异常对象，因此也需要被捕获。
         */
        then.call(
          x,
          (y) => {
            if (called) return;
            called = true;
            resolvePromise(promise2, y, resolve, reject);
          },
          (r) => {
            if (called) return;
            called = true;
            reject(r);
          },
        );
      } else {
        resolve(x);
      }
    } catch (e) {
      if (called) return;
      called = true;
      reject(e);
    }
  } else {
    resolve(x);
  }
};

class TedPromise {
  constructor(executor) {
    const self = this;
    self.status = PENDING;
    self.nextOnFulfilleds = [];
    self.nextOnRejecteds = [];

    /**
     * 1. executor为用户自定义函数，执行时需要try...catch...对可能出现的异常进行处理。
     * 2. then的参数onFulfilled & onRejected同理。
     * 3. onFulfilled & onRejected返回的thenable同理。
     */
    try {
      executor(resolve.bind(self), reject.bind(self));
    } catch (e) {
      reject.call(self, e);
    }
  }

  then(onFulfilled, onRejected) {
    const self = this;
    /**
     * onFulfilled/onRejected不符合要求时，覆盖为缺省函数。
     */
    if (!isFunction(onFulfilled)) onFulfilled = (value) => value;

    if (!isFunction(onRejected))
      onRejected = (reason) => {
        throw reason;
      };

    /**
     * 1. then返回新的promise2.
     * 2. 目标:
     *    2.1 根据当前promise的目标状态，将结果传递至相应的onFulfilled/onRejected.
     *    2.2 onFulfilled/onRejected的[最终]结果为promise2的最终结果.
     *    2.3 可能出现onFulfilled/onRejected的返回结果为Promise实例/thenable对象的场景;
     *        此时需[持续]执行新的onFulfilled/onRejected, 直至得到最终结果并传递至promise2.
     *    2.4 onFulfilled/onRejected必须为宏任务/微任务, 在当前执行栈的宏任务执行完毕后，再进行执行。
     */
    const promise2 = new TedPromise((resolve, reject) => {
      const doOnFulfilled = () =>
        setTimeout(() => {
          try {
            const x = onFulfilled(self.value);
            resolvePromise(promise2, x, resolve, reject); // 将onFulfilled结果传递至promise2
          } catch (e) {
            reject(e);
          }
        });

      const doOnRejected = () =>
        setTimeout(() => {
          try {
            const x = onRejected(self.reason);
            resolvePromise(promise2, x, resolve, reject); // 将onRejected结果传递至promise2; 其结果可能为非Error对象
          } catch (e) {
            reject(e);
          }
        });

      if (self.status === FULFILLED) {
        doOnFulfilled();
      } else if (self.status === REJECTED) {
        doOnRejected();
      } else if (self.status === PENDING) {
        /**
         * 多重then调用时，可能出现前序promiseX的状态仍然是PENDING的场景, 此时需要:
         * 1. 压入相应的onFulfilleds & onRejecteds队列。
         * 2. 在前序promiseX执行完毕后, 依次执行队列里的函数, 并将结果传递至promise2.
         */
        self.nextOnFulfilleds.push(doOnFulfilled);
        self.nextOnRejecteds.push(doOnRejected);
      }
    });
    return promise2;
  }

  catch(onRejected) {
    return this.then(undefined, onRejected);
  }

  finally(cb) {
    return this.then(
      (value) => TedPromise.resolve(cb()).then(() => value),
      (reason) =>
        TedPromise.resolve(cb()).then(() => {
          throw reason;
        }),
    );
  }

  static resolve(value) {
    if (value instanceof TedPromise) return value; // 若参数为promise, 不做处理

    return new TedPromise((resolve, reject) => {
      try {
        let thenable = false;
        if (value) {
          // 考虑thenable的场景
          const { then } = value;
          if (isFunction(then)) {
            thenable = true;
            /**
             * 原生Promise: thenable在下一个宏任务执行时开始执行，晚于当前执行栈中的各promise微任务。
             */
            setTimeout(() => then(resolve, reject));
          }
        }

        if (!thenable) {
          resolve(value);
        }
      } catch (e) {
        reject(e);
      }
    });
  }

  static reject(reason) {
    return new TedPromise((resolve, reject) => {
      reject(reason);
    });
  }

  static all(promises) {
    return new TedPromise((resolve, reject) => {
      const results = [];
      if (!promises) resolve(results);

      try {
        const it = promises[Symbol.iterator]();

        for (let i = -1, next = it.next(); !next.done; next = it.next()) {
          i++;
          Promise.resolve(next.value).then(
            /* eslint-disable */
            (rt) => {
              results[i] = rt;
            },
            /* eslint-enable */
            (reason) => reject(reason),
          );
        }

        resolve(results);
      } catch (e) {
        reject(e);
      }
    });
  }

  static race(promises) {
    return new TedPromise((resolve, reject) => {
      if (!promises) resolve();

      try {
        const it = promises[Symbol.iterator]();
        let next = it.next();
        while (!next.done) {
          Promise.resolve(next.value).then(
            (data) => resolve(data),
            (reason) => reject(reason),
          );
          next = it.next();
        }
      } catch (e) {
        reject(e);
      }
    });
  }
}

/**
 * 以下扩展用于Promise/A+规范测试。
 * 对应的测试脚本在promises-aplus-tests中。
 * 测试命令: promises-aplus-tests yourPromise.js
 */
/* TedPromise.defer = TedPromise.deferred = function() {
  const dfd = {};
  dfd.promise = new TedPromise((resolve, reject) => {
    dfd.resolve = resolve;
    dfd.reject = reject;
  });
  return dfd;
}; */

module.exports = TedPromise;
```

附: [Promise/A+规范说明](https://promisesaplus.com/)

> [返回]({{site.baseurl}}/Promise总结)
