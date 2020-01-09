---
layout: post
title: eslint总结
date: 2019.7.15
---

## 优先级规则

1. 代码块注释
   1. /\* eslint-disable \*/ 和 /\* eslint-enable \*/
   2. /\* global \*/
   3. /\* eslint \*/ 规则定义
   4. /\* eslint-env \*/ 环境变量定义
2. 命令行选项
   1. --global
   2. --rule
   3. --env
   4. --config, -c
3. 配置文件

   1. 被检测文件同路径下的配置文件
   2. 被检测文件父路径下的配置文件，直至到根或配置文件的 root 为 true
   3. ~/.eslintrc.[ext]

> 其中，配置文件只能为以下几种格式中的一种，且采用的文件优先级如下：

1. .eslintrc.js
2. .eslintrc.yaml
3. .eslintrc.yml
4. .eslintrc.json
5. .package.json

## 配置项

### parserOptions

```javascript
return {
  parserOptions: {
    ecmaVersion: 2015, //启用es6语法，但没有启动es6的全局变量如Set/Map等。可以是6/7/8...或年份2015/2016/2017...
    sourceType: 'module', //代码是否为es module
    ecmaFeatures: {
      globalReturn: false, //是否允许在全局作用域使用return语句
      impliedStrict: false, //是否启用全局严格模式
      jsx: true, //是否启用jsx
    },
  },
};
```

### env/预定义全局变量

> 定义了一组预定义的全局变量

> `parserOptions.ecmaVersion: X`和`env.es6: true`的区别：

- 前者只启用 esX 语法，不支持 es6 下新的全局变量(如 Set/Map 等)。
- 后者在启用 es6 的全局变量的同时，自动将前者设置为 6(而非 es7/8/9...)

```javascript
//注释方式
/* eslint-env: node, mocha */

//配置文件方式
return {
  plugins: ['a-plugin'],
  env: {
    browser: true,
    es6: true,
    node: true,
    'a-plugin/aPluginVar1': true, //插件a-plugin中定义的aPluginVar1
    //...其它配置
  },
};
```

### parser/解析器

- Espree: eslint 默认解析器
- 第三方
  - Babel-Eslint
  - @typescript-eslint/parser

```javascript
return {
  parser: 'babel-eslint',
};
```

### plugins/插件

```javascript
return {
  plugins: [
    'plugin1', //'eslint-plugin-'前缀可忽略
    'eslint-plugin-plugin2',
  ],
};
```

### processor/处理器

```javascript
return {
  plugins: ['a-plugin'],
  processor: 'a-plugin/processor-1', //启用插件a-plugin中的处理器processor-1
};
```

```javascript
return {
  plugins: ['a-plugin'],
  override: {
    files: ['*.md'],
    processor: 'a-plugin/processor-1',
  },
};
```

### globals/自定义全局变量

```javascript
//注释方式
/* global globalA:writable globalB:readonly */

//配置文件方式
return {
  globals: {
    globalA: 'writable',
    globalB: 'readonly',
    Promise: 'off', //禁用Promise
  },
};
```

### rules/规则

#### 规则分类

- 0: 'off'
- 1: 'warn'
- 2: 'error'

#### 启用规则

```javascript
//注释方式
/* eslint eqeqeq: 'off', curly: 'error' */
/* eslint eqeqeq: 0, curly: 2 */
/* eslint quotes: ['error', 'double'], curly: 2 */
/* eslint 'plugin1/rule1': 'error' */

//配置文件方式
return {
  rules: {
    eqeqeq: 'off',
    curly: 'error',
    quotes: ['error', 'double'],
    'plugin1/rule1': 'error',
  },
};
```

#### 禁用规则

```javascript
/* eslint-disable 禁用所有规则 */

alert('foo');
console.log('bar');
/* eslint-enable */

/* eslint-disable no-alert, no-console 禁用指定规则 */
alert('foo');
console.log('bar');
/* eslint-enable no-alert, no-console */

//将注释放在顶部，在整个文件中禁用相应规则。

alert('foo'); // eslint-disable-line no-alert, example/rule-name /* eslint-disable-line no-alert, example/rule-name */

alert('foo'); // eslint-disable-line  /* eslint-disable-line */

// eslint-disable-next-line no-alert, quotes, semi
/* eslint-disable-next-line no-alert, quotes, semi */
alert('foo');
```

#### 批量禁用规则

```javascript
return {
  overrides: [
    {
      files: ['*-test.js', '*.spec.js'],
      rules: {
        'no-unused-expressions': 'off',
      },
    },
  ],
};
```

> [返回]({{site.baseurl}}/webpack总结)
