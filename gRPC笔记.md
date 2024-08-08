## gRPC笔记

### 1. message

类比为Go中的struct

```protobuf
message demo{
	string name=1;//后面是编号，生成文件时按此顺序生成
	repeated int32 nums = 2;//重复多个，生成为切片
}
```

### 2. syntax

语法，规定使用的protobuf版本，不指定默认为V2,指定时必须是第一非空行

```protobuf
syntax="proto3";
```

### 3.message嵌套

```protobuf
message demo {
	
	message Person {
		string name = 1;
	}
	repeated Person d = 1;

}
```

### 4.服务定义

service

protocol buffer编译器会根据所选的语言类型生成服务接口及存根

```properties
service servcieName {
	#rpc 服务函数名(参数) 返回 (返回参数)
	rpc Query(queryRequest) returns (queryResponse)
}
```

### 5. go_package

参数

1. **点号 (.)**：表示当前目录，生成的 Go 文件将会放在当前的目录中。
2. **分号 (;)**：用来分隔路径和包名。
3. **包名 (server)**：表示生成的 Go 包的名称是 `server`。

```protobuf
option go_package = ".;server"

新版
option go_package="./;server"
```



### 6.生成命令

参数

1. **点号** 生成的路径 当前目录
2. proto文件：由哪个文件来生成 
3. **--go_out=.**：生成 Protocol Buffers 的 Go 数据类型代码。
4. **--go-grpc_out=.**：生成 gRPC 服务的 Go 接口代码。

```protobuf
protoc --go_out =. demo.proto 
protoc --go-grpc_out =. demo.proto
合用
protoc --go_out =. --go-grpc_out =. demo.proto

新版
protoc --go_out =./ --go-grpc_out =./ demo.proto
```

#### 7.自定义验证

```sh
gRPC提供了一个接口，实现GetRequestMetadata和RequireTreansportSecurity
```

