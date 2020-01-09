---
title: 大脑Cahce系列--前端JavaScript系统梳理
categories:
  - [技术]
  - [大脑Cache]

tags:
  - [前端]
  - [JavaScript]
date: 2019-10-14 19:28:16
---

> 背景知识
> - 本文依然按照以往的风格，更注重JavaScript技术的系统性梳理，不赘述技术细节，因为Google一下就好。


### 基础
##### 1. 基础知识
- JavaScript：ES5/6 + DOM + BOM
  - ECMAScript 5 2015（ES6）
  - DOM: 针对HTML的编程接口
  - BOM：对浏览器进行操作的编程接口
- 浏览器内核
  - 渲染引擎：Blink、Webkit（用于浏览器）
  - JS引擎：V8引擎
- 标识符
- strict mode
  - 严格模式：添加“use strict”
  - 让代码变得更可靠
  - 使用方式：全局使用，局部使用（在方法中使用）
  - 推荐编写代码的时候使用严格模式

<!--more-->

- 语句
  - 表达式语句：习惯---最好以分号结尾
  - 流控制语句：for-in, with
  - 异常处理语句：
  - 返回值：
- 变量
  - 局部变量：需要添加var，没有的话会升级为全局变量
  - 全局变量：
- 数据类型：
  - 5种基础数据类型（undifend、null、Boolean、Number、String），以及Object。只限制这6种形式，不存在其他类型的类型
  - typeof运算符：不是函数，返回值为字符串（undefined, boolean, string, number, object, function）
  - undefined: 变量被声明但是未初始化
  - null: 空对象指针。当我们声明一个变量是为了保存一个对象时，但是没法立即给他初始化时候，最好把它设置为null，这样就知道未来这个变量是变量。
  - Number与String数据类型转换
  - Object类型：创建
- 运算符
  - JS种的奇技淫巧都来自于对运算符的使用
  - 基础的运算符，和其他语言一样
  - 布尔运算（!!取bool值,&&短路操作,||默认值）
  - 相等运算符：相等（==，!=）,全等（===，!==）,他们的不同。
  - 条件运算符
- 函数
  - function 声明
  - 参数：arguments，参数都是值传递的（copy简单值，copy对象和函数指针），其实就是局部变量
  - 没有函数重载，（不过可以通过模拟函数实现：通过对arguments进行解析判断）

##### 2.引用类型
- Object

```
{} 等价于 new Object()
a.b 等价于 a[b]
```
- Array

```
[] 等价于 new Array()
array.length
```
  - 判断是否是Array的两种方法：instanceof，isArray
  - typeof VS instanceof
- Date：+ new Date() 得到时间戳
- RegExp: 正则表达式
- Function
- 单例内置对象
  - window
  - Math

##### 3. 作用域
- 作用域链
  - 从外向内部的链条，内部可以访问外部，外部无法访问内部。内部没查找到，则向上外层继续查找
- this作用域：永远指向最后调用它的对象，具有一定的作用域。
- bind():可以为函数调用绑定一个作用域（实际是一个对象）
- 没有块级作用域！！！，花括号无法延长作用域，只可以通过函数的方式来增加作用域
- 闭包：有权访问另一个函数作用域中的变量的函数

```js
function applyArr(){
  var arr = [];
  for(var i=0; i < 10; i++){  // 不存在块作用域，不会新增作用域
    // 不使用闭包
    arr.push(function(){  //使用的i是applyArr的作用域，
      return i;           // 最终在外部执行的时候，i取值一直都是10
    })
    // 使用闭包
    (function(index){     // 通过函数构造一个新的作用域,并将i作为参数传入
      arr.push(function(){
        return index;     // 这样return返回的时候，取得是新创建作用域中的值
      })
    })(i);
  }
  return arr;
}
var arr = applyArr();
var b = arr[1]();
// 如果不使用闭包，那么b最终的值为10
// 使用闭包后，b的值为1
```
##### 4. 面向对象的JavaScript
- 对象属性
  - 一般方法：直接赋值新增属性
  - 数据属性：Object.defineProperty(obj,"name",{configurable,enumerable,writable,value});
  - 访问器属性：get，set
- 创建对象
  - 工厂模式
  - 构造函数模式
  - 原型模式：prototype
  - 构造函数与原型组合构造：属性放在构造函数中，方法放在原型中
- 实例属性和原型属性
  - in：“name” in obj， 用来判断实例obj是否有name属性
  - obj.hasOwnProperty("name"): 用来判断在实例化之后的对象上，目标的属性是否是实例属性or实例方法。
- 对象继承
  - 基于原型链的继承：缺点，属性也被继承了，继承的属性在子类中是原型属性
  - 基于构造函数的继承：父类方法在子类中拷贝，资源浪费。
  - 组合继承：

##### 5. DOM编程
- DOM节点
  - 节点间的关系：父子，兄弟
  - 节点特征：nodeType,nodeName,nodeValue,parentNode...
- document节点的属性
  - document.title：页面标题
  - document.referrer：来自于那个页面，用于安全性判断，防盗链
  - document.domain：解决域不同的问题
- DOM元素节点的默认属性
  - .id
  - .className
  - .title
  - .lang：文本
  - .dir
  - .getAttribute('id')
- DOM操作
  - 创建：document.createElement("div");
  - 添加：someNode.appendChild(newNode);
  - 插入：someNode.insertBefore(newNode,beforeChildNode);
  - 替换：someNode.replaceChild(newNode,firstChild)
  - 删除：someNode.remove(someNode.firstNode);
  - 查找：document.getElementById('id'); document.getElementsByTagName()

##### 6. 事件
- 事件流
  - 用来判断页面的哪一个部分会拥有某个特定的事件，即事件的定位
  - 事件流：冒泡
    - 逐级向上级传递，接收和处理
  - 事件流：捕获
    - 从顶层节点向下传递，接收和处理
  - DOM事件流：组合了冒泡
    - 先是捕获阶段，然后冒泡阶段，每个几点有两次处理事件的机会。
- 事件处理
  - HTML标签中：比如onclick
  - DOM0事件处理：domElement.onclick = function...
  - DOM2事件处理：element.addEventListener("click",function()..., true/false),第三个参数指明是在捕获阶段true还是冒泡阶段处理事件。先绑定的事件先处理。
- 事件处理跨浏览器
  - 封装跨浏览器的事件处理方法，比如安卓控制网页中的事件。
- event属性方法
  - e.stopPropagation()：阻止事件冒泡
  - e.preventDefault(); 阻止一些默认事件
- 事件类型
  - 各种事件，用的时候查
  - PC事件和移动端事件

##### 7. JSON数据通信
- JSON
  - JSON是一种数据格式，和XML是编程语言
  - JSON更高效
  - JSON无法表示undifend类型
  - JSON字符串是双引号
  - JSON中的对象的属性也必须是字符串
  - JSON.stringify(obj)：将js对象序列化成json字符串
  - JSON.parse(jsonText,function(key,value))：解析JSON，第二个参数为解析函数

- AJAX
  - 异步加载
  - 无需前端界面刷新

```js
var xhr = new XMLHttpRequest();
xhr.open("get","url",false); // 请求是否异步【异步：请求期间可以执行后续js，同步：不允许请求期间执行后续js】
xhr.send("");
// 同步的话
xhr.status
xhr.responseText
// 异步的话
xhr.onreadystatechange = function(){  //在执行的过程中会不断执行onreadystatechange
  if(xhr.readyState === 4){ //0.未初始化，1.启动，2.发送，3.接收，4.完成
    xhr.status
    xhr.responseText
  }
}

```

### 三、ES6
##### 1. 历史
- ES发展
  - ES5
  - ES6 = ES2015
  - ES2015... 一年一更
- Evergreen浏览器
  - IE，chrome，firefox，opera...
- 环境
  - Node：执行环境
  - NPM： 依赖以及包管理工具
  - Babel：ECMASCRIPT的转译器，将先进的标准转化为已经支持的标准
- 如何支持新标准
  - 原生支持：开发者查看浏览器的支持情况
  - Polyfill：以贴近新标准的方式封装API的效果
  - 转译器：babel

##### 2. ES6的模块
- 模块
  - 易于拆分，模块加载
  - 模块化
- 模块加载规范
  - UMD 
    - CommonJS
    - AMD
  - CMD
- ES6中的模块加载

```js
// module1.js
export default function methodA(){}
export default function methodB(){}
export default function methodC(){}
// module2.js
import { methodA }from './module1.js';
```

- 打包工具
  - browserify、webpack等：
  - 将多个js模块进行整体打包成一个js文件
  - 在打包之前需要先使用babel将ES6转化为ES5

##### 3. ES6的Class
- 不需要像ES5那样用原型取声明各种方法
- class的typeof值其实也为function
- class内部声明的是实例属性
- ES6实际上是提供了一些语法糖

```js
class ClassName {
  constructor(){}
  getName(){
    return name;
  }
}
var hh = new ClassName();
```

- 继承
  - ES5特别麻烦，ES6就一个extends就可以了

##### 4. ES6常用特性
- 解构赋值

```js
// 从data
const {id = 1} = data;
```
- 展开操作符

```js
//数组展开
[1,2,3,...x]
//属性展开
{a:3,b:4,...c}
```

- 取余

```
[a,b,...rest] = [1,2,3,4,5,6]
// a = 1
// b = 2
// rest = [3,4,5,6]
```
- 模板

```js
`my name is ${name}`
```
- Set 和 Map
 - 类似ES5中的数组和对象
 - 但是不允许出现元素的重复

```js
var set = new Set([1,2,3,4,4]);
set.add(5)

var map = new Map([
  ['a',1],
  ['b',2]]);
map.put('c',3);
```

- Generator函数和Iterator函数
  - Generator用来实现状态机
  - Iterator用来实现迭代和遍历

##### 5.ES6的异步编程 
- 回调函数/事件
  - 遇到多种回调嵌套的时候写起来会很不友好
- Promise
  - 是异步编程的一种解决方案，比传统的基于回调函数和事件的方式更合理和强大
  - Promise对象代表一个异步操作，有三种状态
    - Pending：进行中
    - Resolved：已完成
    - Rejected：已失效
  - Promise.then()
    - 可以接受两个回调函数为参数: then（（resolved, rejected）=>{}）
    - then的返回值可以还是一个Promise对象，即仍是一个异步操作。
      - 可以采用链式写法，即then方法后再调用另一个then
      - 后续的回调函数会等到该Promise对象处理完成之后才会被调用

  ```js
  // 第一个异步操作
  function p1 (value){
    return new Promise((resolve,reject)=>{
      someoperation(value,...)=>{
        if(...){
          reject();
        }else{
          resolve(value)
        }
      })
    });
  }
  //第二个异步操作
  function p2 (value){
    return new Promise((resolve,reject)=>{
      someoperation(value,...)=>{
        if(...){
          reject();
        }else{
          resolve(value)
        }
      })
    });
  }
  // 按照先1->2->1的顺序执行
  p1(10)
  .then(value => {
    p2(value)
  }, rej => {})
  .then(value => {
    p1(value)
  }, rej => {});
```

- asyncy异步编程
### 未完待续