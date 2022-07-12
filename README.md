### 测试过的 golang 版本

```
go version
```

```
go version go1.10 linux/amd64
```

```
wget https://go.dev/dl/go1.10.linux-amd64.tar.gz
wget https://go.dev/dl/go1.10.darwin-amd64.pkg
```

### 生成证书

```
export NGROK_DOMAIN=ng.example.com
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
openssl genrsa -out client.key 2048
openssl req -new -key client.key -subj "/CN=$NGROK_DOMAIN" -out client.csr
openssl x509 -req -in client.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out client.crt -days 5000
```

### 替换编译时使用的证书

```
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp client.crt assets/server/tls/snakeoil.crt
cp client.key assets/server/tls/snakeoil.key
```

### 开始编译

服务端：

```
GOOS=linux GOARCH=amd64 make release-server
GOOS=darwin GOARCH=amd64 make release-server
```

客户端

```
GOOS=linux GOARCH=amd64 make release-client
GOOS=darwin GOARCH=amd64 make release-client
```

生成的编译文件在当前目录：./bin

### 使用示例

服务端

```
./bin/ngrokd -domain="ng.example.com" -httpAddr=":801" -httpsAddr=":802"
```

客户端配置文件：~/.ngrok

```
server_addr: "ng.example.com:4443"
trust_host_root_certs: false
tunnels:
  localapp:
   proto:
     http: 80
   subdomain: test
```

客户端启动

```
ngrok start-all
```

最终访问域名：

```
test.ng.example.com:801
```
