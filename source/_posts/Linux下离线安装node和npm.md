title: Linux下离线安装node和npm
author: Peng Fang
tags:
  - 前端环境
categories:
  - 环境安装
date: 2017-08-10 17:56:00
---

本文记录在Linux环境下怎样离线安装node和npm环境。

## 上传安装包到服务器
>从[Download Node.js and npm](https://nodejs.org/en/download/)下载Linux Binaries 
>可运行`uname -a`查看系统的位数，如x86_64为64为系统。
>先上传安装包到FTP，用root账号登录服务器后从FTP下载安装包

## 执行安装命令
1. 解压安装包到想要安装node的目录
``` sh
sudo mkdir /usr/lib/nodejs
sudo tar -xJvf node-v8.2.1-linux-x64.tar.xz -C /usr/lib/nodejs
cd /usr/lib/nodejs
sudo mv node-v8.2.1-linux-x64/ node-v8.2.1
```
2. 设置环境变量
设置环境变量`vi ~/.profile`，在最后添加下面的shell命令
```sh
# Nodejs
export NODEJS_HOME=/usr/lib/nodejs/node-v8.2.1
export PATH=$NODEJS_HOME/bin:$PATH
```
执行`source ~/.profile`使环境变量生效

3. 创建软链接
配置了环境变量，有可能出现npm命令找不到的情况。我的办法是添加软链接：
```sh
sudo ln -s /usr/lib/nodejs/node-v8.2.1/bin/node /usr/bin/node
sudo ln -s /usr/lib/nodejs/node-v8.2.1/bin/node /usr/lib/node
sudo ln -s /usr/lib/nodejs/node-v8.2.1/bin/npm /usr/bin/npm
```
> 注意在Windows下运行过npm install安装了node组件的，到Linux下面删除node_components目录重新运行npm install。

4. 测试安装
执行下面的命令测试安装是否成功
```sh
node -v
```
这里输出：
> v8.2.1

```sh
npm version
```
这里输出：
>     { 
>      'vue-riskaudit': '1.0.5',
>       npm: '5.3.0',
>       ares: '1.10.1-DEV',
>       cldr: '31.0.1',
>       http_parser: '2.7.0',
>       icu: '59.1',
>       modules: '57',
>       node: '8.2.1',
>       openssl: '1.0.2l',
>       tz: '2017b',
>       unicode: '9.0',
>       uv: '1.13.1',
>       v8: '5.8.283.41',
>       zlib: '1.2.11' 
>     }