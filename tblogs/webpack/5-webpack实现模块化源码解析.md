---
layout: post
title: webpack实现模块化源码解析
date: 2019.7.13
---

### `webpack`实现

`webpack`是使用`node`编写的项目构建工具，原生支持`CommonJS`规范。

`webpack`自身维护了一套模块化系统，兼容了`ES6 Module` `CommonJS` `AMD`等模块化规范。

> Demo

用于打包的源码清单：

```javascript
// ./src/a.js: 遵循ES6 Module规范, 打包的入口文件
import b from './b';
import { bVal } from './b';
import * as bObj from './b';

const aVal = 'this is a val';
const aVal2 = 'this is a val2';

export { b, bVal, bObj, aVal, aVal2 };

export default 'this is default a';
```

```javascript
// ./src/b.js
const bVal = 'this is b val';
export { bVal };

export default 'this is default b';
```

```javascript
// ./src/c.js: 遵循CommonJS规范，打包的入口文件
const d = require('./d');
const { dVal } = require('./d');

const c = 'this is cVal';

module.exports = {
  c,
  d,
  dVal,
};
```

```javascript
// ./src/d.js
const dVal = 'this is dVal';

exports.dVal = dVal;
```

`webpack`打包配置：

```javascript
// ./webpack.config.js
const path = require('path');

module.exports = {
  entry: {
    esModule: './a.js',
    cjsModule: './c.js',
  },
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: '[name].bundle.js',
  },
  mode: 'development',
};
```

> `webpack`打包后的源码解析

`webpack`原生支持`CommonJS`规范，并对其它规范进行兼容：从打包后，c.js 的模块加载函数中便可以看出。

打包后的源码清单如下，源码内容详见最后：

- dist
  - esModule.bundle.js
  - cjsModule.bundle.js

#### 源码骨架：以 esModule.bundle.js 为例

```javascript
//自执行函数
(function(modules) {
  //模块加载函数
  function __webpack_require__(moduleId) {
    /**
     * 检查模块是否已加载至缓存中，若有则直接返回。
     **/

    const moduleImportFunction = modules[moduleId];
    moduleImportFunction.call(args, __webpack_require__);

    /**
     * 返回模块
     **/
  }

  return __webpack_require__('./a.js'); //从根入口a.js开始加载模块。
})({
  './src/a.js': function aImportFuntion(args, __webpack_require__) {
    /**
     * a模块的具体加载逻辑
     **/

    __webpack_require__('./b.js'); //a中导入了b模块，因此需要在此加载b模块
  },
  './src/b.js': function bImportFunction() {
    /**
     * b模块的具体加载逻辑
     **/
  },
});
```

#### \_\_webpack_require\_\_

`webpack`为每一个解析的模块创建相应的`module`对象，其中，模块暴露(导出)给外界的 API(对象)被放置在`module.exports`中。

之后的文章中，统一将`module.exports`称之为模块的导出对象。

源码中，`ES6 Module`规范定义的模块被视为`harmony`(和谐的).

本文中，`non-harmony`仅指`CommonJS`规范定义的模块，其它规范不在考虑范围中。

```javascript
var installedModules = {}; //缓存已加载的模块

function __webpack_require__(moduleId) {
  // 若moduleId对应的模块已经被加载，则直接从缓存中读取。
  if (installedModules[moduleId]) {
    return installedModules[moduleId].exports; //返回模块的导出对象。
  }

  // 创建module, 用于存放加载后的结果。并将module被放置在缓存中。
  var module = (installedModules[moduleId] = {
    i: moduleId,
    l: false, //表示模块是否被加载完成，若加载完毕则置位true
    exports: {}, //用于存放模块的导出对象
  });

  // modules[moduleId]为具体的模块加载函数，调用该函数，执行加载逻辑。
  modules[moduleId].call(
    module.exports,
    module,
    module.exports,
    __webpack_require__,
  );

  // 模块加载完毕后，将l置位true
  module.l = true;

  // 返回模块的导出对象。
  return module.exports;
}
```

#### \_\_webpack_require\_\_.r

`webpack`遇到`ES6 Module`规范定义的模块后，会调用该方法，标记相应的模块是 ES6 模块。

```javascript
__webpack_require__.r = function(exports) {
  // 若支持Symbol, 则在导出对象中，增加Symbol属性，标记模块的Symbol是Module
  if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
    Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
  }

  // 导出对象中，增加__esModule属性，并置为true, 表明模块是ES6 Module规范定义的模块。
  Object.defineProperty(exports, '__esModule', { value: true });
};
```

#### \_\_webpack_require\_\_.d

`webpack`遇到`ES6 Module`规范定义的模块后，会调用相应方法，将导出的变量放置在`module.exports`中。

```javascript
// 判断对象是否原生拥有某属性
__webpack_require__.o = function(object, property) {
  return Object.prototype.hasOwnProperty.call(object, property);
};

// 向导出对象exports中新增枚举属性name, 该枚举属性的值可通过相应的getter获取。
__webpack_require__.d = function(exports, name, getter) {
  if (!__webpack_require__.o(exports, name)) {
    Object.defineProperty(exports, name, { enumerable: true, get: getter });
  }
};
```

#### \_\_webpack_require\_\_.n

模块导出对象中的默认值`default`的 getter.

`ES6 Module`规范中导出的默认值，被放置在`module.default`中。

`CommonJS`规范中，没有导出的默认值，默认值可被视为`module`自身。

```javascript
// getDefaultExport function for compatibility with non-harmony modules
__webpack_require__.n = function(module) {
  var getter =
    module && module.__esModule
      ? function getDefault() {
          return module['default']; //ES6 Module规范的默认值为module.default
        }
      : function getModuleExports() {
          return module; //CommonJS规范的默认值即为module本身
        };
  __webpack_require__.d(getter, 'a', getter);
  return getter;
};
```

#### \_\_webpack_require\_\_.t

未在源码中找到调用该函数的地方，可能是`webpack`的其它特性所使用的函数，待定。

```javascript
// create a fake namespace object
// mode & 1: value is a module id, require it
// mode & 2: merge all properties of value into the ns
// mode & 4: return value when already ns object
// mode & 8|1: behave like require
__webpack_require__.t = function(value, mode) {
  if (mode & 1) value = __webpack_require__(value);
  if (mode & 8) return value;
  if (mode & 4 && typeof value === 'object' && value && value.__esModule)
    return value;
  var ns = Object.create(null);
  __webpack_require__.r(ns);
  Object.defineProperty(ns, 'default', { enumerable: true, value: value });
  if (mode & 2 && typeof value != 'string')
    for (var key in value)
      __webpack_require__.d(
        ns,
        key,
        function(key) {
          return value[key];
        }.bind(null, key),
      );
  return ns;
};
```

#### 模块的具体加载逻辑，以 a.js 为例

```javascript
// module: 模块加载完毕之后的对象
// __webpack_exports__: 指向module.exports
// __webpack_require__
(function(module, __webpack_exports__, __webpack_require__) {
  'use strict';
  __webpack_require__.r(__webpack_exports__);

  // 在导出对象中增加属性aVal及相应getter
  __webpack_require__.d(__webpack_exports__, 'aVal', function() {
    return aVal;
  });

  __webpack_require__.d(__webpack_exports__, 'aVal2', function() {
    return aVal2;
  });

  // 通过__webpack_require__加载b
  var _b__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(
    /*! ./b */ './src/b.js',
  );

  // 在导出对象中增加b及相应getter, 指向b.js中的默认导出值default
  __webpack_require__.d(__webpack_exports__, 'b', function() {
    return _b__WEBPACK_IMPORTED_MODULE_0__['default'];
  });

  __webpack_require__.d(__webpack_exports__, 'bVal', function() {
    return _b__WEBPACK_IMPORTED_MODULE_0__['bVal'];
  });

  __webpack_require__.d(__webpack_exports__, 'bObj', function() {
    return _b__WEBPACK_IMPORTED_MODULE_0__;
  });

  var aVal = 'this is a val';
  var aVal2 = 'this is a val2';

  // 在导出对象中增加default, 指向a中的默认导出值。
  __webpack_exports__['default'] = 'this is default a';
});
```

#### 源码清单

> esModule.bundle.js(a.js b.js 打包生成的源码)

```javascript
// /dist/esModule.bundle.js
(function(modules) {
  // webpackBootstrap
  // The module cache
  var installedModules = {};

  // The require function
  function __webpack_require__(moduleId) {
    // Check if module is in cache
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    // Create a new module (and put it into the cache)
    var module = (installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {},
    });

    // Execute the module function
    modules[moduleId].call(
      module.exports,
      module,
      module.exports,
      __webpack_require__,
    );

    // Flag the module as loaded
    module.l = true;

    // Return the exports of the module
    return module.exports;
  }

  // expose the modules object (__webpack_modules__)
  __webpack_require__.m = modules;

  // expose the module cache
  __webpack_require__.c = installedModules;

  // define getter function for harmony exports
  __webpack_require__.d = function(exports, name, getter) {
    if (!__webpack_require__.o(exports, name)) {
      Object.defineProperty(exports, name, { enumerable: true, get: getter });
    }
  };

  // define __esModule on exports
  __webpack_require__.r = function(exports) {
    if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
      Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
    }
    Object.defineProperty(exports, '__esModule', { value: true });
  };

  // create a fake namespace object
  // mode & 1: value is a module id, require it
  // mode & 2: merge all properties of value into the ns
  // mode & 4: return value when already ns object
  // mode & 8|1: behave like require
  __webpack_require__.t = function(value, mode) {
    if (mode & 1) value = __webpack_require__(value);
    if (mode & 8) return value;
    if (mode & 4 && typeof value === 'object' && value && value.__esModule)
      return value;
    var ns = Object.create(null);
    __webpack_require__.r(ns);
    Object.defineProperty(ns, 'default', { enumerable: true, value: value });
    if (mode & 2 && typeof value != 'string')
      for (var key in value)
        __webpack_require__.d(
          ns,
          key,
          function(key) {
            return value[key];
          }.bind(null, key),
        );
    return ns;
  };

  // getDefaultExport function for compatibility with non-harmony modules
  __webpack_require__.n = function(module) {
    var getter =
      module && module.__esModule
        ? function getDefault() {
            return module['default'];
          }
        : function getModuleExports() {
            return module;
          };
    __webpack_require__.d(getter, 'a', getter);
    return getter;
  };

  // Object.prototype.hasOwnProperty.call
  __webpack_require__.o = function(object, property) {
    return Object.prototype.hasOwnProperty.call(object, property);
  };

  // __webpack_public_path__
  __webpack_require__.p = '';

  // Load entry module and return exports
  return __webpack_require__((__webpack_require__.s = './src/a.js'));
})({
  './src/a.js': function(module, __webpack_exports__, __webpack_require__) {
    'use strict';
    __webpack_require__.r(__webpack_exports__);

    /* harmony export (binding) */
    __webpack_require__.d(__webpack_exports__, 'aVal', function() {
      return aVal;
    });

    /* harmony export (binding) */
    __webpack_require__.d(__webpack_exports__, 'aVal2', function() {
      return aVal2;
    });

    /* harmony import */
    var _b__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(
      /*! ./b */ './src/b.js',
    );

    /* harmony reexport (safe) */
    __webpack_require__.d(__webpack_exports__, 'b', function() {
      return _b__WEBPACK_IMPORTED_MODULE_0__['default'];
    });

    /* harmony reexport (safe) */
    __webpack_require__.d(__webpack_exports__, 'bVal', function() {
      return _b__WEBPACK_IMPORTED_MODULE_0__['bVal'];
    });

    /* harmony reexport (module object) */
    __webpack_require__.d(__webpack_exports__, 'bObj', function() {
      return _b__WEBPACK_IMPORTED_MODULE_0__;
    });

    var aVal = 'this is a val';
    var aVal2 = 'this is a val2';

    /* harmony default export */
    __webpack_exports__['default'] = 'this is default a';
  },

  './src/b.js': function(module, __webpack_exports__, __webpack_require__) {
    'use strict';
    __webpack_require__.r(__webpack_exports__);
    /* harmony export (binding) */
    __webpack_require__.d(__webpack_exports__, 'bVal', function() {
      return bVal;
    });
    var bVal = 'this is b val';

    /* harmony default export */
    __webpack_exports__['default'] = 'this is default b';
  },
});
```

> cjsModule.bundle.js(c.js d.js 打包生成的源码)

```javascript
// /dist/cjsModule.bundle.js
(function(modules) {
  // webpackBootstrap
  // The module cache
  var installedModules = {};

  // The require function
  function __webpack_require__(moduleId) {
    // Check if module is in cache
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    // Create a new module (and put it into the cache)
    var module = (installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {},
    });

    // Execute the module function
    modules[moduleId].call(
      module.exports,
      module,
      module.exports,
      __webpack_require__,
    );

    // Flag the module as loaded
    module.l = true;

    // Return the exports of the module
    return module.exports;
  }

  // expose the modules object (__webpack_modules__)
  __webpack_require__.m = modules;

  // expose the module cache
  __webpack_require__.c = installedModules;

  // define getter function for harmony exports
  __webpack_require__.d = function(exports, name, getter) {
    if (!__webpack_require__.o(exports, name)) {
      Object.defineProperty(exports, name, { enumerable: true, get: getter });
    }
  };

  // define __esModule on exports
  __webpack_require__.r = function(exports) {
    if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
      Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
    }
    Object.defineProperty(exports, '__esModule', { value: true });
  };

  // create a fake namespace object
  // mode & 1: value is a module id, require it
  // mode & 2: merge all properties of value into the ns
  // mode & 4: return value when already ns object
  // mode & 8|1: behave like require
  __webpack_require__.t = function(value, mode) {
    if (mode & 1) value = __webpack_require__(value);
    if (mode & 8) return value;
    if (mode & 4 && typeof value === 'object' && value && value.__esModule)
      return value;
    var ns = Object.create(null);
    __webpack_require__.r(ns);
    Object.defineProperty(ns, 'default', { enumerable: true, value: value });
    if (mode & 2 && typeof value != 'string')
      for (var key in value)
        __webpack_require__.d(
          ns,
          key,
          function(key) {
            return value[key];
          }.bind(null, key),
        );
    return ns;
  };

  // getDefaultExport function for compatibility with non-harmony modules
  __webpack_require__.n = function(module) {
    var getter =
      module && module.__esModule
        ? function getDefault() {
            return module['default'];
          }
        : function getModuleExports() {
            return module;
          };
    __webpack_require__.d(getter, 'a', getter);
    return getter;
  };

  // Object.prototype.hasOwnProperty.call
  __webpack_require__.o = function(object, property) {
    return Object.prototype.hasOwnProperty.call(object, property);
  };

  // __webpack_public_path__
  __webpack_require__.p = '';

  // Load entry module and return exports
  return __webpack_require__((__webpack_require__.s = './c.js'));
})({
  './src/c.js': function(module, exports, __webpack_require__) {
    var d = __webpack_require__(/*! ./d */ './src/d.js');

    var _require = __webpack_require__(/*! ./d */ './src/d.js'),
      dVal = _require.dVal;

    var c = 'this is cVal';
    module.exports = {
      c: c,
      d: d,
      dVal: dVal,
    };
  },

  './src/d.js': function(module, exports) {
    var dVal = 'this is dVal';
    exports.dVal = dVal;
  },
});
```

> [返回]({{site.baseurl}}/webpack总结)
