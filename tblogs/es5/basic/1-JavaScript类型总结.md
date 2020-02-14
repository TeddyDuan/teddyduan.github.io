---
layout: post
title: JavaScript类型总结
date: 2020.2.14
---

## 类型判断

#### 1. 原始类型: undefined, null, boolean, number, string, symbol

- boolean/number/string/symbol 对应有相应的包装类 Boolean/Number/String/Symbol.

- 除了 Symbol 之外，其余包装类 T 均可以通过构造函数 new 的方式创建实例对象.

- 普通函数调用方式 T()返回相应的基础类型。

  ```javascript
  const t = new T();
  typeof t; // object
  t instancof T; // true
  ```

> **除了 null 之外，其余的基础类型均可以通过 typeof 判断.**

```javascript
// typeof null返回object, 其余均返回对应的基础类型。
const raws = [undefined, null, false, 0, '', Symbol('a')];
raws.forEach((raw) => {
  console.log(raw, typeof raw);
});
```

#### 2. 引用类型: Object/Function/Array/Date/RegExp

- 相应的包装类 T 均为函数

- <a href="../引用类型包装类typeof对照表.html" target="__blank">typeof T & typeof new T() & typeof T()对照表</a>

> **通过调用构造函数(new)方式创建的实例对象，需要通过 instanceof 来判断类型，引用类型的包装类也不例外。**

### 3. **类型判断最佳实践:Object.prototype.toString.call**

> [返回]({{site.baseurl}}/ES总结汇总)
