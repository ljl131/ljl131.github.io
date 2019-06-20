---
title: gcc的编译过程
date: 2019-06-19 18:07:13
tags: C语言
---
# gcc的编译过程
## 预处理
宏替换，头文件展开，条件编译
```
gcc hello.c -E -o hello.i
```
## 编译
检查语法，生成汇编文件
```
gcc hello.i -S -o hello.s
```
## 汇编
将汇编文件生成目标文件
```
gcc hello.s -c -o hello.o
```
## 链接
生成可执行文件
```
gcc hello.o -o hello
```