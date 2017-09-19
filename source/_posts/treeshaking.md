---
title: webpack2之tree shaking
date: 2017-09-14 07:00:00
tags: [webpack, 工程化]
---
译文：http://2ality.com/2015/12/webpack-tree-shaking.html
>作者:  Dr. Axel Rauschmayer, Twitter, Mastodon

tree shaking, 顾名思义：摇树，把树上面的枯枝烂叶子甩下来，目的是减小bundle size。这个构建优化手段起源于另一个打包工具[rollup](https://github.com/rollup/rollup).

### 工作原理
举例有es6模块如下：

```
export function foo() {
    return 'foo';
}
export function bar() {
    return 'bar';
}
```

首先，webpack将所有的es6 module打包到bundle文件里面。那些从来没有被import的export, 不再被export， 变成了[dead code](https://en.wikipedia.org/wiki/Dead_code)。 dead code就是在编译时不被export或者跟本用不到的函数、变量等等。比如代码中的bar.

```
function(module, exports, __webpack_require__) {

	/* harmony export */ exports["foo"] = foo;
	
    /* unused harmony export bar */;

  function foo() {
    return 'foo';
  }
  // i am dead function .... 
  function bar() {
    return 'bar';
  }
}
```
需要注意的是， babel-preset-es2015默认会用transform-es2015-modules-commonjs把es6 module处理成commonjs。这就干掉了es6 module的静态特性。但是由于cmd模块系统是个对象，运行时才能确定模块的特定属性，那么在编译时就不能确定哪些exports是没用的了。模块将被转化成为如下cmd：
```
function(module, exports) {
	'use strict';
	Object.defineProperty(exports, "__esModule", {
	    value: true
	});
	exports.foo = foo;
	exports.bar = bar;
	function foo() {
	    return 'foo';
	}
	function bar() {
	    return 'bar';
	}

}
```
目前babel有个简单的配置可以跳过模块的处理, 保留es6 module：
```
module: false;
```

然后，利用ulglify工具干掉dead code, 进行混淆:
```
function (t, n, r) {
    function e() {
        return "foo"
    }
    n.foo = e
}
```

最后对bundle进行压缩操作, 有效减小了bundle文件体积。

下面一段是瞎写写的：

有些人觉得自己不会写出这些dead code、垃圾代码，因而怀疑tree shaking的意义。其实现在很多库是设计成可以按需加载的，如echarts, rxjs, ramda，目的就是为了不引入所有的代码， 减小bundle体积。比如工具类的ramda有各种list, object, logic的各种方法， 你可能只用到了一个，是不需要将所有的方法都打包到bundle里面的。echarts可能只用到柱状图，也不需要把其他所有种类的图形打包起来， 这个太大了。就算现在网速快，流量也是要钱的。。。