---
title: update-alternatives 让多个版本库共存
categories:
  - [技术]
tags:
  - linux
date: 2019-12-16 10:16:33
---

> 记录在解决Ubuntu系统python2.7与python3.4共存的问题，使用update-alternatives在两个版本之间进行切换

### update-alternatives
##### 检查update-alternatives是否已经检测到两个版本的python
```
update-alternatives --list python
```
如果出现以下错误，则说明未添加
```
update-alternatives: error: no alternatives for python
```

<!--more-->
##### 更新替代列表，将两个版本的python放入其中
update-alternatives --install [link] [name] [path] [priority]
```
update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
update-alternatives --install /usr/bin/python python /usr/bin/python3.4 2
```
##### 再此检查是否添加成功
```
update-alternatives --list python
```
结果
```
/usr/bin/python2.7
/usr/bin/python3.4
```
##### 切换配置
```
update-alternatives --config python
```
结果
```
  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /usr/bin/python3.4   2         auto mode
  1            /usr/bin/python2.7   1         manual mode
  2            /usr/bin/python3.4   2         manual mode

Press enter to keep the current choice[*], or type selection number:
```

### 总结
> 使用update-alternatives工具之后，在不同版本之间穿梭将变得十分方便，是个值得掌握的工具