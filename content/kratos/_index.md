+++
title = "Kratos 实践"
description = "Kratos 一套轻量级 Go 微服务框架，包含大量微服务相关框架及工具。"
weight = 21
url = "/kratos/"
+++

{{< panel title="Kraots简介" style="info" >}}
 Kratos 一套轻量级 Go 微服务框架，包含大量微服务相关框架及工具。

## 原则
 - 简单：不过度设计，代码平实简单；
 - 通用：通用业务开发所需要的基础库的功能；
 - 高效：提高业务迭代的效率；
 - 稳定：基础库可测试性高，覆盖率高，有线上实践安全可靠；
 - 健壮：通过良好的基础库设计，减少错用；
 - 高性能：性能高，但不特定为了性能做 hack 优化，引入 unsafe ；
 - 扩展性：良好的接口设计，来扩展实现，或者通过新增基础库目录来扩展功能；
 - 容错性：为失败设计，大量引入对 SRE 的理解，鲁棒性高；
 - 工具链：包含大量工具链，比如 cache 代码生成，lint 工具等等；

## 特性
 - APIs：协议通信以 HTTP/gRPC 为基础，通过 Protobuf 进行定义；
 - Errors：通过 Protobuf 的 Enum 作为错误码定义，以及工具生成判定接口；
 - Metadata：在协议通信 HTTP/gRPC 中，通过 Middleware 规范化服务元信息传递；
 - Config：支持多数据源方式，进行配置合并铺平，通过 Atomic 方式支持动态配置；
 - Logger：标准日志接口，可方便集成三方 log 库，并可通过 fluentd 收集日志；
 - Metrics：统一指标接口，可以实现各种指标系统，默认集成 Prometheus；
 - Tracing：遵循 OpenTelemetry 规范定义，以实现微服务链路追踪；
 - Encoding：支持 Accept 和 Content-Type 进行自动选择内容编码；
 - Transport：通用的 HTTP/gRPC 传输层，实现统一的 Middleware 插件支持；
 - Registry：实现统一注册中心接口，可插件化对接各种注册中心；

{{< /panel >}}

{{< childpages >}}