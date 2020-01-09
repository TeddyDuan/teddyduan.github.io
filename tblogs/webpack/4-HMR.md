---
layout: post
title: 模块热替换(hot module replacement)
date: 2019.7.8
---

- 当某个模块代码发生修改时，无需刷新整个页面，只替换相应模块代码。
- 开发过程使用，提高开发效率。无需在生产环境进行。

要实现`HMR`, 需要完成以下两步操作：

## 1. 开启开关

### webpack-dev-server

> 默认支持`live reloading`, 即当`webpack-dev-server`检测到代码修改后，刷新整个页面来应用变化。

> `live reloading`不是`HMR`.

以下配置开启`HMR`

```javascript
return {
  devServer: {
    hot: true,
  },
  plugins: [new webpack.HotModuleReplacementPlugin()],
};
```

### express: 集成 webpack-hot-middleware 中间件

## 2. 为需要支持 HMR 的模块，实现相应的接口：

- 实现接收到更新后，处理相应更新的业务逻辑。

```javascript
if (module.hot) {
  module.hot.accept('./someModule.js', () => {
    console.log('updated detected');
    //移除旧有的逻辑

    //增加新的逻辑
  });
}
```

## 原生实现了 HMR 接口的工具

- style-loader
- vue-loader
- react-hot-loader
- angular-hrm
- 其它...

> [返回]({{site.baseurl}}/webpack总结)
