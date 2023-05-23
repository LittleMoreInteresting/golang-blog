+++
title = "二、Streaming RPC"
description = "gRPC 是 Google 开发的一款高性能、开源的远程过程调用（RPC）框架。它使用 Protocol Buffers 作为接口定义语言（IDL），可以在多种编程语言和平台之间进行通信。"
weight = 2
url = "/grpc/02.html"
+++
# gRPC调用方式
在gRPC中，一共包含四种调用方式。

- Unary RPC:一元RPC
- Server-side streaming RPC: 服务端流式RPC
- Client-side streaming RPC: 客户端流式RPC
- Bidirectional streaming RPC: 双向流式RPC
# Streaming RPC
> streaming RPC 适用于大数据包场景，可以进行实时交互；

**服务端流式RPC **：客户端发起一次普通的RPC请求，服务端通过流式响应多次发送数据集，客户端Recv接收数据集
**客户端流式RPC**：客户端发起多次请求给服务端，而服务端仅响应客户端一次
**双向流式RPC**: 由客户端以流式请求服务端，服务端同样以流式响应客户端。
# 代码实现
1、proto文件：创建proto文件定义三种流模式服务，执行protoc命令编译代码；
```protobuf
syntax = "proto3";

package pb;

option go_package = "pb/;pb";

message Reply {
  string type = 1;
  string value = 2;
}

message Request {
  string type = 1;
  string value = 2;
}

service Streaming {
  rpc ServerStream (Request) returns (stream Reply);
  rpc ClientStream (stream Request) returns (Reply);
  rpc Bidirectional (stream Request) returns (stream Reply);
}
```
2、server：
客户端流模式下 服务端手到 io._EOF时服务端需要通过 _stream.SendAndClose 向客户端返回；客户端 stream.CloseAndRecv 与之对应
```go
var port string

func init() {
	flag.StringVar(&port, "p", "8000", "启动端口号")
	flag.Parse()
}

type StreamServer struct {
	pb.UnimplementedStreamingServer
}

func (ss StreamServer) ServerStream(in *pb.Request, stream pb.Streaming_ServerStreamServer) error {
	for i := 0; i < 20; i++ {
		err := stream.Send(&pb.Reply{Type: "Server-Side stream", Value: fmt.Sprintf("val:%d", i)})
		if err != nil {
			return err
		}
	}
	return nil
}

func (ss StreamServer) ClientStream(stream pb.Streaming_ClientStreamServer) error {
	for {
		recv, err := stream.Recv()
		if err == io.EOF {
			return stream.SendAndClose(&pb.Reply{
				Type:  "Client-Side stream",
				Value: "close",
			})
		}
		if err != nil {
			return err
		}
		log.Printf("%v", recv)
	}
}
func (ss StreamServer) Bidirectional(stream pb.Streaming_BidirectionalServer) error {
	i := 0
	for {
		_ = stream.Send(&pb.Reply{Type: "Bidirectional stream", Value: fmt.Sprintf("val:%d", i)})
		recv, err := stream.Recv()
		if err == io.EOF {
			return nil
		}
		if err != nil {
			return err
		}
		log.Printf("%v", recv)
		i++
	}
}

func main() {
	server := grpc.NewServer()
	pb.RegisterStreamingServer(server, &StreamServer{})
	reflection.Register(server)
	lis, _ := net.Listen("tcp", ":"+port)
	_ = server.Serve(lis)
}
```
3、client ：未方便后面测试，通过参数 -m 启动不同流模式客户端；
```go
var port string
var method int

func init() {
	flag.StringVar(&port, "p", "8000", "启动端口号")
	flag.IntVar(&method, "m", 1, "流模式")
	flag.Parse()
}
func ServerStreamClient(client pb.StreamingClient, r *pb.Request) error {
	stream, err := client.ServerStream(context.Background(), r)
	if err != nil {
		return err
	}
	for {
		recv, err := stream.Recv()
		if err == io.EOF {
			break
		}
		if err != nil {
			return err
		}
		log.Printf("response:%v", recv)
	}
	return nil
}

func ClientStreamClient(client pb.StreamingClient) error {
	stream, err := client.ClientStream(context.Background())
	if err != nil {
		return err
	}
	for i := 0; i < 10; i++ {
		_ = stream.Send(&pb.Request{Type: "ClientStreamClient", Value: fmt.Sprintf("val:%d", i)})
	}
	recv, err := stream.CloseAndRecv()

	log.Printf("resp :%v", recv)
	return err
}

func Bidirectional(client pb.StreamingClient) error {
	stream, err := client.Bidirectional(context.Background())
	if err != nil {
		return err
	}
	for i := 0; i < 10; i++ {
		_ = stream.Send(&pb.Request{Type: "BidirectionalClient", Value: fmt.Sprintf("val:%d", i)})
		recv, err := stream.Recv()
		if err == io.EOF {
			break
		}
		if err != nil {
			log.Printf("resp :%v", err)
			return err
		}
		log.Printf("resp :%v", recv)
	}
	return stream.CloseSend()
}

func main() {
	conn, _ := grpc.Dial(":"+port, grpc.WithTransportCredentials(insecure.NewCredentials()))
	defer conn.Close()
	log.Printf("mothod %d", method)
	client := pb.NewStreamingClient(conn)
	switch method {
	case 1:
		_ = ServerStreamClient(client, &pb.Request{Type: "ss client", Value: "ok"})
	case 2:
		_ = ClientStreamClient(client)
	case 3:
		_ = Bidirectional(client)
	default:
		log.Fatal("error method")
	}
}
```
4、启动测试；
> go run server.go
> go run client.go -m 1/2/3

