---
layout: post
title:  "实习日记63(一个开源的小项目infix)"
date:   2021-10-25
categories: jekyll update
---

## Day63

- windows安装golang：

  - 连接：https://golang.org/dl/
  - 有现成的安装包，跟着操作走就完事了，没啥难度
  - go evn或go version查看一下环境或者版本测试一下安装是否成功
  - btw,$GOGROOT和$GOPATH不能设置为同一目录下，因为前者已经包含了标准库中的包，需要与自己通过go get导入的包区分开。

- windows安装dep：

  - dep：golang的一个官方下载工具
  - 下载release版本。打开页面https://github.com/golang/dep/releases，下载最新的[**dep-windows-amd64.exe**](https://github.com/golang/dep/releases/download/v0.4.1/dep-windows-amd64.exe)
  - 将**dep-windows-amd64.exe**放入GOPATH/bin下，修改名称为dep
  - 在cmd中用dep env查看环境测试一下是否安装成功

- infix:尝试失败，使用dep构建项目包的时候cmd一直卡着，唯一能看的资料也就一个github上该项目的README了

  - 后续：我发现是用go get拉包的时候就没拉完整？但是一直会报连接错误，挂了梯子换了代理都一样，可能在cn是拉不下这个东西了吧。

- Dep包管理工具(话说我为啥要去看一下这玩意儿)：

  **状态和流动**：

  - Dep主要围绕“四状态系统”这一模型（一种分类和组织磁盘状态的模型），与包管理器进行交互。尽管这四个状态模型中许多原则都是从现有的包管理器中派生出来的，但还是首先来阐述下这一模型。
    简单的说，四状态是：

    - 当前工程的源代码
    - 描述当前项目需要依赖的文件，在dep中，是Gopkg.toml文件。
    - 一个包含可重复描述的完整依赖关系图，在dep中，是Gopkg.lock文件。
    - 依赖包源代码，在dep的当前设计中，是vendor目录

    如下：

    ![这里写图片描述](https://img-blog.csdn.net/20180509232437310?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2JlbmJlbl8yMDE1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

  **函数式流动**：

  把dep看作一个系统，那么这些系统会对这些状态之间的关系施加一个单向的函数流。这些函数将上述四个状态视为输入和输出，并将它们从左向右移动。具体来说，有两个函数：

  - solving功能，它将当前项目中的导入包和Gopkg.toml中的规则作为输入，不可变的依赖关系图作为传递完成后的输出，形成Gopkg.lock。
  - vendor功能，将Gopkg.lock中的信息作为输入，确保项目编译时能使用在Gopkg.lock文件中锁定的版本。

  具体如下图所示：

  ![这里写图片描述](https://img-blog.csdn.net/20180509232517963?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2JlbmJlbl8yMDE1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

  **dep命令详解**：

  `dep ensure`，这是很典型的流动，当Gopkg.toml文件存在时使用，当你的项目中还没有Gopkg.toml文件时，`dep init`可以帮你生成一个。它们之间的区别在于：`dep init`不是从现有的Gopkg.toml文件读取数据，而是从用户的GOPATH中构建出一个数据，或者从另一个工具中获取元数据文件。也就是说，`dep init`自动将项目从其他途径迁移到当前组织的依赖关系。

  - **命令参数**
    - `-no-vendor`标识会导致只生成一个新的Gopkg.lock文件，而没有形成vendor目录

    - `-vendor-only`标识会跳过生成新的Gopkg.lock文件，导致vendor目录中的包是从已存在的Gopkg.lock中进行读取引入。如下图所示：

      ![这里写图片描述](https://img-blog.csdn.net/20180509232643092?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2JlbmJlbl8yMDE1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

    - `-add`标识是为了简化新的依赖包的引入，然而`-update`在管理项目包时仅限于原路径（例如：github.com/foo/bar），`-add`参数可以将任何包导入路径作为参数（例如：github.com/foo/bar或者github.com/foo/bar/baz）。`dep ensure -add`命令至少会运行下面动作中的一种：

      - 执行solving功能，为新的依赖包生成一个新的`Gopkg.lock`文件。
      - 将版本限制信息写入到`Gopkg.toml`文件。