---
layout: post
title: toString&valueOf
date: 2020.2.14
---

> 本文涉及的包装类型为 原始类型包装类 Boolean/Number/String/Symbol 和 引用类型包装类 Object/Array/RegExp/Date/Function

#### toString: 返回对象的字符串值

1. 各包装类 prototype 上均定义了 toString 函数。 各实例调用 toString 等价于 String(x);
2. Date 实例调用 toString, 返回标准的时间字符串，如`Mon Feb 17 2020 01:23:44 GMT+0800 (China Standard Time)`
3. 数组实例调用 toString, 若为空则返回'', 否则返回 Array.prototype.join.call(arr, ',')
4. 对象实例调用 toString, 也即 Object.prototype.toString.call({})

#### valueOf: 返回对象的原始值

1. Array/RegExp/Function 的 prototype 上没有定义 valueOf 函数; 其实例调用 valueOf 时，会沿着原型链一直寻找，并最终调用 Object.prototype.valueOf 函数。
2. Date 实例调用 valueOf, 返回毫秒值。
3. 其余实例调用 valueOf, 返回实例本身的值。

## 转换为原始类型

### 类似于 String()的操作(如 alert 等)

1. 调用对象的 toString 函数(沿着作用域链)，若结果为原始值，则将原始值转换为 string 并返回。
2. 调用对象的 valueOf 函数(沿着作用域链)，若结果为原始值，则将原始值转换为 string 并返回。
3. 抛出 TypeError.

以下为测试代码，可以验证上述逻辑。

```javascript
const { valueOf } = Object.prototype;

delete Array.prototype.toString;

Object.prototype.toString = function() {
  console.log('toString');
  return {};
};

Object.prototype.valueOf = function() {
  console.log('valueOf');
  return valueOf.call(this);
};

const a = [],
  o = {};
try {
  o[a] = 1;
} catch (e) {
  console.log(e); // toString, valueOf, TypeError.
}
```

### 类似于 Number()的操作(如+运算等)

1. 调用对象的 valueOf 函数(沿着作用域链)，若结果为原始值，则将原始值转换为 number 并返回。
2. 调用对象的 toString 函数(沿着作用域链), 若结果为原始值，则将原始值转换为 number 并返回。
3. 抛出 TypeError.

以下为测试代码，可以验证上述逻辑。

```javascript
delete Array.prototype.toString;

Object.prototype.toString = function() {
  console.log('toString');
  return {};
};

Object.prototype.valueOf = function() {
  console.log('valueOf');
  return [];
};

const a = [1, 2, 3];

try {
  console.log(-a);
} catch (e) {
  console.log(e);
}
```

### 各类型转换为 boolean/number/string/symbol

<a href="../类型转换.html" target="__blank">转换结果详见</a>

> [返回]({{site.baseurl}}/ES总结汇总)
