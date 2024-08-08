## openSSL命令

#### 1. 生成私钥

```sh
openssl genrsa -out server.key 2048
```

#### 2.生成证书

```sh
openssl  req -new -x509 -key server.key -out server.crt -days 36500
```

#### 3.生成csr

```sh
openssl req -new -key server.key -out server.csr
```



#### 4.修改文件

```sh
#更改openssl.cfg
#1)复制一份你安装的openssl的bin目录里面的openssl.cfg 文件到你项目所在的目录
#2)找到[CA_default]，打开 copy_extensions = copy (就是把前面的#去掉)
#3)找到[ req ],打开 req_extensions = v3_req # The extensions to add to a certificate request
#4)找到[v3_req]，添加 subjectAltName = @alt_names
#5)添加新的标签[ alt_names ]，和标签字段
DNS.1 =*.kuangstudy.com
```

#### 5.生成证书私钥(.key)

```sh
openssl genpkey -algorithm RSA -out test.key
```

#### 6.通过证书私钥生成证书请求文件（.csr）

```sh
 openssl req -new -nodes -key test.key -out test.csr -x509  -days 3650 -subj "/C=cn/OU=myorg/O=mycomp/CN=myname" -config ./openssl.cfg -extensions v3_req

```

#### 7.生成SAN证书 （.pem）

```sh
openssl x509 -req -days 365 -in test.csr -out test.pem -CA server.crt -CAkey server.key -CAcreateserial -extfile ./openssl.cfg -extensions v3_req

```

#### 8.注意事项

1. 证书路径为绝对路径

