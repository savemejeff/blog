---
layout: post
title:  "First github page blog"
date:   2022-05-29 19:21:00 +0800
categories: Jiang Jingwei
---
花了一个多小时，总算把这个博客搭起来了。准备用这个博客用来记录写代码时遇到的坑，以及一些杂七杂八的东西。

这一篇顺便记录一下搭博客的时候遇到的坑。

#### Github CodeSpaces

很久之前注册了`Code Spaces`的测试计划。最近收到了进入测试计划的邮件，就想着用`Code Spcaes`去搭建和更新一个博客，同时也给自己留了个坑。

#### Jekyll

`Jekyll`（不知道怎么念）是github默认的Pages模板。执行`jekyll b`生成静态的网站。

#### 搭建博客的整体思路

整体的思路来自油管上的这个[视频][Creating-a-Blog-with-Hugo-and-Github-in-10-minutes]。
- 创建两个repo。一个保存博客的源码，一个保存由工具生成的静态网站代码。
- 通过`git`的`submodule`功能，在一个文件夹下，同时管理两个repo。

#### 坑

不知什么原因，在`Code Spaces`中，管理`submodule`的时候push总是失败，提示没有权限。

#### 摸爬滚打

- 尝试配置`user.name`和`user.email`
- 尝试配置`ssh key`
- 尝试配置remote url为`user@github.com/reponame.git`

#### 最终的解决方案

只找到了一个能临时解决问题的方案。在`github - settings - developer settings - personal access tokens`中生成一个新的token，然后修改remote url为`token@github.com/reponame.git`

[Creating-a-Blog-with-Hugo-and-Github-in-10-minutes]: https://www.youtube.com/watch?v=LIFvgrRxdt4