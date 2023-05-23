+++
title = "三、拦截器与Metadata"
description = "gRPC 是 Google 开发的一款高性能、开源的远程过程调用（RPC）框架。它使用 Protocol Buffers 作为接口定义语言（IDL），可以在多种编程语言和平台之间进行通信。"
weight = 3
url = "/grpc/03.html"
+++
# 拦截器 
根据RPC调用类型可以将gRPC拦截器分为两种：

- 一元拦 截器(Unary Interceptor) :拦截和处理一元RPC调用。
- 流拦截器(Stream Interceptor) :拦截和处理流式RPC调用。
## 1、服务端拦截器
服务端一元拦截器类型为 grpc.UnaryServerInterceptor；流拦截器类型为 StreamServerInterceptor; 下面实现一个服务端一元拦截器：在调用服务前后各打印一条日志
```go
func HelloInterceptor() grpc.UnaryServerInterceptor {
	return func(ctx context.Context,
		req interface{},
		info *grpc.UnaryServerInfo,
		handler grpc.UnaryHandler) (resp interface{}, err error) {
		log.Println("Hello")
		resp, err = handler(ctx, req)
		log.Println("Bye bye")
		return
	}
}
```
在服务端代码中配置拦截器；
```go
opts := []grpc.ServerOption{
		grpc.UnaryInterceptor(interceptors.HelloInterceptor()),
	}
server := grpc.NewServer(opts...)
```
## 2、使用多个拦截器
gRPC 默认只支持设置单个拦截器，设置过个拦截器会painc
> panic: The unary server interceptor was already set and may not be reset.

设置多个拦截器可以使用 grpc-ecosystem/go-grpc-middleware
>  go get -u github.com/grpc-ecosystem/go-grpc-middleware@latest

```go
opts := []grpc.ServerOption{
		grpc.UnaryInterceptor(grpc_middleware.ChainUnaryServer(
			interceptors.HelloInterceptor(),
			interceptors.DurationInterceptor(),
		)),
	}
```

## 3、客户端拦截器 超时控制实现
超时拦截器
```go
func TimeoutInterceptor(second) grpc.UnaryClientInterceptor {
	return func(ctx context.Context,
		method string,
		req,
		reply interface{},
		cc *grpc.ClientConn,
		invoker grpc.UnaryInvoker,
		opts ...grpc.CallOption) error {
		if _, ok := ctx.Deadline(); !ok {
			timeout := 3 * time.Second
			var cancel context.CancelFunc
			ctx, cancel = context.WithTimeout(ctx, timeout)
			defer cancel()
		}
		return invoker(ctx, method, req, reply, cc, opts...)
	}
}
```

设置客户端拦截器
```go


opt := []grpc.DialOption{
		grpc.WithTransportCredentials(insecure.NewCredentials()),
		grpc.WithUnaryInterceptor(interceptors.TimeoutInterceptor(time.Second)),
	}
	conn, _ := grpc.Dial(":"+port, opt...)
```
## 4、服务端超时拦截器
服务端拦截器可以参考 [go-zero](https://github.com/zeromicro/go-zero) 中拦截器的实现代码：定义done, paincChan 两个channel；单独开一个goroutine执行 handler 逻辑并处理panic；之后通过select 语句阻塞监听 ctx.Done;done 和panicChan；

```go
func UnaryTimeoutInterceptor(timeout time.Duration) grpc.UnaryServerInterceptor {
	return func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo,
		handler grpc.UnaryHandler) (interface{}, error) {
		var cancel context.CancelFunc
		if _, ok := ctx.Deadline(); !ok {
			ctx, cancel = context.WithTimeout(ctx, timeout)
		}
		if cancel != nil {
			defer cancel()
		}

		var resp interface{}
		var err error
		var lock sync.Mutex
		done := make(chan struct{})
		// create channel with buffer size 1 to avoid goroutine leak
		panicChan := make(chan interface{}, 1)
		go func() {
			defer func() {
				if p := recover(); p != nil {
					// attach call stack to avoid missing in different goroutine
					panicChan <- fmt.Sprintf("%+v\n\n%s", p, strings.TrimSpace(string(debug.Stack())))
				}
			}()

			lock.Lock()
			defer lock.Unlock()
			resp, err = handler(ctx, req)
			close(done)
		}()

		select {
		case p := <-panicChan:
			panic(p)
		case <-done:
			lock.Lock()
			defer lock.Unlock()
			return resp, err
		case <-ctx.Done():
			err := ctx.Err()
			if err == context.Canceled {
				err = status.Error(codes.Canceled, err.Error())
			} else if err == context.DeadlineExceeded {
				err = status.Error(codes.DeadlineExceeded, err.Error())
			}
			return nil, err
		}
	}
```

#  Metadata
在gRPC中，metadata实际上就是-一个 map结构:
> type MD map [string][] string

本质.上也可以通过Header来传递数据
1、设置metadata
```go
md := metadata.New(map[string]string{
		"auth": "golang",
	})
mdCtx := metadata.NewOutgoingContext(ctx, md)
```
2、读取metadata
```go
md, b := metadata.FromIncomingContext(ctx)
if b {
    log.Printf("metadata:%v", md)
    if auth, ok := md["auth"]; ok {
        log.Printf("auth:%v", auth)
    }
}
```
