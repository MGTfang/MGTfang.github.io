title: 利用travis自动化部署博客项目
author: Peng Fang
tags:
  - 自动化部署 博客
categories:
  - 博客
date: 2017-07-25 17:36:00
---
## 目的
本文记录如何用travis实现自动化部署GitHub仓库中的博客项目，并完成域名访问。

## travis登录和授权
travis可以用GitHub账号登录，登录后在profile页授权允许travis访问的仓库。 
> 比如我这里的GitHub仓库是vue-ghpages-blog，也是我准备部署的博客项目。

## GitHub pages的一点概念
搭建项目站点的两种方式：
>一种是创建一个名为 `GitHub用户名.github.io`的仓库，在外网以该仓库名作为地址进行访问就可以访问到仓库里面的静态页面了。
> 另一种是给仓库创建一个gh-pages分支，将运行`npm run build`命令编译生成到项目dist目录下的文件push到gh-pages分支。访问`GitHub用户名.github.io/仓库名`如`https://github.com/MGTfang/vue-ghpages-blog`，也能访问到编译项目生成的静态文件。

## travis自动化部署的条件
在授权travis访问仓库（项目）的前提下，还需要在仓库的根目录下有个.travis.yml文件，部分内容如下：
```javascript
script:
  - npm test

after_success:
  - npm run build
  - cd dist
  - echo "dtechvoi.com" > CNAME
  - cp index.html 404.html
  - git add --all .
  - git commit --message "Automatically update from travis-ci"
  - git push --quiet "https://${GH_TOKEN}@${GH_REF}" gh-pages:gh-pages

# Note: you should set Environment Variables here or 'Settings' on travis-ci.org
env:
  global:
    - GH_REF: github.com/MGTfang/vue-ghpages-blog.git
    # - GH_TOKEN: 'Your GitHub Personal access tokens, via https://github.com/settings/tokens'
```
当将更改的文件push到远端仓库（这里是vue-ghpages-blog的develop分支）的时候，travis会自动执行一次build，build操作根据.travis.yml内容来进行。例如根据上面的脚本（有部分省去了），先设置环境变量GH_REF、GH_TOKEN，再更新nvm，运行npm install、npm test。
>     我这里报错：没有找到electron这个包，因此运行`npm install electron --save`进行安装
>     在`npm run test`成功以后，会将build生成的文件推送到gh-pages分支。
>     这里可能会出现没权限访问远端仓库的error，这样需要通过`https://github.com/settings/tokens`来设置访问token，特别主要要将repo勾选，不然照样没有权限。

## 域名绑定
在域名的解析设置下添加三条记录：
>     记录类型 	主机记录 	解析线路(运营商) 	记录值
>     CNAME	      @	       默认	    MGTfang.github.io.
>         A	      www	     默认	    192.30.252.153
>	     A	      www	     默认	    192.30.252.154

包括两条A记录和一条CNAME记录，A记录指向以www开头的域名，记录值是GitHub的IP地址，CNAME的记录值是可以访问GitHub pages的地址。   
除了绑定域名，还需要在访问仓库额根目录下创建一个CNAME文件，并在其中写入绑定的域名，这样直接访问域名就能访问gh-pages分支下的静态页面。