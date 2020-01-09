---
layout: post
title: babel实现模块化兼容的原理
date: 2019.7.4
---

> `javascript` `主流`的模块化规范有：`ES6 Module` `CommonJS`

- `ES6 Module`
  - 官方标准规范
  - 在此之前有`CommonJS` `AMD` `CMD`等第三方规范推出。
- `CommonJS`
  - 第三方规范
  - 主要被`node`采用

## `ES6 Module`与`CommonJS`规范共存

### `babel`实现

#### ES6 导出

```javascript
//ES6 export
export default 123;
export const a = 1;

const b = 2,
  c = 3;
export { b, c };
```

`babel`上述代码转化为`CommonJS`规范：

```javascript
Object.defineProperty(exports, '__esModule', {
  value: true,
}); //标记为ES6模块

exports['default'] = exports.a = exports.b = exports.c = void 0;

const _default = 123;
exports['default'] = _default;

const a = 1;
exports.a = a;

const b = 2;
exports.b = b;

const c = 3;
exports.c = c;

return exports;
```

#### ES6 导入 default

```javascript
//导入默认值
import a from './a.js';
```

`babel`转化思路

```javascript
// CommonJS require导入源模块所有暴露对象，而不是默认值。
// 为兼容ES6 Module, 需要转换为require().default
// 为兼容CommonJS, require导入的对象需被default包裹。

function _interopRequireDefault(obj) {
  return obj && obj.__esModule ? obj : { default: obj }; //CommonJS模块的导出对象用default进行包裹。
}

return _interopRequireDefault(require('./a.js')).default;
```

#### ES6 导入整体

```javascript
//导入源模块默认值&所有暴露的对象
import * as a from './a.js';
```

`babel`转化思路

```javascript
function _interopRequireWildcard(obj) {
  if (obj && obj.__esModule) {
    return obj;
  } else {
    var newObj = {};
    if (obj != null) {
      for (var key in obj) {
        if (Object.prototype.hasOwnProperty.call(obj, key)) {
          var desc =
            Object.defineProperty && Object.getOwnPropertyDescriptor
              ? Object.getOwnPropertyDescriptor(obj, key)
              : {};
          if (desc.get || desc.set) {
            Object.defineProperty(newObj, key, desc);
          } else {
            newObj[key] = obj[key];
          }
        }
      }
    }
    newObj['default'] = obj; //CommonJS导出的变量，缺少default属性，需增加。
    return newObj;
  }
}

return _interopRequireWildcard(require('./a.js'));
```

#### ES6 导入部分变量

```javascript
//es6导入部分变量
import { valA, valB } from './a.js';
```

`babel`转化思路

```javascript
const _source = require('./a.js');

Object.defineProperty(exports, 'valA', {
  enumerable: true,
  get: function get() {
    return _source.valA;
  },
});

Object.defineProperty(exports, 'valB', {
  enumerable: true,
  get: function get() {
    return _source.valB;
  },
});
```

基于`babel`的以上机制，可直接使用`ES6 Module语法`导入`CommonJS`模块，反之亦可。

### `webpack`实现

- 见[webpack 实现模块化源码解析](../5-webpack实现模块化源码解析)

> [返回]({{site.baseurl}}/webpack总结)
