---
title: git and github使用指南
date: 2019-06-20 10:27:18
tags: git
---
# Git工具
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
## 创建新仓库
创建新文件夹，打开，然后执行
```
git init
```
## 将文件添加到仓库
### 1. 在仓库下新建一个文件readme.txt,内容：1111111
### 2. 使用git add命令将文件添加到暂存区
```
git add readme.txt
```
### 4. 使用git status命令，查看状态
```
git status -s
```
### 3. 使用git commit命令，把文件提交到仓库
```
git commit -m 'readme.txt commit'
```
引号中的内容为提交的注释  
### 5. 使用git status命令，查看状态
```
git status
```
显示如下信息，代表将所有改动都提交了
```
On branch master
nothing to commit, working tree clean
```
### 6. 改动readme.txt,的内容，用git status查看状态
```
git status
```
结果
```
 M readme.txt
```
### 7. git diff查看改动的内容,然后重新提交。
```
git diff readme.txt
```
## 版本回退
### 1. 显示提交日志
```
git log
```
### 2. 版本回退
回到上一个版本
```
git reset --hard HEAD^
```
回到上上一个版本
```
git reset --hard HEAD^^
```
回到前100个版本
```
git reset  --hard HEAD~100
```
### 3. 获得版本号
```
git reflog
```
### 4. 根据版本号回退
```
git reset --hard b2a3418
```
# github
## 本地上传
创建一个同名的仓库
然后通过下面的命令使本地和远程库联系起来
```
git remote add origin https://github.com/ljl131/testgit.git
```
第一次推送master分支时，加-u参数。不但会把本地的master分支推送到远程，还会把本地的master分支和远程的分支关联起来。
```
git push -u origin master
```
以后就可以只输入
```
git push origin master
```
就可以把最新的修改推送的github上。

## 远程克隆
```
git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```
网址 本地文件夹
# 创建于合并分支
## 创建分支dev
```
git checkout -b dev
```
创建完之后自动进入此分支
```
git branch dev
```
只创建不进入
## 查看当前分支
```
git branch
```
## 在当前分支上提交内容
```
git add readme.txt
git commit -m 'dev readme.txt'
```
## 切换分支
```
git checkout master
```
## 和并分支
将指定分支合并到本分支上
```
git merge dev
```
## 删除分支
```
git branch -d dev
```
## 分支策略
首先master主分支应该是非常稳定的，也就是用来发布新版本，一般情况下不允许在上面干活，干活一般情况下在新建的dev分支上干活，干完后，比如上要发布，或者说dev分支代码稳定后可以合并到主分支master上来。   
例如bug分支：在开发中，会经常碰到bug问题，那么有了bug就需要修复，在Git中，分支是很强大的，每个bug都可以通过一个临时分支来修复，修复完成后，合并分支，然后将临时的分支删除掉。
# 多人协作
## 查看远程库信息
```
git remote -v
```
## 推送分支
```
git push origin master
```
```
git push origin dev
```
## 抓取分支
```
git checkout  –b dev origin/dev
```
因此：多人协作工作模式一般是这样的：

1. 首先，可以试图用git push origin branch-name推送自己的修改.
2. 如果推送失败，则因为远程分支比你的本地更新早，需要先用git pull试图合并。
3. 如果合并有冲突，则需要解决冲突，并在本地提交。再用git push origin branch-name推送。