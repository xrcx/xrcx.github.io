---
title: Gitbook
categories: 工具
toc: true 
theme: next
date: 2017-07-01 20:41:34
---


### 概述
一个非常好的本地，在线文库制作的工具，可用输出文档，积累开发经验，完成调研任务的记录等。
1. 官网 : https://www.gitbook.com/

## 操作
### (一)安装gitbook

1. `npm install gitbook -g`
2. `npm install gitbook-cli -g`

###（二）生成书并浏览
1. `gitbook init`  初始化书
2. `gitbook build` 编译
3. `gitbook serve` 开启服务
SUMMARY.md 文件夹下执行 gitbook serve, 然后http://localhost:4000

###（三）生成不同格式书记(pdf/epub/mobi)
如果没安装，先安装环境
#### 安装环境步骤
1. `npm install gitbook-pdf -g`
2. 安装calibre    http://calibre-ebook.com/download
3. 将calibre的安装目录加入到系统环境变量Path里面,重启电脑

#### 生成电子书
1. `gitbook pdf`

#### 注意
1. README.md : 介绍
2. SUMMARY.md: 目录

