+++
title = "02-新手入门"
description = ""
weight = 2
url = "/beginner/02.html"
+++

> 在本教程中，您将获得Go编程的简要介绍
> - 安装Go .
> - 写一些简单的 "Hello, world" .
> - 使用go命令运行代码.
> - 使用Go程序包发现工具查找可以在自己的代码中使用的程序包.
> - 调用外部模块的功能.


## 前期准备

- **一些编程经验**。 这里的代码非常简单，但了解一些函数相关的知识会有所帮助。.
- **用于编辑代码的工具**。** ** 任何文本编辑器都可以。大多数文本编辑器都很好地支持Go。最受欢迎的是VSCode（免费）、GoLand（付费）和Vim（免费）。
- **一个命令终端.** Go在Linux和Mac上以及Windows中的PowerShell或cmd上使用任何终端都能很好地工作。

## Hello World
按一下步骤写一段 “Hello World”代码

1、打开一个命令提示符并cd到您的主目录。
Linux/Mac 系统执行 : `cd`
Window系统执行： `cd %HOMEPATH%`

2、为您的第一个Go源代码创建一个hello目录
可以使用一下命令：

```shell
 mkdir hello
 cd hello
```

3、为代码启用依赖管理
<p style="text-indent: 2em">
当您的代码导入其他项目中的包（package）时，您可以通过自己的代码来管理这些模块的依赖关系。该模块由go.mod文件定义，通过该文件追踪提供包的这些模块。该go.mod文件与您的代码一起保存，包括在您的源代码存储库中。
要通过创建go.mod文件为代码启用依赖管理，请运行go mod init+模块的名称 命令。该名称是模块的模块路径。
在实际开发中，模块路径通常是保存源代码的存储仓库位置。例如，模块路径可能是github.com/mymodule。如果您计划发布您的模块供他人使用，则模块路径必须是Go工具可以下载您的模块的位置。
对于本教程，只需使用example/hello。</p>

```shell
 go mod init example/hello
 go: creating new go.mod: module example/hello
```

4、在编辑器中，创建一个文件hello.go，在其中编写代码。
5、将以下代码粘贴到hello.go文件中并保存该文件。
```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello World（^-^）")
}
```
接下来看一下这段代码的内容：

- 声明一个主包(main)（包是对函数进行分组的一种方式，它由同一目录中的所有文件组成）。
- 导入流行的fmt包，其中包含格式化文本的功能，包括打印到控制台。这个包是Go的标准库之一。
- 实现一个主要功能，将消息打印到控制台。当您运行主程序包时，默认情况下会执行一个主函数（main）。

6、运行代码
```shell
go run .
# Hello World（^-^）
```
7、go run命令是众多go命令之一。使用 `go help `命令可以查看其他命令帮助文档：
## 调用外部包
<p style="text-indent: 2em">
当你需要你的代码来做一些可能已经被其他人实现的事情时，你可以导入一个实现了这些功能的包直接使用。</p>
1、使用外部模块的功能，使打印的消息更加有趣

- 访问pkg.go.dev并搜索“quote”包
- 在搜索结果中找到并单击rsc.io/quote包（如果您看到rsc.io/quote/v3，请暂时忽略它）。
- 在“文档”部分的“索引”下，记下可以从代码中调用的函数列表。您将使用Go功能。
- 请注意，在本页顶部，"quote"包含在rsc.io/quote模块中。
<p style="text-indent: 2em">
 您可以使用pkg.go.dev网站查找已发布的模块，这些模块的包中有您可以在自己的代码中使用的功能。包发布在模块中，比如rsc.io/quote，其他人可以在其中使用它们。随着时间的推移，新版本会对模块进行改进，您可以升级代码以使用改进的版本。
</p>
2、在Go代码中，导入rsc.io/quote包并调用其函数 quote.Go()。之后，您的代码应该包括以下内容：

```go
package main

import "fmt"

import "rsc.io/quote"

func main() {
    fmt.Println(quote.Go())
}
```
3、添加模块依赖和校验
Go程序把“quote”包添加到依赖中（go.mod），以及用于验证模块的go.sum文件中。
```shell
 $ go mod tidy
 go: finding module for package rsc.io/quote
 go: found rsc.io/quote in rsc.io/quote v1.5.2
```

4、运行您的代码以查看您正在调用的函数生成的消息。
```shell
 $ go run .
 # Don't communicate by sharing memory, share memory by communicating.
```
请注意，您的代码调用Go函数，打印出一条关于通信的格言。
> 当您运行go mod tidy，它找到并下载了包含您导入的包的rsc.io/quote模块。默认情况下，它下载了最新版本v1.5.2。

