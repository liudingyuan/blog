---
title: IIFE -> CommonJS -> ES modules
date: '2019-01-29'
---

前端开发我们为什么要引入模块化，模块有什么优点：
* 复用性

  一个应用总有一些相同和相似的部分，可能是某个功能，可能是某部分UI。抽象出一个模块，通过复用可以大大提高我们的开发效率。
* 组合性

  将模块进行组合可以得到新的模块，就像乐高积木一样。现在说的很多的组件化开发就是如此，将复杂、大型的应用拆解为一个个小模块，可以降低开发难度。
* 隔离性

  模块之间不会相互耦合，我们可以单独测试、替换应用中的某个模块，便于维护。
  
在ES6之前，JavaScript并没有原生的模块语法（我想js的创造者一定没想到如今js会用来开发越来越复杂的应用），社区中诞生了许多解决方案。我们有必要来了解下目前依然在广为使用的几种方案。

## IIFE
```js
(function (win) {
  win.utils = {}

  function add(num) {
    return num + 1
  }

  utils.echo = function (num) {
    console.log('echo: ', add(num))
  }
})(window)
```
上面的代码就是一个IIFE的例子，IIFE就是一个立即执行的匿名函数表达式。上述代码的执行结果为：window全局对象上新增了一个utils对象，可以调用utils对象里的工具方法。通过这个模式，我们可以暴露出指定的属性和方法，可以私有化内部的属性和帮助函数，外部无法访问。

使用该方案要注意的是，我们始终污染了全局对象（前面例子中的utils），存在被重写的风险。如果我们引入的其他库或者其他代码中使用了同名的变量，那就出事了。

## CommonJS
```js
// utils.js
function add(num) {
  return num + 1
}
module.exports = {
  add
}

// index.js
const utils = require('./utils.js')
const result = utils.add(2)
```
这就是CommonJS，使用`module.exports`暴露模块接口，`require`导入模块。它解决了IIFE遗留的问题，不会污染到全局对象。

Node支持这个方案，但无法在浏览器中使用。实际上它也不适合浏览器使用，因为它是同步导入模块的，这会使得浏览器等待模块下载造成页面空白，影响用户体验。

## 📦模块打包工具的做法
使用webpack打包上面CommonJS例子中的代码，得到如下结果：

<details>
<summary>打包结果</summary>

```js
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
/******/ 		}
/******/ 	};
/******/
/******/ 	// define __esModule on exports
/******/ 	__webpack_require__.r = function(exports) {
/******/ 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/ 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/ 		}
/******/ 		Object.defineProperty(exports, '__esModule', { value: true });
/******/ 	};
/******/
/******/ 	// create a fake namespace object
/******/ 	// mode & 1: value is a module id, require it
/******/ 	// mode & 2: merge all properties of value into the ns
/******/ 	// mode & 4: return value when already ns object
/******/ 	// mode & 8|1: behave like require
/******/ 	__webpack_require__.t = function(value, mode) {
/******/ 		if(mode & 1) value = __webpack_require__(value);
/******/ 		if(mode & 8) return value;
/******/ 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
/******/ 		var ns = Object.create(null);
/******/ 		__webpack_require__.r(ns);
/******/ 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
/******/ 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
/******/ 		return ns;
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = "./index.js");
/******/ })
/************************************************************************/
/******/ ({

/***/ "./index.js":
/*!******************!*\
  !*** ./index.js ***!
  \******************/
/*! no static exports found */
/***/ (function(module, exports, __webpack_require__) {

eval("const utils = __webpack_require__(/*! ./utils */ \"./utils.js\")\n\nconst result = utils.add(2)\n\n\n//# sourceURL=webpack:///./index.js?");

/***/ }),

/***/ "./utils.js":
/*!******************!*\
  !*** ./utils.js ***!
  \******************/
/*! no static exports found */
/***/ (function(module, exports) {

eval("function add(num) {\n  return num + 1\n}\n\nmodule.exports = {\n  add\n}\n\n\n//# sourceURL=webpack:///./utils.js?");

/***/ })

/******/ });
```

</details>

可以看到，webpack使用IIFE的模式自己实现了一个模块化方案，它将所有模块的代码都塞到了一个文件中。同时支持了CommonJS的语法，也没有污染全局对象。

浏览器理解这些代码，并且打包后的文件包含了所有模块的代码，也就不存在浏览器挂起等待模块下载的情况。

## ES Modules
```js
// utils.js
function add(num) {
  return num + 1
}

export {
  add
}

// index.js
import {add} from './utils.js'

const result = add(2)
```
ES6原生支持的模块语法，nodejs、webpack和浏览器都支持。

```html
// 在浏览器中使用
<script src="module.js" type="module"></script>
```
浏览器中module script的加载是异步的，效果等同于设置`defer`属性。

es modules设计模块是静态的，当前模块依赖哪些模块需要在文件顶部声明，不能使用在条件判断语句中。这样的设计给我们带来了一些好处，方便我们看出当前模块的外部依赖情况；利用静态代码分析，去掉没有使用到的代码，也就是**tree shaking**技术。

新的语法提案，动态导入`import()`，可以用来做**code split**。

## 总结
很高兴js有了原生的模块化语法支持😊。可惜浏览器当前的支持情况并不理想，IIFE和各个打包工具依然是主要的使用方案。

---
参考：

* [JavaScript Modules: From IIFEs to CommonJS to ES6 Modules](https://tylermcginnis.com/javascript-modules-iifes-commonjs-esmodules/)
* [understandinges6: Modules](https://github.com/nzakas/understandinges6/blob/master/manuscript/13-Modules.md)
