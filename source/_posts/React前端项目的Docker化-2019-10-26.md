---
title: React前端项目的Docker化
categories: [技术]
tags:
  - React
  - Docker
  - 前端
date: 2019-10-26 23:09:00
---

### 背景
之前的整个视频云项目都是直接部署在实验室的服务器上，前端直接通过Nginx进行部署（运用反向代理）。现在需要将开发完成的项目整体迁移到阿里云上进行部署，能够通过外网IP和域名进行访问。

### 为什么要前端Docker化
在进行选型的时候，选择了阿里云提供的Kubernetes容器计算服务，因为相比直接购买ECS云主机或者轻量云服务器，更适合我们项目前期的低使用率以及后期的可扩展性需求，不必花费几千元仅购买了个性能较强的云主机。并且将服务拆分为一个个小小的Docker中也更便于项目的运维和部署。

<!--more-->

### 前端Docker化思路
##### 1. 前端镜像形式
- 以Nginx为基础镜像
- 将前端打包完成后的build内容copy到镜像中
- 将Nginx的配置文件Copy到镜像中的对应位置
- 启动nginx服务，对外暴露80端口

##### 2. 前端打包过程也要在构建中进行

- 直接将打包完成的build文件Copy进镜像是可行的，但是如果想使用阿里云提供的容器镜像仓库的自动构建服务，需要将前端的打包过程也在镜像构建的过程中进行。
- 我们知道，最终的前端镜像中不需要那些用于打包的各种依赖，并且实际上我们的代码仓库中，node_modules等依赖并没有添加到git仓库中。
- Docker构建可以分阶段进行：也就是构建过程分为build环境和最终的production环境。
- 而通过Dockerfile正好支持这样的构建方式

### 所使用的Dockerfile文件

```dockerfile
# build environment
FROM 	  node as build
WORKDIR	/app
ENV	    PATH /app/node_modules/.bin:$PATH
COPY 	  . /app
RUN	    yarn
RUN 	  yarn build

# production environment
FROM 	  nginx
COPY	  --from=build /app/build /usr/share/nginx/html
RUN 	  rm /etc/nginx/conf.d/default.conf
COPY 	  deploy/nginx.conf /etc/nginx/conf.d
EXPOSE 	80
CMD 	  ["nginx", "-g", "daemon off;" ]
```

### 所使用的Nginx配置文件

```
# deploy/nginx.conf
server {
	listen 80;
	
	location / {
		root	/usr/share/nginx/html;
		index 	index.html	index.htm;
		try_files $uri $uri/	/index.html;
	}
	
	error_page 500 502 503 504	/50x.html;

	location = /50x.html {
		root	/usr/share/nginx/html;
	}
}

```

### 问题与思考
##### 1. 镜像的构建过程太漫长了
- 主要时间大部分都用在了build环节，并且是依赖安装环节
- 缩减构建时间很重要
- 优化思路
  - 项目中有很多多余的依赖，去除这些未使用的依赖
  - 更换yarn依赖安装的源（阿里似乎已经帮我们更换了）
  - 区分依赖区分开发环境和生产环境，比如类似eslint这样的依赖，在生产打包中就不需要了。


### 总结 
最后，由衷感受，Docker真是个好东西，搞技术的都应该学一下。