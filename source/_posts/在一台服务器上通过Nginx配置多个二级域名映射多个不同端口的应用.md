---
title: 在一台服务器上通过Nginx配置多个二级域名映射多个不同端口的应用
date: 2019-09-21 23:35:32
tags: Nginx
categories: [技术]
comments: true
---

> 背景：需要在实验室服务器上部署gitlab，jira，jenkins等等Server服务，这些服务工作在不同的端口，无法直接通过http的80端口访问，所以之前是通过IP+端口号的方式进行访问。在申请了自己的一个域名之后，想要通过多个二级域名来访问。

#### 需求
能够通过gitlab.domain.cn访问绑定在82端口上的gitlab服务，通过jenkins.domain.cn访问绑定在9090端口上的jenkins服务。
#### 思路
- 1.在域名的解析中分别添加gitlab, jenkins,以及www三个记录，指向均为实验室服务器的IP地址。
- 2.使用Nginx作为代理服务器，监听80端口，通过server_name进行匹配，将请求转发到对应的应用服务器。(这应该属于Nginx的反向代理功能，即为多个服务提供代理服务)
#### 实际解决
- 1.编辑Nginx配置信息（Ubuntu）
```bash
# 配置文件路径：/etc/nginx/conf.d/domain.conf，（没有的话需要创建）
# Nginx会自动include /etc/nginx/conf.d 目录下所有以.conf结尾的配置到主配置文件nginx.conf中.

server {
        listen 80;  # 设置Nginx对外监听80端口
        server_name *.domain.cn;  # 绑定到该服务器的域名
                if ( $http_host ~* "^(.*?)\.domain\.cn" ) {  # 对http_host进行正则匹配，解析domain
                        set $domain $1;
                }
        location / {
                proxy_set_header        X-Real-IP       $remote_addr;
                proxy_set_header        Host            $http_host;
                # 分别处理各个domain
                if ( $domain ~* "www" ) {
                        proxy_pass http://localhost:82;  # 通过proxy_pass 进行代理转发
                }
                if ( $domain ~* "gitlab" ) {
                        proxy_pass http://localhost:82;
                }
                if ( $domain ~* "jenkins" ) {
                        proxy_pass http://localhost:9090;
                }
        }
}
```
- 2.检查配置文件合法性
```bash
# 需要将nginx添加到环境变量中
sudo nginx -t
```
- 3.更新Nginx配置
```bash
sudo nginx -s reload
```
#### 踩坑
最开始的思路是在Nginx中配置多个server块，再每个块中再配置proxy_pass来实现，配置文件如下。
```bash
# jenkins.domain.cn.conf
server {
        listen 80;  # 设置Nginx对外监听80端口
        server_name jenkins.domain.cn;  
        proxy_pass http://localhost:9090;
}
```
```bash
# gitlab.domain.cn.conf
server {
        listen 80;  # 设置Nginx对外监听80端口
        server_name gitlab.domain.cn;  
        proxy_pass http://localhost:82;
}
```
Reload Nginx之后发现，二级域名是时效的，两个域名均指向的是gitlab，无法实现想要的效果。分析应该是gitlab.domain.cn.conf先被include到nginx.conf中，所以生效的是gitlab吧。
不确定，还需要进一步研究。
#### 总结
Nginx是个强大的工具，可以作为Web服务器，文件服务器，代理服务器，反向代理服务器，直播服务器，负载均衡服务器等等，值得深入学习。