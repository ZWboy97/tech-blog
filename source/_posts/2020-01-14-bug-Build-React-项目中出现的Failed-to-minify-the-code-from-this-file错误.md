---
title: '[bug] Build React 项目中出现的Failed to minify the code from this file错误'
categories:
  - [技术]
tags:
  - BUG
date: 2020-01-14 20:54:43
---

> 在React项目中运用到了flv.js以及three.js，通过yarn安装依赖，在开发环境下一切正常，但当执行yarn build打生产环境包时，出现了如下错误。

```js
Failed to minify the code from this file:
./node_modules/flv.js/src/utils/logger.js:21
```

### 解决方案
问题出在了库的引入上，原本代码为：
```js
import flv.js from 'flv.js';
```
更改为
```js
import flv.js from 'flv.js/dist/flv.min.js';
```
然后build就可以通过
<!--more-->

### 原因分析
用到的第三方软件包没有被正确编译成ES5
解决方法
- 去项目站点提issue，让第三方软件包开发人员提供pre-compiled
- 自己编译，然后上传
- 将第三方库code直接复制到项目中使用（如果代码比较少的话）

### 总结
解这个bug花了不小的时间，主要还是对React的打包过程和机制还不是很了解，有时间进一步了解一下。