+++
title = "六、证书验证"
description = "gRPC 是 Google 开发的一款高性能、开源的远程过程调用（RPC）框架。它使用 Protocol Buffers 作为接口定义语言（IDL），可以在多种编程语言和平台之间进行通信。"
weight = 6
url = "/grpc/06.html"
+++

> 还记得前面章节中gRPC的服务中客户端在链接服务器中通过 grpc.WithInsecure() 选项跳过了对服务器证书的验证吗？为了保障gRPC通信不被第三方监听篡改或伪造，我们试一下对服务器启动TLS加密特性。


# 一、生成证书
> 通过一个安全可靠的根证书分别对服务器和客户端的证书进行签名。这样客户端或服务器在收到对方的证书后可以通过根证书进行验证证书的有效性。

1、CA证书
创建 ca.conf
```shell
[ req ]
default_bits       = 4096
distinguished_name = req_distinguished_name
 
[ req_distinguished_name ]
countryName                 = Country Name (2 letter code)
countryName_default         = CN
stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Beiging
localityName                = Locality Name (eg, city)
localityName_default        = Beijing
organizationName            = Organization Name (eg, company)
organizationName_default    = company
commonName                  = Common Name (e.g. server FQDN or YOUR name)
commonName_max              = 64
commonName_default          = Ted CA Test
```
生成ca根证书
```shell
SERVER_CN=localhost
CLIENT_CN=localhost

openssl genrsa -passout pass:1111 -des3 -out ca.key 4096
# Generates ca.crt
openssl req -passin pass:1111 -new -x509 -days 365 -key ca.key -out ca.crt -subj "/CN=${SERVER_CN}" -config ./ca.conf
```

2、服务端证书
注意：go 1.15 版本开始废弃 CommonName，因此推荐使用 SAN 证书

- 编辑server.conf 文件 其中 alt_names 配置中的内容 后面客户端请求会用到
```shell
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
req_extensions     = req_ext
 
[ req_distinguished_name ]
countryName                 = Country Name (2 letter code)
countryName_default         = CN
stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = JiangSu
localityName                = Locality Name (eg, city)
localityName_default        = NanJing
organizationName            = Organization Name (eg, company)
organizationName_default    = your_company
commonName                  = Common Name (e.g. server FQDN or YOUR name)
commonName_max              = 64
commonName_default          = www.eline.com
 
[ req_ext ]
subjectAltName = @alt_names
 
[alt_names]
DNS.1 = localhost 
DNS.2 = www.test.com

```

- 生成server 端证书文件
```shell
# Generate server key:
openssl genrsa -passout pass:1111 -des3 -out server.key 4096

openssl req -passin pass:1111 -new -key server.key -out server.csr -subj "/CN=${SERVER_CN}" -config ./server.conf
# Generates server.crt
openssl x509 -req -passin pass:1111 -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt -extfile ./server.conf -extensions req_ext
# Remove passphrase from server key
openssl rsa -passin pass:1111 -in server.key -out server.key
```

3、客户端证书
```shell
# Generate client key
openssl genrsa -passout pass:1111 -des3 -out client.key 4096

openssl req -passin pass:1111 -new -key client.key -out client.csr -subj "/CN=${CLIENT_CN}" -config ./server.conf

# Generates client.crt
openssl x509 -passin pass:1111 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt -extfile ./server.conf -extensions req_ext
# Remove passphrase from client key:
openssl rsa -passin pass:1111 -in client.key -out client.key
```

注意：1、命令中的 ${CLIENT_CN}  ${SERVER_CN} 都是最开始定义好的shell变量；2、如果是win系统需要单独安装openssl：[下载OpenSSL](http://slproweb.com/products/Win32OpenSSL.html)；

# 二、服务端
> 服务器端用credentials.NewTLS函数生成证书，通过ClientCAs选择CA根证书，并通过ClientAuth选项启用对客户端进行验证。

```shell

type SafetyServer struct {
	pb.UnsafeSafetyDemoServer
}

func (s SafetyServer) Secret(ctx context.Context, in *pb.Request) (*pb.Reply, error) {
	return &pb.Reply{
		Content: "Secret Content： " + in.Name,
	}, nil
}

func main() {

	certificate, err := tls.LoadX509KeyPair("keys/server.crt", "keys/server.key")
	if err != nil {
		log.Fatal(err)
	}
	certPool := x509.NewCertPool()
	ca, err := ioutil.ReadFile("keys/ca.crt")
	if err != nil {
		log.Fatal(err)
	}
	if ok := certPool.AppendCertsFromPEM(ca); !ok {
		log.Fatal(" failed to append certs ")
	}
	creds := credentials.NewTLS(&tls.Config{
		Certificates: []tls.Certificate{certificate},
		ClientAuth:   tls.RequireAndVerifyClientCert,
		ClientCAs:    certPool,
	})

	server := grpc.NewServer(grpc.Creds(creds))
	pb.RegisterSafetyDemoServer(server, &SafetyServer{})
	reflection.Register(server)
	lis, _ := net.Listen("tcp", ":9090")
	server.Serve(lis)
}

```

# 三、客户端
> 客户端通过 credentials.NewTLS函数调用，引入一个CA根证书和服务器（ServerName ）的名字来实现对服务器进行验证。客户端在链接服务器时会首先请求服务器的证书，然后使用CA根证书对收到的服务器端证书进行验证。

```shell
func main() {

	certificate, err := tls.LoadX509KeyPair("keys/client.crt", "keys/client.key")
	if err != nil {
		log.Fatal(err)
	}
	certPool := x509.NewCertPool()
	ca, err := ioutil.ReadFile("keys/ca.crt")
	if err != nil {
		log.Fatal(err)
	}
	if ok := certPool.AppendCertsFromPEM(ca); !ok {
		log.Fatal(" failed to append certs ")
	}
	creds := credentials.NewTLS(&tls.Config{
		Certificates: []tls.Certificate{certificate},
		ServerName:   "www.test.com", // NOTE: 需要与生成证书的配置匹配
		RootCAs:      certPool,
	})
	opt := []grpc.DialOption{
		grpc.WithTransportCredentials(creds),
	}
	conn, _ := grpc.Dial(":9090", opt...)
	defer conn.Close()

	client := pb.NewSafetyDemoClient(conn)

	speak, err := client.Secret(context.Background(), &pb.Request{Name: "golang"})
	if err != nil {
		log.Println("error")
		log.Fatal(err)
		return
	}
	log.Println(speak)
}

```
ServerName  需要与生成证书的配置匹配;比匹配是会返回错误：
> rpc error: code = Unavailable desc = connection error: desc = "transport: authentication handshake failed: x509: certificate is valid for localhost, www.test.com, not www.test.com1"

获取代码：[链接](https://github.com/LittleMoreInteresting/grpc-demo/tree/master/safety)
