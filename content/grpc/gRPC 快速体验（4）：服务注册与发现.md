+++
title = "四、服务注册与发现"
description = "gRPC 是 Google 开发的一款高性能、开源的远程过程调用（RPC）框架。它使用 Protocol Buffers 作为接口定义语言（IDL），可以在多种编程语言和平台之间进行通信。"
weight = 4
url = "/grpc/04.html"
+++
# 环境依赖
- 安装并且启动[etcd](https://etcd.io/docs/v3.5/install/)
- 按照etcd SDK
>  go get  go.etcd.io/etcd/client/v3@v3.5.4
>  go get google.golang.org/grpc@v1.48.0

旧版本grpc与etcd可能存在不兼容问题，建议使用最新兼容版本
# 服务注册与发现
实现思路：服务端启动服务时将自己的服务信息如IP、端口、版本等信息注册到注册中心（此处为ETCD），客户端在进行服务调用时会以约定好的服务名到注册中心查询，发现具体可以调用的服务。
服务端实现：
```go
func init() {
	flag.StringVar(&port, "p", "8000", "启动端口号")
	flag.Parse()
}
func main() {
	server := grpc.NewServer()
	discover.RegisterDiscoverDemoServer(server, &DiscoverDemo{})
	reflection.Register(server)

	taget := fmt.Sprintf("grpc-demo/grpc/%s", types.SERVER_NAME)
	client, err := etcd3.New(etcd3.Config{
		Endpoints: []string{"http://127.0.0.1:2379"},
	})
	if err != nil {
		panic(err)
	}
    addr :=  "127.0.0.1:"+port
	err = register.EtcdAdd(client, taget,addr)
	if err != nil {
		return
	}
	lis, _ := net.Listen("tcp", addr)
	server.Serve(lis)
}
```

register.EtcdAdd 方法实现
```go
func EtcdAdd(c *clientv3.Client, service, addr string) error {
	em, _ := endpoints.NewManager(c, service)
	list, _ := em.List(c.Ctx())
	fmt.Println(list)
	return em.AddEndpoint(c.Ctx(), service+"/"+addr, endpoints.Endpoint{Addr: addr})
}
```

此时启动服务
> go run server.go -p 8001
> go run server.go -p 8002

可以通过 etcdctl 命令查看到注册信息：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/12487795/1661955413390-bb148eaa-edfb-4613-bc67-ea53556e17c0.png#clientId=u4f6a8c0e-be59-4&from=paste&height=128&id=ub5cacb20&originHeight=128&originWidth=433&originalType=binary&ratio=1&rotation=0&showTitle=false&size=3357&status=done&style=none&taskId=ud26ed930-7594-459f-a0e9-a7a1766616e&title=&width=433)

客户端实现：
```go
func main() {
	cli, _ := clientv3.NewFromURL("http://127.0.0.1:2379")

	taget := "grpc-demo/grpc/discover-demo"

	n := 0
	for {
		conn, err := register.EtcdDial(cli, taget)
		if err != nil {
			panic(err)
		}
		defer conn.Close()
		client := discover.NewDiscoverDemoClient(conn)
		reply, err := client.Discover(context.Background(), &discover.Request{Name: "DDDD"})
		if err != nil {
			panic(err)
		}
		fmt.Printf("%d: %v \n", n, reply)
		n++
		time.Sleep(time.Second)
	}
}


//register.EtcdDial
func EtcdDial(c *clientv3.Client, service string) (*grpc.ClientConn, error) {
	etcdResolver, err := resolver.NewBuilder(c)
	if err != nil {
		return nil, err
	}
	opts := []grpc.DialOption{
		grpc.WithTransportCredentials(insecure.NewCredentials()),
		grpc.WithResolvers(etcdResolver),
	}
	return grpc.Dial("etcd:///"+service, opts...)
}
```
启动客户端
> 0: content:"request content from8001 => DDDD"
> 1: content:"request content from8001 => DDDD" 
> 2: content:"request content from8001 => DDDD" 
> 3: content:"request content from8001 => DDDD" 
> 4: content:"request content from8002 => DDDD" 
> 5: content:"request content from8001 => DDDD" 

至此我们实现了服务端的注册，但停掉服务后会发现etcd中的注册信息并没有注销；实际服务端还应该实现服务的定期检测以及主动下线功能；
服务停止是主动下线：
```go
// server.go

go server.Serve(lis)

quit := make(chan os.Signal)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
<-quit
log.Println("Shotdown Server ...")
_ = register.EtcdDelete(client, taget, addr)


//register.EtcdDelete
func EtcdDelete(c *clientv3.Client, service, addr string) error {
	em, _ := endpoints.NewManager(c, service)
	return em.DeleteEndpoint(c.Ctx(), service+"/"+addr)
}

```

同时可以通过ETCD设置租期，和定时更新实现定时检查和异常自动下线
```go
func EtcdKeepAlive(c *clientv3.Client, service, addr string, ttl int64) error {
	target := service + "/" + addr

	go func() {
		ticker := time.NewTicker(time.Second * 10)
		for {
			grant, err := c.Grant(c.Ctx(), ttl)
			if err != nil {
				log.Println(err)
			}
			em, _ := endpoints.NewManager(c, service)
			update := endpoints.NewAddUpdateOpts(target, endpoints.Endpoint{Addr: addr}, clientv3.WithLease(grant.ID))
			err = em.Update(c.Ctx(), []*endpoints.UpdateWithOpts{update})
			if err != nil {
				log.Println(err)
			}
			log.Println("update")
			select {
			case <-stopSignal:
				return
			case <-ticker.C:
			}
		}

	}()

	return nil
}

func EtcdDelete(c *clientv3.Client, service, addr string) error {
	close(stopSignal)
	em, _ := endpoints.NewManager(c, service)
	return em.DeleteEndpoint(c.Ctx(), service+"/"+addr)
}
```
# Reference
[grpc-load-balancing](https://grpc.io/blog/grpc-load-balancing/)
