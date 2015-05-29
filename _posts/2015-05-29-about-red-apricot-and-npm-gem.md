---
layout: post
title: npm，gem，android sdk加速器——红杏公益代理
description: 红杏开放了公益代理，专门为开发人员开放，免费使用
category: blog
---
我是一直长期使用[红杏](http://honx.in/_VAafG-z5ND6_19m-)，这是攻城狮的必备神器呀。而且他是基于浏览器按需进行代理，所以特别符合需求。全局代理模式不实用。因为工作时间段就不能用了——连不上公司内部开发环境。

但是用「红杏」又碰到一个棘手的问题，像通过npm, gem, android sdk, github等不稳定，偶尔就无法使用。

前段时间，[红杏](http://honx.in/_VAafG-z5ND6_19m-)开放了公益代理，专门为开发人员开放，免费使用。强力推荐！

这个是我的推广链接:[http://honx.in/_VAafG-z5ND6_19m-](http://honx.in/_VAafG-z5ND6_19m-), 你通过这个链接安装购买1个月以上的服务，你我都将获赠10天。

### 如何使用红杏的公益代理

公益代理为：`http://hx.gy:1080` (简拼)  
hostname: `hx.gx`  
port:`1080`  

**github加速**  
`git config --global http.proxy http://hx.gy:1080`

**npm加速**  
For `HTTP`:  
```bash
npm config set strict-ssl false  
npm config set proxy http://hx.gy:1080  
npm install xxx  
```
or    
```bash
npm --proxy http://hx.gy:1080 install
npm --proxy http://hx.gy:1080 --without-ssl --insecure -g install
```

**gem加速**  
`gem install -p http://hx.gy:1080 compass`

**各类IDE加速**  
直接在IDE的proxy中配置http代理可以了

### 何为公益代理
只供开发人员使用，是免费的。目前暂只支持的域名如下：

> android.com    
s3.amazonaws.com  
bitbucket.org  
bintray.com  
chromium.org  
clojars.org  
registry.cordova.io  
dartlang.org  
download.eclipse.org  
marketplace.eclipse.org  
github.com  
githubusercontent.com  
golang.org  
googlesource.com  
storage.googleapis.com  
code.google.com  
googlecode.com  
dl.google.com  
dl-ssl.google.com  
getcomposer.org  
gradle.org  
elpa.gnu.org  
gopkg.in  
ionicframework.com  
cordova.iriscouch.com  
plugins.jetbrains.com  
macports.org  
maven.org  
melpa.org  
packages.meteor.com  
mendeley.com  
nbd.name  
www.nuget.org  
npmjs.com  
npmjs.org  
pypi.python.org  
packagist.org  
packagecontrol.io  
rubygems.org  
repo.typesafe.com  
tromey.com  
9fans.net  


### 郑重推荐使用红杏
[红杏](http://honx.in/_VAafG-z5ND6_19m-)非常简单好用的工具，省时省力。可以轻松管理需要代理的域名，对于不可访问的域名，智能提示。身边同事差不多人手一个。

好工具，当然要分享落。



