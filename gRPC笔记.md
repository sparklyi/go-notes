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

### 7.自定义验证

```sh
gRPC提供了一个接口，实现GetRequestMetadata和RequireTreansportSecurity
```



### 8. gRPC调用类型

**Unary（单一请求-响应）**：一个请求对应一个响应。

**Server Streaming（服务端流）**：客户端发出请求，服务器返回一个流，包含多个响应。

`stream` 关键字用于定义服务端返回的响应是一个流（多个响应）。

**Client Streaming（客户端流）**：客户端发送多个请求，服务器返回一个响应。

**Bidirectional Streaming（双向流）**：客户端和服务器同时发送多个请求和响应，通信是全双工的。

#### 何时使用流模式？

- **大数据传输**：在传输大量数据或实时数据时，流模式比单一请求-响应更高效。
- **实时通讯**：像实时聊天或游戏更新这样需要双向通信的场景非常适合双向流模式。
- **数据处理流水线**：通过流式处理可以逐步传输数据并逐步响应，从而减少延迟。



### gRPC拦截器(中间件)

**Unary 拦截器**：用于单一请求-响应模式。

**Stream 拦截器**：用于流模式的请求-响应。

#### 拦截器的应用场景

1. **日志记录**：记录每个 gRPC 请求的详细信息。
2. **认证和鉴权**：在调用 gRPC 方法前检查身份认证和权限。
3. **错误处理**：在请求出错时统一处理。
4. **请求统计**：统计服务请求的数量、响应时间等。