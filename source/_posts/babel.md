---
title: 关于Babel与构建优化
date: 2017-08-28 15:38:15
tags: [webpack, babel, 工程化]
---

# 1.preset-*
### [官方preset](https://babeljs.io/docs/plugins/#presets-official-presets)
举例：preset-es2015将ES6编译成ES5，preset-2017处理2017中的语法特性.

### [babel-preset-env](https://babeljs.io/docs/plugins/preset-env/)
支持所有es标准的特性，能够根据配置采用特定语法处理插件。比方说配置成支持chrome 56, 很多es6已经支持了，就不需要在转成一大坨es5了，减小了bundle的体积。原理比较粗暴，他参考[compat-table](https://github.com/kangax/compat-table) 把每个浏览器版本支持哪些特性都写在本地的文件里了。

### [preset-latest](https://babeljs.io/docs/plugins/#presets-stage-x-experimental-presets-)
顾名思义，最新的，包括所有的官方preset-*, 但是已经deprecated, 因为env的存在。

### [stage-X](https://babeljs.io/docs/plugins/#presets-stage-x-experimental-presets-)
相对于preset-*, 此插件支持尚未被发布为es标准的实验性质的语法特性，意思就是后面可能会被干掉的。比如说object-rest-spread.
这个比较蛋疼，只能说是特定时间，处于某个状态（0，1，2，3，4）的一系列特性的集合。 babel正在考虑废除，因为直接配置插件更加直观。

babel处于构建时，只处理语法，不处理api.
# 2. babel-core(标准库)
顾名思义，核心！核心！核心！各种es5,6,7各种polyfill。
例如对Arry.from进行Polyfill:
````
var core = require('core-js/library');
core.Array.from(new core.Set([1, 2, 3, 2, 1])); 
````

# 3. babel-runtime
由core-js 与 regenerator-runtime构成。
使用方法：const Promise = require('babel-runtime/core-js/promise'), 比较原始。
供编译模块复用工具函数, 主要目的是减小包体积。对比babel-polyfill, 后者是会污染全局的，使用也很粗暴： 
````
import 'babel-polyfill';
````


# 4. babel-plugin-transform-runtime, babel-polyfill

transform-runtime对于JavaScript新增的API和一些全局对象上的方法，在运行时动态插入兼容补丁，避免在每个文件插入重复的补丁。调试的时候可以看一下每个文件头部被插入了一段的代码。 
配置简单(.babelrc)，比babel-runtime更加智能，实际上是站在babel-runtime的肩膀上，而babel-runtime是站在了babel-core的肩膀上。
容易和babel-polyfill搞混。参考知乎上的一篇[文章](https://zhuanlan.zhihu.com/p/29058936)。
主要区别在于babel-polyfill侵入性太强，会修改builtins 和 prototype, 类似于很早的prototype.js, 因此绝对不能用来搞轮子，容易破坏别人的轮子。babel-plugin-transform-runtime采用sandbox，不修改全局变量, 而且不修改prototype， 适合开发第三方库。

# 5. babel-register,babel-node
其他参考阮老师文章 http://www.ruanyifeng.com/blog/2016/01/babel.html

# 关于构建优化

对于后台系统开发，大部分情况下可以考虑现在浏览器. 由于目前现在浏览器对es6的适配已经非常高了，chrome60达到90%, 那么就没有必要将es6一股脑都编译成es5了。 我们只需要配置preset-env和一些transform插件就可以愉快地跑起来了。
对ts可以做同样的优化：先将ts编译的target设置为es6, 再送到babel按浏览器环境进行polyfill， 最终在浏览器里调试的代码要好看的多。我觉得最好的一点就可以保留class， 而不是其他的方式去实现它。

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
