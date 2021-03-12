# 代码优化

代码优化主要在于代码体积的精简和 dom 树的精简：

### 1、代码体积精简

在编译到小程序代码的时候，会将整个 `vue runtime` 和一些 vue 特性插件（如 vue-router、vuex 等）给打包进来，这样会导致代码包比较庞大，而这些代码是无法去除的，因此得从业务代码上着手进行一些缩减。业务上存在一些代码可以用小程序接口替代，这部分是完全不需要打包进来的，因此可以使用一个行内 loader 和环境变量来进行代码的去除，简单做法如下：

```
npm install --save-dev reduce-loader
```

```js
import Vue from 'vue'
import ActionSheet from 'reduce-loader!./action-sheet' // 使用行内 loader，剔除 action-sheet 文件的引入

// 通过注入的环境变量判断代码运行环境，进而执行不同的逻辑
if (!process.env.isMiniprogram) {
  // web 端
  ActionSheet.show([1, 2, 3], success)
} else {
  // 小程序端
  wx.showActionSheet({
      itemList: [1, 2, 3],
      success,
  })
}
```

### 2、dom 树的精简

对于一些站点会使用响应式设计，即 pc 端和 h5 端会共用一套代码，通常 pc 端很多节点在 h5 端是不需要展示出来的，这就需要在样式上对节点设置 display: none，而这些节点仍旧存在于 dom 树上，只是不渲染在视图上。如果这套代码直接转成小程序代码，也必定会创建一些无需展示的 dom 节点，这些节点本身是可以直接剔除。

因此可以使用另外一个 loader 对这些节点进行删减，在 webpack 配置中 vue loader 执行之前再添加一个 `vue-improve-loader`：

```
npm install --save-dev vue-improve-loader
```

```js
{
  test: /\.vue$/,
  use: [
    {
      loader: 'vue-loader',
      options: {
        compilerOptions: {
          preserveWhitespace: false
        }
      }
    },
    'vue-improve-loader',
  ]
},
```

然后在 vue 文件中给要剔除的节点加上 check-reduce 属性：

```html
<!-- 删减前代码 -->
<template>
  <div>
    <span>some text</span>
    <a check-reduce>
      <span>some text other</span>
    </a>
  </div>
</template>
```

因为 web 端代码构建和小程序端代码构建走不同的配置，所以 web 端代码会忽略这个属性，而小程序端代码则会删减掉带这个属性的节点。以下便是会输出给 vue-loader 的代码，从构建层面上剔除掉不需要的节点。

```html
<!-- 删减后代码 -->
<template>
  <div>
    <span>some text</span>
  </div>
</template>
```

>PS：vue-improve-loader 必须在 vue-loader 之前执行，这样 vue-loader 才会接收到被删减后的代码。

### 小程序扩展库

小程序支持了扩展库功能，使用小程序扩展库可以不占用小程序包体积。扩展库目前已内置了 kbone 的 miniprogram-render 和 miniprogram-element 核心依赖库，在 mp-webpack-plugin 配置中补充如下配置即可使用：

```js
module.exports = {
  generate: {
    autoBuildNpm: false,
  },
  appExtraConfig: {
    useExtendedLib: {
        kbone: true,
    },
  },
  // ... other options
}
```

>PS：如果使用扩展库，需要将 generate.autoBuildNpm 置为 false，这两个配置暂不支持同时使用。

>PS：因为近期 kbone 迭代较快，扩展库的版本会稍微落后于 npm 上最新版本，使用时敬请注意。