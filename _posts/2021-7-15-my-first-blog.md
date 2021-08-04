---
layout: post
title:  "我的第一篇博客"
date:   2021-07-15
categories: jekyll update
---

#### 我的第一篇博客记录一下使用jekyll更新个人博客的教程吧

### 1.安装jekll

#### 1.1安装ruby+Devkit

前往官网https://rubyinstaller.org/downloads/下载RubyInstaller with devkit，这里使用的是2.7.4版本，安装完后finish页面的那个可以不选，但msys2一定要安装

#### 1.2验证ruby和gem是否安装

输入ruby -v和gem -v有版本号即可，一般来说如果安装了wiht devkit的RubyInstaller好像无需手动装RubyGem

#### 1.3安装jekyll

gem install jekyll，安装好后输入jekyll -version验证即可

### 2.创建一个新的项目

#### 2.1自己创建新的项目

输入jekyll new myblog，如果卡住则ctrl+c停止批处理，然后把新创建的项目文件中的source更改即可

#### 2.2使用模板创建新的项目

推荐网址：http://jekyllthemes.org

#### 2.3运行项目

在项目根目录使用jekyll server即可运行项目，默认4000端口

#### 2.4更改博客设置

在_config.yml中可自定义博客基础设置

### 3.上传帖子

#### 3.1上传新帖子

如果使用本模板，需要保证在帖子顶部有相应的YAML Front Matter，然后把md文件放入_posts目录即可，保证md文件名中不出现中文

#### 3.2md文件规范

本人使用的是Typora来写文件，Typora自带的有序列表似乎在页面显示的时候会出现问题，推荐自己实现或使用无序列表