+++
title = "01-Golang安装与体验"
description = ""
weight = 1
url = "/beginner/01.html"
+++

> 下载并安装按照一下步骤快速进行。
> 有关安装的其他内容，您可能对以下内容感兴趣：
> - [Go多版本安装管理](https://go.dev/doc/manage-install) --如何安装多版本并卸载。
> - [Go 源码安装](https://go.dev/doc/install/source) -- 如何下载源码并在自己的机器上编译安装Go。

下载地址在这里 [go.dev/dl/](https://go.dev/dl/) 各系统版本都有，下载一个与自己系统匹配的版本，可以选最新版本进行安装学习。
## 安装Go
> 选择下面对应的计算机操作系统，然后按照其安装说明进行操作。

### Linux

1. 删除/usr/local/Go文件夹（如果存在），删除以前的Go安装，然后将刚下载的存档提取到/usr/local中，在/usr/local/go:
 {{< code lang="shell" class="pl-4" >}}rm -rf /usr/local/go && tar -C /usr/local -xzf go1.20.4.linux-amd64.tar.gz{{< /code >}} 

<p class="pl-4">（您可能需要以root用户身份或通过sudo运行该命令）。不要将归档文件解压缩到现有的/usr/local/go目录中。这会导致Go安装失败</p>

2. 将/usr/local/go/bin添加到PATH环境变量中

<p class="pl-4">您可以通过将以内容添加到$HOME/.profile或/etc/profile（全局安装）来完成此操作：</p>
{{< code lang="shell" class="pl-4" >}} export PATH=$PATH:/usr/local/go/bin{{< /code >}} 

<p class="pl-4"> **注意**：对配置文件所做的更改可能要等到下次登录计算机时才能应用。要立即应用更改，只需直接运行shell命令，或者使用`source $HOME/.profile` 等命令从概要文件中执行这些命令。
</p>
3. 通过打开命令提示符并输入以下命令来验证是否已安装Go
{{< code lang="shell" class="pl-4" >}}
$ go version
{{< /code >}} 
4. 确认该命令打印已安装的Go版本。

### Mac

1. 打开下载的软件包文件，并按照提示安装Go

该软件包将Go安装到/usr/local/go。该包应该将/usr/local/go/bin目录放在PATH环境变量中。可能需要重新启动任何打开的终端会话才能使更改生效。

2. 通过打开命令提示符并键入以下命令来验证是否已安装Go
> go version

3. 确认该命令打印已安装的Go版本。

### Window
1、打开您下载的MSI文件，并按照提示安装Go。
默认情况下，安装程序将安装Go到 Program Files或Program Files（x86）。您可以根据需要更改位置。安装后，您需要关闭并重新打开任何打开的命令提示符，以便在命令提示符下反映安装程序对环境所做的更改。

2、验证您是否已安装Go；

1. 在Windows中，单击“开始”菜单。
2. 在菜单的搜索框中，键入cmd，然后按Enter键。
3. 在出现的“命令提示符”窗口中，键入以下命令： go version
4. 确认该命令打印已安装的Go版本。
## Reference

- [Installing Go](https://go.dev/doc/install)



