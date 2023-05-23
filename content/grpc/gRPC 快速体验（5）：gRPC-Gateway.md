+++
title = "五、gRPC-Gateway"
description = "gRPC 是 Google 开发的一款高性能、开源的远程过程调用（RPC）框架。它使用 Protocol Buffers 作为接口定义语言（IDL），可以在多种编程语言和平台之间进行通信。"
weight = 5
url = "/grpc/05.html"
+++
# 简介
> gRPC-Gateway是protoc的一个插件。它读取gRPC服务定义并生成反向代理服务器，该服务器将RESTful JSON API转换为gRPC。此服务器根据gRPC定义中的自定义选项生成。 

[grpc-gateway文档](https://grpc-ecosystem.github.io/grpc-gateway/)
![](https://cdn.nlark.com/yuque/0/2022/svg/12487795/1662001502595-799f0c0c-c9d4-40f5-ac1d-5616856a30d5.svg#clientId=uef1d0fb0-3fe7-4&from=paste&id=u46f7a2cf&originHeight=760&originWidth=1120&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ufb7b917c-de12-448e-b5fb-6a5e8924871&title=)

# 安装依赖
> go get github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway


# proto 文件
gateway/pb/gateway.proto 文件；其中 需要 import google/api/annotations.proto；可以将 github.com\grpc-ecosystem\grpc-gateway 下的 third_party\googleapis\google\api  文件拷贝到项目目录下；执行protoc命令
> protoc _-I_=./gateway/pb/ _-I_=./third_party/ _--go_out_=./gateway/ _--go-grpc_out_=./gateway/ _-grpc-gateway_out_=./gateway/ ./gateway/pb/gateway.proto

```go
syntax = "proto3";

package stream;

option go_package = "pb/;gateway";
import "google/api/annotations.proto";

message Request {
  string name = 1;
}

message Reply {
  string content = 1;
}

service GatewayDemo {
  rpc Gate(Request) returns (Reply) {
    option (google.api.http) = {
      get: "/v1/gate/{name}"
    };
  }
}
```

# Server端
server端与正常gRPC服务没什么差别；
```go
package main

import (
	"context"
	"fmt"
	"net"

	gateway "github.com/grpc-demo/gateway/pb"
	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"
)

type Gate struct {
	gateway.UnimplementedGatewayDemoServer
}

func (g Gate) Gate(ctx context.Context, req *gateway.Request) (*gateway.Reply, error) {
	return &gateway.Reply{
		Content: fmt.Sprintf("name:%s;", req.Name),
	}, nil
}

func main() {
	server := grpc.NewServer()
	gateway.RegisterGatewayDemoServer(server, &Gate{})
	reflection.Register(server)
	lis, _ := net.Listen("tcp", ":9000")
	_ = server.Serve(lis)
}
```

# Http服务端
通过 gateway.RegisterGatewayDemoHandlerFromEndpoint 将http请求代理到给RPC服务；
```go
package main

import (
	"context"
	"net/http"

	gateway "github.com/grpc-demo/gateway/pb"
	"github.com/grpc-ecosystem/grpc-gateway/runtime"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
)

const (
	// gRPC服务地址
	ServerAddr = "127.0.0.1:9000"

	ClientAddr = "127.0.0.1:8000"
)

func main() {
	ctx, cancelFunc := context.WithCancel(context.Background())
	defer cancelFunc()
	mux := runtime.NewServeMux()
	opts := []grpc.DialOption{grpc.WithTransportCredentials(insecure.NewCredentials())}
	err := gateway.RegisterGatewayDemoHandlerFromEndpoint(ctx, mux, ServerAddr, opts)
	if err != nil {
		panic(err)
	}
	err = http.ListenAndServe(ClientAddr, mux)
	if err != nil {
		panic(err)
	}
}
```

# 启动测试
> go run server.go

访问 127.0.0.1:8000/v1/gate/grpc
同时也可以通过grpcurl 测试服务端
> grpcurl -plaintext -d '{"name":"grpc"}'  localhost:9000 stream.GatewayDemo/Gate

