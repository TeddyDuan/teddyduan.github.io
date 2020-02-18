---
layout: post
title: 函数基础总结&prototype&constructor
date: 2020.2.16
---

### 基础总结

1. 函数属性
   1. length: 函数定义
   2. [prototype](#prototype)
2. 函数内部属性
   1. this
   2. arguments
      1. 类数组对象
      2. arguments.length: 实际传递参数的个数
      3. arguments.callee: 指向拥有 arguments 的函数
         1. arguments.callee.caller: ES5, 调用当前函数的函数

### <a name="prototype">prototype & constructor</a>

> 以下内容涉及的包装类型为 原始类型包装类 Boolean/Number/String/Symbol 和 引用类型包装类 Object/Array/RegExp/Date/Function

1. T.\_\_proto\_\_

   ```javascript
   const { toString } = Object.prototype;
   // 任何包装类都由Function构造函数创建。
   T.__proto__ === Function.prototype;
   toString.call(T) === '[object Function]';
   ```

2. T.prototype.\_\_proto\_\_

   ```javascript
   const { toString } = Object.prototype;

   Object.prototype.__proto__ === null;

   // 除Object之外的其它包装类T
   T.prototype.__proto__ === Object.prototype;

   // 注意：正常情况下，toString.call(val)
   // val.__proto__对应的原型V输出[object V]

   // 但是，toString.call(T.prototype)
   // 不一定会根据T.prototype.__proto__输出[object Object]
   // 只有Date/RegExp会输出[object Object], 其它会输出[object T].
   [
     Boolean,
     Number,
     String,
     Symbol,
     Object,
     Array,
     RegExp,
     Date,
     Function,
   ].forEach((Type) => {
     console.log(toString.call(Type.prototype));
   });
   ```

3. constructor

   1. t.constructor = T; T.prototype.constructor = T;

      ```javascript
      function T() {}
      const t = new T();
      console.log(t.constructor === T);
      console.log(T.prototype.constructor === T);
      ```

   2. T.constructor === Function
      ```javascript
      [
        Boolean,
        Number,
        String,
        Symbol,
        Object,
        Array,
        RegExp,
        Date,
        Function,
      ].forEach((Type) => {
        console.log(
          Type.prototype.constructor === Type,
          Type.constructor === Function,
        );
      });
      ```
