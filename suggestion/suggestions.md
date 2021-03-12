# 开发建议

1. 虽然此方案将完整的 vue runtime 包含进来了，但必然存在一些无法直接适配的接口，比如 getBoundingClientRect，一部分会通过 dom/bom 扩展 api 间接实现，一部分则完全无法支持。查看 dom/bom 扩展 api 文档。

2. 可能存在部分逻辑在 web 端和小程序端需要使用不同的实现，该部分代码可以抽离成一个单独的模块或者插件，暴露接口给业务端代码使用。在模块内可以使用上述提到的 process.env.isMiniprogram 环境变量进行判断区分当前运行环境。比如上述提到的 actionSheet 实现就可以抽离成一个 vue 插件实现。

>PS：注意这里使用 process.env.isMiniprogram 环境变量时尽量不要加其他动态条件，以方便 webpack 编译时剪除死代码，比如 if (false) { console.log('xxxx') } 就属于死代码

```js
// 正确使用方式
if (!process.env.isMiniprogram) {
  // web 端
  if (isIPhone) {
    // do something
  }
}

// 错误使用方式
if (!process.env.isMiniprogram && isIPhone) {
  // web 端
  // do something
}
```

3. 如果需要使用第三方库，尽量选择使用轻量的库，以缩减构建出来的代码体积。
4. vue 组件命名尽量不要和小程序内置组件同名。
5. 避免使用 id 选择器、属性选择器，尽量少用标签选择器和 * 选择器，尽可能使用 class 选择器代替。
6. 为了确保模板解析不出问题，标签上布尔值属性建议使用 = 号赋值的写法，如下例子所示：

```
<input type="checkbox" checked="checked" />
<input type="checkbox" :checked="{{true}}" />
```