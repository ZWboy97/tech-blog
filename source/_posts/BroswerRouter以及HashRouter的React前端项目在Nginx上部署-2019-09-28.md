---
title: BroswerRouter以及HashRouter的React前端项目在Nginx上部署
date: 2019-09-28 13:43:12
categories: [技术]
tags: 
- React 
- Nginx
---

> 背景：视频云的前端项目使用了React框架，并通过React-Router实现了SPA单页应用。前端包括管理端和观看端，而React-Router有两种常用类型的Router，分别是HashRouter以及BrowserRouter，为了体验二者的不同，在管理端使用了HashRouter，而在观看端使用了HashRouter。

### 两种React-Router

##### 1. HashRouter
- history: 使用 hash history
- 通过Hash部分来表示路由，属于浏览器端路由，Hash部分在请求的时候并不会提交到服务器，也就是说不管是请求 #foo 还是 #bar ，服务只知道请求了 index.html 并不知道 hash 部分的细节。
- url样例：/#/user
- 不需要服务器支持，由浏览器负责路由。

<!--more-->

##### 2. BrowserRouter
- 使用的是 HTML5 的 History API，浏览器提供相应的接口来修改浏览器的历史记录
- 需要服务器支持，属于服务器路由
- url样例：/user/lives
- 路由直接在URL中，原则上每次URL切换的时候，都会像服务器请求页面
- 但借助html5里面history的新的api，可以使用js改变url的值，而不会向服务端发请求。(刷新界面的时候还是会请求页面)
- 是react-router官方推荐的

### 使用Nginx部署的时候需要特殊的配置

- 传统的前端部署之后，浏览器都是直接通过浏览器地址栏中的uri去请求服务器对应位置的文件，而在使用React-Router进行路由管理来实现SPA之后，依然采用这样的配置的话，初次加载首页是正常的，但当浏览器URI被改变之后，再点击刷新，则页面不显示或者404错误。
- 原因在于，使用React-Router之后，页面内的路由是由前端自行渲染的，未通过后端，而是js动态更改了location。而当我们刷新界面时，首先访问的是后台地址，然后在返回的页面中再去加载React代码并执行。
- 问题在于，React应用在运行的时候可能会去更改浏览器的URL，而实际服务器上可能并不存在与URI对应的资源（因为资源的定位已经交给React来做了），所以刷新页面的时候，浏览器直接用URI去寻找资源，可想而知会产生异常（404或者不显示）。
- 而我们使用Webpack来打包React之后，最终的输出就是一个index.html附加一些文件，当然文件的定位逻辑也存在这些文件中。所以，默认的由Nginx服务器来定位这些资源的方式是行不通的，需要将该任务交给React本身来做。
- 所以，思路是，浏览器在使用React应用期间，无论URI是否更改，服务器都直接无脑返回index.html这个页面就可以，浏览器得到index.html，加载React，剩下的任务都交给React本身去做就好了。
- 使用Nginx的try_files，来将前端的请求都映射到index.html。

### 部署HashRouter的Nginx配置
- 在Nginx中配置一个vhost

```bash
# nginx.conf整体配置大概如下：
server {
    listen 8083;
    root /home/name/..../build; # 前端build完成之后的静态资源路径
    index index.html index.htm;

    location / {
        try_file $uri $uri/ /index.html;  # url 切换时始终返回index.html
    }

    # 其他配置
    # 图片样式缓存1年
    location ~* /app.*\.(js|css|png|jpg)$ {
       access_log off;
        expires    365d;
    }
    # html/xml/json 文件不缓存
    location ~* /app.*\.(?:manifest|appcache|html?|xml|json)$ {
        expires    -1;
    }
}

```

### 总结
部署项目的时候搞得很头痛和低效，深入搞明白之后，再也不怕React前端的部署了。

### 参考
- 1.[react-router-dom下的BrowserRouter和HashRouter](https://www.cnblogs.com/soyxiaobi/p/11096940.html)