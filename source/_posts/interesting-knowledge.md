---
title: Interesting Knowledge
date: 2024-10-19 13:03:06
tags:
---

本文的目的是避免遇到重复的问题每次都要去重新查询，记录问题的解决方案能够有效提升效率。

## C++

### 编译工具

`github`的`release`中选择`x86_win32_ucrt`版本

## Git

### 克隆失败

一般是项目过大，下载速度跟不上，导致`http`请求被强行终止。因此，需要使用国内镜像

`http://hub.nuaa.cf/{project name}`

**其他镜像**：

`https://hub.yzuu.cf/`

`https://hub.nuaa.cf/`

`https://hub.fgit.ml/`

### submodule

submodule is designed to resolve problem that import other's project.

* **add submodule**

  `git submodule add <remote url of third lib> <path in your project>`

  this command import other's project into your project.

* 

## Power Shell

### conda 虚拟环境

想要在`powershell`中能够切换`conda`虚拟环境，需要使用`conda init powershell`创建脚本文件，从而可以实现虚拟环境的切换。重启`powershell`后会出现错误：`can not load file`。这是因为`powershell`**默认不允许**执行`powershell`脚本，需要修改注册表规则，允许其执行脚本`Set-ExecutionPolicy -ExecutionPolicy RemoteSigned`

## 换镜像源

### pip

* 查看当前源：
* 更换镜像源：

## 环境变量

本节记录与环境变量相关的问题。环境变量是向系统提供可执行文件的路径，方便从命令行运行程序。

### 环境冲突

如果在环境变量中存在两个路径拥有同名的程序，会执行排名靠前的程序。因此如果发现程序不符合预期，可以通过`Get-Command python | Format-List *`命令查看当前程序使用的环境变量路径，从而确定是否是预期路径。如果不是，可以通过删除环境变量或者调整优先级的方式修复。

## 网络代理

*Clash*是一款功能强大的代理软件,广受网络用户的喜爱。它采用规则引擎的设计,使用户可以根据自己的需求灵活配置代理规则。通过Clash,用户可以实现按需代理、分流、负载均衡等高级功能,大大提升上网体验。

### 概念

**系统代理**：也就是代理程序会将计算机发出的请求拦截并转发到代理服务器。一般**不能转发UDP（游戏）流量。**

**Tun模式**：代理程序会**创建虚拟网卡**，配置操作系统，让**所有请求都发送到该虚拟网卡**，然后代理程序再从该网卡读取请求，**因此流量消耗会变大**。

### 问题

* **内网无法访问**

  这是因为代理程序使用了代理程序指定的DNS，而内网一般直接使用网关作为DNS，所以需要修改配置文件将DNS改为内网的DNS。

## NPM

使用NPM来创建

`npm config set registry https://registry.npmmirror.com`

## Regex

正则表达式可以看作由基本单位、量词组成。基本单位就是一次匹配的基本单元，量词就是描述希望基本单位出现次数的符号。

### unit

基本单位除了**字符值**之外，还可以通过**函数**等方式构建。

* 字符值：a-z, A-Z, 0-9等显式可见字符
* 函数: [a-z], [A-Z], [\^0-9]等都可以视为一个unit因为他们只匹配一个字符。
* 还可以通过(abc)来构建多个字符组成一个unit。

以上规则可以看[正则表达式 – 元字符 | 菜鸟教程 (runoob.com)](https://www.runoob.com/regexp/regexp-metachar.html)

## Typora

Typora is a markdown editor.

#### PicGo

use PicGo establish a bed of picture in the GitHub.

* Setup GitHub token in developer setting to fill the settings of PicGo.
* Fill the picture bed settings of GitHub in PicGo.
* Typora setting the picture action.
