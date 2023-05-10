+++
title = "04-入门多模块工作区"
description = ""
weight = 4
url = "/beginner/04.html"
+++
本教程介绍了 Go 中多模块工作区的基础知识。使用多模块工作区，您可以告诉 Go 命令您正在同时在多个模块中编写代码，并轻松地在这些模块中构建和运行代码。
在本教程中，您将在共享的多模块工作区中创建两个模块，对这些模块进行更改，并在构建中查看这些更改的结果。
## 前期准备

- 安装 Go 1.18 或更高版本。
- 一个编辑代码的工具。
- 一个命令终端。

本教程需要** go1.18** 或更高版本。使用go.dev/dl中的链接确保您已在 Go 1.18 或更高版本中安装了 Go 。
## 创建一个模块
首先，为您要编写的代码创建一个模块。
一、打开命令提示符并切换到您的主目录。
```shell
# Linux 或 Mac：
$ cd
# Windows：
C:\> cd %HOMEPATH%
```
>  本教程的其余部分将显示 $ 作为提示符。您使用的命令也适用于 Windows。

二、在命令提示符下，创建workspace目录。
```shell
$ mkdir workspace
$ cd workspace
```

三、初始化模块
> 本示例将创建一个依赖 golang.org/x/example 模块的hello 模块。

1、创建 hello 模块：
```shell
$ mkdir hello
$ cd hello
$ go mod init example.com/hello
go: creating new go.mod: module example.com/hello
```
2、使用 go get  添加 golang.org/x/example 依赖、。
```shell
$ go get golang.org/x/example
```
3、在 hello 目录下创建 hello.go，内容如下：
```go
package main

import (
    "fmt"

    "golang.org/x/example/stringutil"
)

func main() {
    fmt.Println(stringutil.Reverse("Hello"))
}
```
4、运行 hello 程序：
```shell
$ go run example.com/hello
olleH
```
## 创建工作区
> 在此步骤中，我们将创建一个go.work文件来指定模块的工作区。

### 初始化工作区
在workspace目录中，运行：
```shell
$ go work init ./hello
```
该go work init命令会生成go.work文件从而创建一个包含  ./hello 目录的工作空间

该go命令生成一个go.work如下所示的文件：
```go
go 1.18

use ./hello
```

该go.work文件的语法与go.mod 类似
第一行 go 1.18  告诉 Go 应使用哪个版本的 Go 编译文件，与go.mod文件go.mod 。
第二行 use ./hello 告诉 Go 在编译时 hello 目录的模块应该是主模块
所以在任何子目录下的workspace模块都会被激活。
### 运行工作区目录下的程序
在workspace目录中，运行：
```shell
$ go run example.com/hello
olleH
```

Go 命令包括工作区中的所有模块作为主模块。这允许我们引用模块中he模块外的包。在模块或工作区外运行该go run命令会报错，因为该go命令不知道要使用哪些模块。

接下来，我们将模块的本地golang.org/x/example添加到工作区。然后我们将向stringutil包中添加一个新函数，我们可以使用它来代替Reverse.
### 下载并修改golang.org/x/example模块
在此步骤中，我们将下载包含该模块的 Git 存储库的副本golang.org/x/example，将其添加到工作区，然后向其中添加我们将在 hello 程序中使用的新函数。
克隆存储库
1、在工作区目录中，运行git命令以克隆存储库：
```shell
$ git clone https://go.googlesource.com/example
Cloning into 'example'...
remote: Total 165 (delta 27), reused 165 (delta 27)
Receiving objects: 100% (165/165), 434.18 KiB | 1022.00 KiB/s, done.
Resolving deltas: 100% (27/27), done.
```
2、将模块添加到工作区
```shell
$ go work use ./example
```
该go work use命令将一个新模块添加到 go.work 文件中。如下：
```go
go 1.18

use (
    ./hello
    ./example
)
```
该模块现在包括模块example.com/hello和golang.org/x/example模块。
我们将在本地stringutil模块中编写的新代码，而不是使用go get命令下载的模块版本。

3、添加新功能。
添加一个将字符串转大写的新函数到golang.org/x/example/stringutil包中。
在 workspace/example/stringutil 目录中创建一个toupper.go；写入一下代码：
```go
package stringutil

import "unicode"

// ToUpper uppercases all the runes in its argument string.
func ToUpper(s string) string {
    r := []rune(s)
    for i := range r {
        r[i] = unicode.ToUpper(r[i])
    }
    return string(r)
}
```

4、修改 hello 程序以使用该函数。
修改内容为workspace/hello/hello.go包含以下内容：
```go
package main

import (
    "fmt"

    "golang.org/x/example/stringutil"
)

func main() {
    fmt.Println(stringutil.ToUpper("Hello"))
}
```
#### 在工作区运行代码
从工作区目录，运行
```go
$ go run example.com/hello
HELLO
```
Go命令在go.work文件指定的目录example.com/hello下查找命令行指定的模块 ，同样使用go.work文件解析导入golang.org/x/example
go.work可以替换 replace 指令来跨多个模块工作。
由于这两个模块位于同一个工作区中，因此很容易在一个模块中进行更改并在另一个模块中使用它。
#### 更进一步
现在，要正确发布这些模块，我们需要发布模块golang.org/x/example ，例如在v0.1.0. 这通常是通过在模块的版本控制存储库上标记提交来完成的。 有关更多详细信息，请参阅 [模块发布工作流程文档](https://go.dev/doc/modules/release-workflow)。发布完成后，我们可以在hello/go.mod中增加对 golang.org/x/example的依赖：
```shell
cd hello
go get golang.org/x/example@v0.1.0
```

这样，该go命令就可以正确解析工作区外的模块。

### 了解有关工作区的更多信息
除了我们在本教程前面看到的go work init之外，该go命令还有几个用于处理工作区的子命令：

- go work use [-r] [dir] ：如果文件存在，则为go.work文件添加一个use dir指令，如果参数目录不存在，则删除该目录。-r表示递归地检查子目录。
- go work edit：编辑go.work文件类似于go mod edit
- go work sync：将工作区构建列表中的依赖项同步到每个工作区模块中。

