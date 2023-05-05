+++
title = "00-文档简介"
description = "Golang 官方文档简介及学习计划"
weight = 1
+++

{{< panel title="文档简介" style="INFO" >}}

Golang官方文档地址为：<a href="https://go.dev/doc/" target="_blank">go.dev/doc</a>；文档开始简单介绍了Golang 的语言特性：简洁、高效、天然并发、快速编译、垃圾回收、反射，新类型系统等等。接下来分按入门，进阶、高级划分了大模块让初学者能够寻寻渐进的深入学习理解Golang；接下来我们将根据 **_Getting Started _**部门一步一步入门Golang。
{{< /panel >}}

## 入门路线
入门教程大致可以分成三个部分：
### 1、环境搭建，预备知识和基础概念。
- 安装Golang环境
  - 介绍Golang下载和环境安装，包括Linux，Mac，Window 系统的安装；

- 快速入门
  - 带你写一个Hello，World。了解一些Go代码、工具、包和模块。

- 使用Go Module
  - 介绍如何使用Go modules；（Go modules 是 Go 语言中正式官宣的项目依赖解决方案；正式于 Go1.14 推荐在生产上使用）
### 2、基础概念和语法
- 如何编写Go代码: How to write Go code
    
  - 学习如何在模块中开发一组简单的Go包，使用Go命令来构建和测试包。
- Go之旅:A Tour of Go 
  - 学习基本语法和数据结构、方法和接口的使用、Go的并发原语

### 3、简单编程体验
- 用Go和Gin开发RESTful API：Tutorial: Developing a RESTful API with Go and Gin 
  - 介绍使用Go和Gin web框架编写RESTful web服务API的基础知识。

- 编写Web应用程序:Writing Web Applications 
  - 构建一个简单的web应用程序。

### 4、新特性和其他补充
- 使用Go泛型：Tutorial: Getting started with generics 
  - 学习如何使用Go语言泛型；(Go 1.18 Go语言增加泛型)
- 模糊测试：Tutorial: Getting started with fuzzing 
  - 学习Go中使用模糊测试（模糊测试是一种通过向目标系统提供非预期的输入并监视异常结果来发现软件漏洞的方法。）
- 使用工作空间：Tutorial: Getting started with multi-module workspaces 
  - 绍在Go中创建和使用多模块工作空间；（Go 1.18增加了工作空间模式，本地做多版本管理很方便）

## 学习计划

- 第一、二单元多为基础概念将以翻译文档为主，补充部分示例代码；
- 第三单元已代码练习为主，已代码练习Golang基本语法
- 第四部分进行简单项目练习


