+++
title = "一、Hello World"
description = "gRPC 是 Google 开发的一款高性能、开源的远程过程调用（RPC）框架。它使用 Protocol Buffers 作为接口定义语言（IDL），可以在多种编程语言和平台之间进行通信。"
weight = 1
url = "/grpc/01.html"
+++

> 一个高性能、开源的通用RPC框架
> A high performance, open source universal RPC framework

# 一、环境配置
1、下载并安装 [protocol-buffers](https://developers.google.cn/protocol-buffers)
2、安装Go语言插件
> go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28 
> go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2

3、创建GRPC项目目录：
> mkdir grpc-demo
> go mod init grpc-demo


# 二、创建并编译proto 文件
> hello/proto/helloworld.proto

Protobuf 中 除了支持如double、float、 int32、int64、 uint32、uint64、 bool、 string 和bytes 等基本数据类型还支持enum，oneof，map，repeated 等类型；更详细的Protobuf文档可以查阅 ：[protocol-buffers 文档](https://developers.google.cn/protocol-buffers/docs/proto3#simple)
```go
syntax = "proto3";

package helloworld;
option go_package = "proto/;proto";
service Greeter {
    rpc SayHello(HelloRequest) returns (HelloReply) {};
}

enum Gender {
    Unknown = 0;
    Female = 1;
    Male = 2;
}
message HelloRequest {
    string name = 1;
    int64 age = 2;
    oneof call {
        string mobile = 3;
        string phone = 4;
    }
    map<int64,string> role = 5;
    Gender gender = 6;
}

message HelloReply {
    string message = 1;
}
```
创建好proto文件后执行编译命令，生成go文件代码
> protoc _-I_=./hello/proto/  _--go_out_=./hello/ _--go-grpc_out_=./hello/ ./hello/proto/helloworld.proto  

参数说明：
_**-I**_**  或  --proto_path **：指定指定要搜索的目录进口。可以指定多次；将按顺序搜索目录。如果当前proto文件中依赖第三方文件，一般需要指定第三方文件路径；
_**--go_out**_**= ：**设置所生成的Go代码输出的目录
_**--go-grpc_out**_**= ： **设置所生成的  protoc-gen-go-grpc 生成代码输出的目录
执行成功后生成 hello/proto/helloworld.pb.go hello/proto/helloworld_grep.pb.go 两个文件

# 三、服务端代码实现
我们在第二步中的proto文件中定义了 Greeter 服务，Greeter 服务包含一个SayHello方法；在生成的grpc文件中可以找到 一个方法
> func RegisterGreeterServer(s grpc.ServiceRegistrar, srv GreeterServer)

接下来服务端要做的就是实现 GreeterServer 接口注册并启动服务
```go
var port string

func init() {
	flag.StringVar(&port, "p", "8000", "启动端口号")
	flag.Parse()
}

type Server struct {
	pb.UnimplementedGreeterServer
}

func (gs Server) SayHello(ctx context.Context, r *pb.HelloRequest) (*pb.HelloReply, error) {
	str := r.GetMobile()
	if len(str) == 0 {
		str = r.GetPhone()
	}
	var url string
	for k, v := range r.Role {
		url = fmt.Sprintf("%d=%s&", k, v)
	}
	return &pb.HelloReply{Message: "Hello World " + r.Name + str + "?" + url}, nil
}
func main() {
	server := grpc.NewServer()
	pb.RegisterGreeterServer(server, &Server{})
	reflection.Register(server)
	lis, _ := net.Listen("tcp", ":"+port)
	server.Serve(lis)
}
```
> 26：reflection.Register(server) 增加后，可以通过grpcurl进行测试


# 四、客户端代码开发
客户端调用代码 在helloworld_grpc.pb.go 已经生成：func NewGreeterClient(cc grpc.ClientConnInterface) GreeterClient
创建客户端对象，直接调用服务端方法；
```go
var port string

func init() {
	flag.StringVar(&port, "p", "8000", "启动端口号")
	flag.Parse()
}
func main() {
	conn, _ := grpc.Dial(":"+port, grpc.WithInsecure())
	defer conn.Close()

	client := pb.NewGreeterClient(conn)
	_ = SayHello(client)
}

func SayHello(client pb.GreeterClient) error {
	reapy := &pb.HelloRequest{
		Name: "eddycjy",
		Call: &pb.HelloRequest_Mobile{Mobile: "151"},
		Role: map[int64]string{1: "888"},
	}
	resp, _ := client.SayHello(context.Background(), reapy)
	log.Printf("client.SayHello resp: %s", resp.Message)
	return nil
}
```
> 其中8行中 grpc.WithInsecure() 方法在新版本中已经不赞成使用；建议使用 ** grpc.WithTransportCredentials(insecure.NewCredentials())**替换


