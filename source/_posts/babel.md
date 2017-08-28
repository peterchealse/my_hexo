---
title: 关于Babel与构建优化
date: 2017-08-28 15:38:15
tags: [webpack, babel, 工程化]
---

# 1.preset-*
### [官方preset](https://babeljs.io/docs/plugins/#presets-official-presets)
举例：preset-es2015将ES6编译成ES5，preset-2017处理2017中的语法特性.

### [Babel-preset-env](https://babeljs.io/docs/plugins/preset-env/)
支持所有es标准的特性，能够根据配置采用特定语法处理插件。比方说配置成支持chrome 56, 很多es6已经支持了，就不需要在转成一大坨es5了，减小了bundle的体积。原理比较粗暴，他参考[compat-table](https://github.com/kangax/compat-table) 把每个浏览器版本支持哪些特性都写在本地的文件里了。

### [preset-latest](https://babeljs.io/docs/plugins/#presets-stage-x-experimental-presets-)
顾名思义，最新的，包括所有的官方preset-*, 但是已经deprecated, 因为env的存在。

### [stage-X](https://babeljs.io/docs/plugins/#presets-stage-x-experimental-presets-)
相对于preset-*, 此插件支持尚未被发布为es标准的实验性质的语法特性，意思就是后面可能会被干掉的。比如说object-rest-spread.
这个比较蛋疼，只能说是特定时间，处于某个状态（0，1，2，3，4）的一系列特性的集合。 babel正在考虑废除，因为直接配置插件更加直观。

# 2. Transform plugin
包含官方和社区插件。他们就是干活的小兵， 组成了preset的头部，裆部和其他部位。

### babel-plugin-transform-runtime
对于JavaScript新增的API和一些全局对象上的方法，在运行时动态插入兼容补丁，避免在每个文件插入重复的补丁。调试的时候可以看一下每个文件头部被插入了一段extend之类的代码。

# 3. Babel-polyfill
需要修改内置api才能达成的功能，譬如：扩展String.prototype，给上面增加includes方法，就属于修改内置API的范畴。这类操作就由polyfill提供。

# 4. babel-register,babel-node,babel-core
参考阮老师文章 http://www.ruanyifeng.com/blog/2016/01/babel.html

对于后台系统开发，大部分情况下可以降低兼容性(忽略IE) , 只适配几种浏览器. 由于目前现在浏览器对es6的适配已经非常高了，（chrome60: 90）, 那么就没有必要将es6都编译成es5了。 我们只需要配置preset-env和一些实现性质的插件就可以愉快的跑起来了。

```
{
  "presets": [
    ["env", {
      "modules": false,
      "debug": true,
      "targets": {
        "chrome": 56
      }
    }]
  ],
  "plugins": [
    "transform-runtime",
    "transform-object-rest-spread"
  ],
}

```