---
title: hexo搭建个人博客
date: 2019-06-19 17:59:40
tags: hexo
---
# 本地部署
## 安装git
linux: 
```
sudo apt-get install git  
```
windows: 去[官网](https://gitforwindows.org)下载   
用git --version来查看版本
## 安装nodejs
Linux：
```
sudo apt-get install nodejs
sudo apt-get install npm
```
windows：去[官网](https://nodejs.org/en/download/)下载   
查看版本
```
node -v
npm -v
```
顺便说一下，windows在git安装完后，就可以直接使用git bash来敲命令行了，不用自带的cmd，cmd有点难用。
## npm换源
国外网站下载速度超慢，换成淘宝的源
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
## 安装hexo
```
cnpm install -g hexo-cli
```
建立一个blog文件夹，通过cd命令进入此文件夹
```
hexo init
```
启动本地服务器
```
hexo s
```
在浏览器中输入http://localhost:4000可看到发布的博客，默认有一个hello world
## 创建新的博客
```
hexo new "博客名"
```
用markdown语法来写
```
hexo clean
hexo g
hexo s
```
# 博客部署到远端
## 创建github仓库
仓库名必须为：YourgithubName.github.io
## 配置git
```
git config --global user.name "yourname"
git config --global user.email "youremail"
```
查看是否配置成功
```
git config user.name
git config user.email
```
## 安装部署插件
```
cnpm install --save hexo-deployer-git
```
## 设置配置文件 _config.yml
```
deploy:
  type: git
  repo: https://github.com/YourgithubName/YourgithubName.github.io.git
  branch: master
```
## 发布
```
hexo g
hexo d
```
通过https://YourgithubName.github.io/访问博客
# 更改主题
```
git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```
blog目录下的_config.yml文件的theme改为yilia
## 配置图片资源
在路径为 themes/yilia/source/下，可添加一个 assets/img的文件夹  
在themes/yilia/_config.yml的配置文件中直接配置照片
```
# 微信二维码图片(打赏二维码)
weixin:  /assets/img/wechat.png

# 头像图片
avatar:  /assets/img/head.jpg

# 网页图标
favicon:  /assets/img/head.jpg
```
## 显示文章目录
方便阅读文章, 在 themes/yilia/_config.ym中进行配置 toc: 2即可，它会将你 Markdown 语法的标题，生成目录，目录查看在右下角。
## 查看所有文件，提示缺失模块
```
cnpm i hexo-generator-json-content --save
```
然后再blog目录下的_config.yml文件添加下面的内容
```
# yilia主题需要添加
# jsonContent
jsonContent:
    meta: false
    pages: false
    posts:
      title: true
      date: true
      path: true
      text: false
      raw: false
      content: false
      slug: false
      updated: false
      comments: false
      link: false
      permalink: false
      excerpt: false
      categories: false
      tags: true
```