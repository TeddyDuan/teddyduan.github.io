---
layout: post
title: 代码切分 & TreeShaking
date: 2019.7.7
---

> 此文章总结的内容均基于`webpack 4.35.3`进行，`webpack`后续可能会在各别配置/插件上进行相应的更新。

# Code splitting/代码分割

### Multi entries ports

简单粗暴的拆分为多个入口

### SplitChunkPlugin: 拆分公共代码

最基础的拆分配置

```javascript
{
  optimization: {
    splitChunks: {
      chunks: 'all';
    }
  }
}
```

### Dynamic Imports

- `Magic Comments`/`webpack`魔法注释

  - webpackChunkName

    - 动态引入的包拆分至[name].bundle.js
      - 如不指定，则默认被拆分至[id].bundle.js 中

    ```javascript
    import(/* webpackChunkName: "lodash" */ 'lodash')
      .then((module) => {})
      .catch((e) => {});
    ```

  - webpackPrefetch

    - 浏览器会在当前页面加载完毕后，闲置下来时加载`预请求的js`

    ```javascript
    import(/* webpackPrefetch: true */ 'LoginModal');
    //打包后的html中会追加<link rel="prefetch" href="login-modal-chunk.js">
    ```

  - webpackPreload

    - 浏览器同步加载当前页面和需要`预加载的js`

    ```javascript
    import(/* webpackPrefetch: true */ 'lodash');
    //打包后的html中会追加<link rel="preload" href="lodash-chunk.js">
    ```

- Lazy Loading/懒加载
  增加用户的交互行为，如点击事件。当用户触发事件时，再加载相应代码，以此减少应用初始的加载量。

> 动态引用虽然已经拆分了代码，且 webpack 尽可能在脚本运行时再进行加载。

> 但是，每次请求页面时，无论多么的延迟，代码仍然、且最终一定会被加载。

> 实际需求里，我们可以增加一些交互条件，如用户点击某个按钮时，才加载相应的代码

# Tree Shaking

在编译阶段，根据各模块间的依赖关系及 API 调用情况，动态去除`dead code`(未被使用的 API), 从而精简代码。

实现`Tree Shaking`需要完成以下四步操作：

- 使用`静态`模块化依赖方案`ES Module`
  - 动态模块化如`CommonJS` `es dynamic import`不支持 tree shaking.
  - webpack 会进行`未使用导出`检测，并对未使用的导出标记注释`unused harmony export`
- 保证任何处理 js 的 loader 编译器不会将`ES Module`转换为`CommonJS`等其它模块化方案
  - 如，禁止`babel-loader`进行模块化转换。
- 通过`sideEffects`标记非`ES Module`的文件。
- 开启`production`模式(默认使用`TerserPlugin`来压缩代码，并去除 unused 标记注释的代码)。

# production: 生产环境常用优化策略总结(待完善)

> [返回]({{site.baseurl}}/webpack总结)
