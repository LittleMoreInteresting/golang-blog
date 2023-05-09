
+++
title = "03-创建Go项目"
description = ""
weight = 3
url = "/beginner/03.html"
+++
这是教程的第一部分，介绍了Go语言的一些基本功能。如果你刚开始使用Go语言，一定要看一下教程：Go入门，它介绍了Go命令、Go模块和非常简单的Go代码。
在本教程中，您将创建两个模块。第一种是用来被其他库或应用导入。第二个模块在程序中调用第一个模块的方法。
本教程包括七个小的部分：

1. 创建一个模块：编写一个小模块，其中包含可以从另一个模块调用的函数。
2. 从另一个模块调用您的代码：导入并使用您的新模块。
3. 返回并处理错误：添加简单的错误处理。
4. 返回一个随机的问候语：处理切片（slices）中的数据（slices：Go的动态大小数组）。
5. 为多个人返回问候语：将键/值对存储在映射（map）中。
6. 添加测试：使用Go内置的单元测试功能来测试代码。
7. 编译并安装应用程序：在本地编译并安装。
### 前期准备

- 一些编程经验。这里的代码非常简单，但了解函数、循环和数组会有所帮助。
- 用于编辑代码的工具。
- 一种命令终端。Go在Linux和Mac上以及Windows中的PowerShell或cmd上使用任何终端都能很好地工作。
### 开始创建模块
首先创建Go模块。我们通常会把一些包含同一类功能函数的包放在同一模块中，例如，您可以创建模块，模块的包具有进行财务分析的功能，以便其他编写财务应用程序的人可以使用您的代码。有关开发模块的更多信息，请参阅[开发和发布模块](https://go.dev/doc/modules/developing).。
Go语言中 代码，包(package)，模块（module）之间的关系是:代码组成包，包组成模块。您开发的模块需要指定运行代码所需的依赖项，包括Go版本及其所需的其他模块。
当您在模块中添加或改进功能时，您将发布模块的新版本。编写调用这个模块的开发人员可以导入新版本模块，并在将其上线之前使用新版本进行测试。

1. 打开命令行，cd到home目录（或其他存放代码的目录）
2. 新建目录 greetings 用来存放Go代码
3. 使用 [go mod init](https://go.dev/ref/mod#go-mod-init) 命令初始化一个module

执行 go mod init <module path> 命令；本例中我们用  example.com/greetings；如果您发布模块，这必须是Go工具可以下载模块的路径，那就是本模块代码仓库的路径（例如：github.com/example）；
```shell
go mod init example.com/greetings
# go: creating new go.mod: module example.com/greetings
```
go mod init 命令创建一个go.mod文件来跟踪代码的依赖关系。到目前为止，该文件只包括模块的名称和代码支持的Go版本。但当你添加依赖项时，go.mod文件会列出你的代码所依赖的版本。这可以保持构建的可复制性，并让你直接控制要使用的模块版本。

4. 在编辑器中新建 greetings.go 文件
5. 在greetings.go文件中写入一下代码并保存
```go
package greetings

import "fmt"

// Hello returns a greeting for the named person.
func Hello(name string) string {
    // Return a greeting that embeds the name in a message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message
}
```
这是您的模块的第一个代码。它会向任何要求调用者返回问候语。下一步我们将调用此函数。
代码解读：

- 声明greetings包存放相关的功能方法
- 实现Hello功能来返回问候语

此函数接受字符串类型的name参数。该函数返回一个字符串。在Go中，名称以大写字母开头的函数可以由不在同一个包中的函数调用。这在Go中被称为导出名称。
![](/img/03/function-syntax.png)

- 声明一个用于保存问候语的message变量

在Go中，:= 运算符是在一行中声明和初始化变量的快捷方式（Go使用右侧的值来确定变量的类型）。完整格式可以这样写：
```go
var message string
message = fmt.Sprintf("Hi, %v. Welcome!", name)
```

- 使用fmt包的Sprintf函数可以创建一条问候消息。第一个参数是一个格式字符串，Sprintf将name参数的值替换%v。插入name参数的值将完成问候语
- 将格式化的问候语文本返回给调用者

在下一步中，您将从另一个模块调用此函数。
### 调用模块
在上一节中，您创建了一个greetings模块。在本节中，您将编写代码来调用刚刚编写的模块中的Hello函数。您将编写可执行的代码，并调用问候语模块中的代码。

1. 创建一个hello目录编写调用代码

创建此目录后，您应该在层次结构的同一级别同时拥有hello和greetings目录，如下所示：
```shell
<home>/
 |-- greetings/
 |-- hello/
```

2. 为即将编写的代码启用依赖项跟踪。
```shell
go mod init example.com/hello
```

3. 在文本编辑器的hello目录中，创建hello.go文件。
4. 编写代码调用Hello函数，打印返回值。
```go
package main

import (
    "fmt"

    "example.com/greetings"
)

func main() {
    // Get a greeting message and print it.
    message := greetings.Hello("Gladys")
    fmt.Println(message)
}
```
代码解读：

- 声明一个主程序包(main)。在Go中，作为应用程序执行的代码必须在主包中。
- 导入两个包：example.com/greetings和fmt包。这使您的代码可以访问这些包中的函数。导入example.com/greetings（包含在您之前创建的模块中的包）可以访问Hello函数。您还可以导入fmt，它具有处理输入和输出文本的功能（例如将文本打印到控制台）。
- 通过调用greetings包的Hello 方法来获得返回值。

5. 编辑example.com/hello模块以使用本地example.com/hellos

在生产环境，Go工具可以从远程代码仓库中下载得到example.com/helles模块。目前，由于您尚未发布该模块，您需要调整example.com/hello模块的依赖关系，以便它可以在本地找到example.com/hellos代码。
用go mod edit命令编辑example.com/hello模块，将go工具从其模块路径（模块不在的地方）重定向到本地目录（模块所在的地方）。
a、在hello目录中的命令提示符下，运行以下命令
```shell
go mod edit -replace example.com/greetings=../greetings
```
该命令指定将依赖example.com/greetings应替换为/greetings。运行该命令后，hello目录中的go.mod文件应包含一个replace指令：
```go
module example.com/hello

go 1.16

replace example.com/greetings => ../greetings
```
b、在hello目录中的命令提示符下，运行go mod tidy 命令来同步example.com/hello模块的依赖项，添加代码所需但尚未在模块中加载的依赖项。
```shell
$ go mod tidy
go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000
```
命令完成后，example.com/hello模块的go.mod文件应该如下所示：
```go
module example.com/hello

go 1.16

replace example.com/greetings => ../greetings

require example.com/greetings v0.0.0-00010101000000-000000000000
```
该命令在greetings目录中找到了本地代码，然后添加了一个require指令，指定example.com/hello需要example.com/greetings。hello.go中导入greetings包时创建了此依赖项。
模块路径后面的数字是一个伪版本号,是自动生成的代替版本号的数字（模块实际还没有）。
要引用已发布的模块，go.mod文件通常会省略replace指令，并使用末尾带有标记版本号的require指令。
```go
require example.com/greetings v1.1.0
```

6. 在hello目录中的命令提示符下，运行代码以确认它是否工作
```shell
$ go run .
Hi, Gladys. Welcome!
```
恭喜！您已经编写了两个功能模块。在下节内容是错误处理。
### 返回并处理错误
处理错误是健壮代码的一个基本特征。在本节中，您将添加一些代码，从greetings 模块返回一个错误，然后在调用者中进行处理。

1. 在 greetings/greetings.go 中添加代码:

如果你不知道该向谁打招呼，那么回电是没有意义的。如果name参数为空，则向调用方返回一个错误。
```go
package greetings

import (
    "errors"
    "fmt"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return "", errors.New("empty name")
    }

    // If a name was received, return a value that embeds the name
    // in a greeting message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message, nil
}
```
代码解析：

- 更改函数，使其返回两个值：一个string和一个error。调用者通过第二个返回值判断是否发生错误 (任何Go函数都可以返回多个值。有关更多信息，请参阅 [Effective Go](https://go.dev/doc/effective_go.html#multiple-returns).)
- 导入Go标准库error包，使用 errors.New 函数。
- 添加if语句判断无效请求（name为空），如果请求无效则返回error。errors.New 函数返回一个错误，其中包含错误消息。
- 在成功返回中添加nil（表示没有错误）作为第二个返回值。这样，调用者就可以判断函数执行成功了。
2. 在hello/hello.go文件中，处理hello函数返回的error以及非error值

将以下代码添加到hello.go。
```go
package main

import (
    "fmt"
    "log"

    "example.com/greetings"
)

func main() {
    // Set properties of the predefined Logger, including
    // the log entry prefix and a flag to disable printing
    // the time, source file, and line number.
    log.SetPrefix("greetings: ")
    log.SetFlags(0)

    // Request a greeting message.
    message, err := greetings.Hello("")
    // If an error was returned, print it to the console and
    // exit the program.
    if err != nil {
        log.Fatal(err)
    }

    // If no error was returned, print the returned message
    // to the console.
    fmt.Println(message)
}
```

代码解析：

- 将日志包配置为在其日志消息的开头打印命令名（“greetings：”），不带时间戳或源文件信息。
- 将两个Hello返回值（包括错误）分配给变量。
- 将Hello参数从Gladys的名称更改为空字符串，这样您就可以测试错误处理代码。判断 error 不等于 - nil，在这种情况下继续下去是没有意义。
- 使用标准库的log包中的函数来输出错误信息。如果出现错误，则使用日志包的Fatal函数打印错误并停止程序。
3. 在hello目录的命令行中，运行hello.go以确认代码是否有效
```shell
$ go run .
# greetings: empty name
# exit status 1
```
这是Go中常见的错误处理：将错误作为值返回，以便调用方可以检查它。

### 随机返回问候语
在本节中，您将更改代码，使其不再每次返回一个问候语，而是返回几个预定义的问候语消息中的一个。
为此，您将使用Go切片(slice)。切片就像一个数组，只是它的大小随着添加和删除项目而动态变化。切片是Go语言最有用的类型之一。
添加一个slice 存放三条问候消息，然后让代码随机返回其中一条消息。

1. 在问候语/问候语.go中，更改代码，如下
```go
package greetings

import (
    "errors"
    "fmt"
    "math/rand"
    "time"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return name, errors.New("empty name")
    }
    // Create a message using a random format.
    message := fmt.Sprintf(randomFormat(), name)
    return message, nil
}

// init sets initial values for variables used in the function.
func init() {
    rand.Seed(time.Now().UnixNano())
}

// randomFormat returns one of a set of greeting messages. The returned
// message is selected at random.
func randomFormat() string {
    // A slice of message formats.
    formats := []string{
        "Hi, %v. Welcome!",
        "Great to see you, %v!",
        "Hail, %v! Well met!",
    }

    // Return a randomly selected message format by specifying
    // a random index for the slice of formats.
    return formats[rand.Intn(len(formats))]
}
```
代码解析：

- 添加一个randomFormat函数，该函数可为问候语返回随机选择的格式。请注意，randomFormat以小写字母开头，使其只能由自己包中的代码访问（换句话说，它不导出）。
- 在randomFormat中，用三种消息格式声明一个formats切片。在声明切片时，可以省略括号中的大小，如下所示：[]string。这告诉Go，片下面的数组大小可以动态更改。
- 使用math/rand包生成一个随机数，用于从切片(slice)中选择项目。
- 添加一个init函数，用当前时间为rand包设定种子。Go在初始化全局变量后，在程序启动时自动执行init函数。
- 在Hello中，调用randomFormat函数来获取要返回的消息的格式，然后将格式和名称值一起使用来创建消息。
- 像以前一样返回消息（或错误）。

2. 在hello/hello.go中，更改代码，如下所示。

只将Gladys的名字（或者其他名字，如果您愿意的话）作为参数添加到Hello.go中的Hello函数调用中。
```go
package main

import (
    "fmt"
    "log"

    "example.com/greetings"
)

func main() {
    // Set properties of the predefined Logger, including
    // the log entry prefix and a flag to disable printing
    // the time, source file, and line number.
    log.SetPrefix("greetings: ")
    log.SetFlags(0)

    // Request a greeting message.
    message, err := greetings.Hello("Gladys")
    // If an error was returned, print it to the console and
    // exit the program.
    if err != nil {
        log.Fatal(err)
    }

    // If no error was returned, print the returned message
    // to the console.
    fmt.Println(message)
}
```

3. 在hello目录的命令行中，运行hello.go以确认代码是否有效。多次运行它，注意问候语变化。
```shell
$ go run .
Great to see you, Gladys!

$ go run .
Hi, Gladys. Welcome!

$ go run .
Hail, Gladys! Well met!
```
### 为多人回复问候
在您将对模块代码进行的最后一次更改中，您将添加对在一个请求中获得多人问候的支持。换句话说，您将处理多值输入，然后将该输入中的值与多值输出配对。要做到这一点，您需要将一组名称传递给一个函数，该函数可以为每个名称返回一个问候语。
但有一个问题。将Hello函数的参数从单个名称更改为一组名称将更改函数的签名。如果您已经发布了example.com/greetings模块，并且用户已经编写了调用Hello的代码，那么这种更改将破坏他们的程序。
在这种情况下，更好的选择是用不同的名称编写一个新函数。新功能将采用多个参数。这保留了旧功能以实现向后兼容性。

1. 在greetings/greetings.go中，更改代如下所示。

```go
package greetings

import (
    "errors"
    "fmt"
    "math/rand"
    "time"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return name, errors.New("empty name")
    }
    // Create a message using a random format.
    message := fmt.Sprintf(randomFormat(), name)
    return message, nil
}

// Hellos returns a map that associates each of the named people
// with a greeting message.
func Hellos(names []string) (map[string]string, error) {
    // A map to associate names with messages.
    messages := make(map[string]string)
    // Loop through the received slice of names, calling
    // the Hello function to get a message for each name.
    for _, name := range names {
        message, err := Hello(name)
        if err != nil {
            return nil, err
        }
        // In the map, associate the retrieved message with
        // the name.
        messages[name] = message
    }
    return messages, nil
}

// Init sets initial values for variables used in the function.
func init() {
    rand.Seed(time.Now().UnixNano())
}

// randomFormat returns one of a set of greeting messages. The returned
// message is selected at random.
func randomFormat() string {
    // A slice of message formats.
    formats := []string{
        "Hi, %v. Welcome!",
        "Great to see you, %v!",
        "Hail, %v! Well met!",
    }

    // Return one of the message formats selected at random.
    return formats[rand.Intn(len(formats))]
}
```

代码解析:

- 添加一个Hellos函数，该函数的参数是名称slice，而不是一个单独的名称。此外，您还可以将其返回类型从字符串更改为map，以便返回名称与问候语的映射。
- 让新的Hellos函数调用现有的Hello函数。这有助于减少重复，同时保留两个功能。
- 创建一个map 变量messages，将每个接收到的名称（作为key）与生成的消息（作为Value）关联起来。Go中可以使用以下语法初始化映射：make（map[key-type]value-type）。您可以使用Hellos函数将此映射返回给调用者。
- 循环查看函数接收到的names，检查每个name是否都有非空值，然后将消息与每个名称关联起来。在这个for循环中，range返回两个值：循环中当前项的索引和项值的副本。索引值用不到，用Go blank标识符（下划线）来忽略它。

2. 在hello/hello.go调用代码中，传入names 切片，然后打印返回的name/message映射的内容。

在hello.go中，更改代码，如下所示。
```go
package main

import (
    "fmt"
    "log"

    "example.com/greetings"
)

func main() {
    // Set properties of the predefined Logger, including
    // the log entry prefix and a flag to disable printing
    // the time, source file, and line number.
    log.SetPrefix("greetings: ")
    log.SetFlags(0)

    // A slice of names.
    names := []string{"Gladys", "Samantha", "Darrin"}

    // Request greeting messages for the names.
    messages, err := greetings.Hellos(names)
    if err != nil {
        log.Fatal(err)
    }
    // If no error was returned, print the returned map of
    // messages to the console.
    fmt.Println(messages)
}
```
代码解析:

- 创建一个names 切片，包含三个name。
- 将names变量作为参数传递给Hellos函数。

3. 在命令行中，切换到包含hello/hello.go的目录，然后使用go run确认代码是否有效。

输出应该是将name与message关联起来的map，如下所示：
```shell
$ go run .
map[Darrin:Hail, Darrin! Well met! Gladys:Hi, Gladys. Welcome! Samantha:Hail, Samantha! Well met!]
```
本主题介绍了用于表示名称/值对的映射。它还引入了通过为模块中的新功能或更改的功能实现新功能来保持向后兼容性的想法。

### 添加测试
现在您已经将代码放到了一个固定的位置，添加一个测试。在开发过程中测试代码可能会暴露出在您进行更改时出现的错误。在本章节中，您将为 Hello 函数添加一个测试。
Go 内置的对单元测试的支持使得在进行测试时更容易。具体来说，使用命名约定、Go 的 testing 包和 Go test 命令，您可以快速编写和执行测试。

1. 在 greetings 目录中，创建一个名为 greetings_test.go 的文件。

以_test.go 结尾的文件名告诉 go test 命令此文件包含测试函数。

2. 在 greeting_test.go 中，添加以下代码并保存文件。
```go
package greetings

import (
    "testing"
    "regexp"
)

// TestHelloName calls greetings.Hello with a name, checking
// for a valid return value.
func TestHelloName(t *testing.T) {
    name := "Gladys"
    want := regexp.MustCompile(`\b`+name+`\b`)
    msg, err := Hello("Gladys")
    if !want.MatchString(msg) || err != nil {
        t.Fatalf(`Hello("Gladys") = %q, %v, want match for %#q, nil`, msg, err, want)
    }
}

// TestHelloEmpty calls greetings.Hello with an empty string,
// checking for an error.
func TestHelloEmpty(t *testing.T) {
    msg, err := Hello("")
    if msg != "" || err == nil {
        t.Fatalf(`Hello("") = %q, %v, want "", error`, msg, err)
    }
}
```

代码解析:

- 在与您正在测试的代码相同的包中实现测试函数。
- 创建两个测试函数来测试 greetings.Hello 函数。测试函数名称的形式为 TestName，其中 Name 表示特定测试的内容。此外，测试函数将指向测试包的 testing.T 类型的指针作为参数。您可以使用此参数的方法从测试中进行报告和日志记录。
- 执行两次测试 
   - TestHelloName 调用 Hello 函数，传递一个名称值，该函数应该能够使用该名称值返回有效的响应消息。如果调用返回错误或意外响应消息（其中不包括您传入的名称），则使用 t 参数的 Fatalf 方法将消息打印到控制台并结束执行。
   - TestHelloEmpty 使用一个空字符串调用 Hello 函数。此测试旨在确认您的错误处理是否有效。如果调用返回非空字符串或没有错误，则使用 t 参数的 Fatalf 方法将消息打印到控制台并结束执行。

3. 在问候目录的命令行中，运行 go test 命令来执行测试。
go test 命令在测试文件（名称以\test.go 结尾）中执行测试函数（名称以 test 开头）。您可以添加-v 标志来获得详细的输出，其中列出了所有测试及其结果。

```shell
$ go test
PASS
ok example.com/greetings 0.364s

$ go test -v
=== RUN TestHelloName
--- PASS: TestHelloName (0.00s)
=== RUN TestHelloEmpty
--- PASS: TestHelloEmpty (0.00s)
PASS
ok example.com/greetings 0.372s
```

4. 打断 greetings.Hello 函数用于查看未通过的测试。
TestHelloName 测试函数检查您指定为 Hello 函数参数的名称的返回值。要查看失败的测试结果，请更改 greetings.Hello 函数，使其不再包含名称。
在 greetings/greetings.go 中，粘贴以下代码来代替 Hello 函数。请注意，高亮显示的行会更改函数返回的值，就好像名称参数被意外删除了一样。

```go
// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
// If no name was given, return an error with a message.
if name == "" {
return name, errors.New("empty name")
}
// Create a message using a random format.
// message := fmt.Sprintf(randomFormat(), name)
message := fmt.Sprint(randomFormat())
return message, nil
}
```

5. 在 greetings 目录的命令行中，运行 go test 来执行测试。
这一次，在没有-v 标志的情况下运行 go test。输出将只包括失败测试的结果，当您有很多测试时，这可能很有用。TestHelloName 测试应该失败—— TestHelloEmpty 仍然通过。
```shell
$ go test
--- FAIL: TestHelloName (0.00s)
greetings_test.go:15: Hello("Gladys") = "Hail, %v! Well met!", <nil>, want match for `\bGladys\b`, nil
FAIL
exit status 1
FAIL example.com/greetings 0.182s
```

### 编译并安装应用
在最后一个主题中，您将学习几个新的 go 命令。虽然 go run 命令是在频繁更改时编译和运行程序的有用快捷方式，但它不会生成二进制可执行文件。

本主题介绍了用于构建代码的两个附加命令:

- go-build 命令编译包及其依赖项，但不安装结果。
- go install 命令编译并安装程序包。

1. 在 hello 目录的命令行中，运行 go build 命令将代码编译为可执行文件。
```shell
$ go build
```

2. 在 hello 目录中的命令行中，运行新的 hello 可执行文件以确认代码是否有效。
请注意，您的结果可能会有所不同，这取决于您在测试后是否更改了 greetings.go 代码。

-  Linux or Mac:
```shell
$ ./hello

map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
```

- On Windows:
```shell
$ hello.exe
map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hail, Samantha! Well met!]
```

您已将应用程序编译为可执行文件，以便运行它。但要当前运行它，您的提示需要位于可执行文件的目录中，或者指定可执行文件路径。
接下来，您将安装可执行文件，这样您就可以在不指定其路径的情况下运行它。

3. 查找 Go 安装路径，Go 命令将在其中安装当前软件包。
您可以通过运行 go list 命令来发现安装路径，如以下示例所示：

```shell
$ go list -f '{{.Target}}'
```

例如，命令的输出可能会说/home/gopher/bin/hello，这意味着二进制文件被安装到/home/gopaher/bin。在下一步中，您将需要此安装目录。

4. 将 Go 安装目录添加到系统的 shell 路径
这样，您就可以运行程序的可执行文件，而无需指定可执行文件的位置。

在 Linux 或 Mac 上，运行以下命令:
```shell
$ export PATH=$PATH:/path/to/your/install/directory
```

在 Windows, 运行以下命令:

```shell
$ set PATH=%PATH%;C:\path\to\your\install\directory
```

另一种选择是，如果您的 shell 路径中已经有一个类似$HOME/bin 的目录，并且您想在那里安装 Go 程序，则可以使用 Go-env 命令设置 GOBIN 变量来更改安装目标：

```shell

$ go env -w GOBIN=/path/to/your/bin
# or

$ go env -w GOBIN=C:\path\to\your\bin
```

5. 更新完 shell 路径后，运行 go install 命令来编译和安装包。

```shell
$ go install
```

6. 只需输入应用程序的名称即可运行应用程序。为了让这变得有趣，打开一个新的命令提示符，并在其他目录中运行 hello 可执行文件名。
```shell
$ hello
map[Darrin:Hail, Darrin! Well met! Gladys:Great to see you, Gladys! Samantha:Hail, Samantha! Well met!]
```
### 
