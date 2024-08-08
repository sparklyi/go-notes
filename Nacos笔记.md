## Nacos笔记

#### docker安装

```sh
docker run --name nacos-standalone -e MODE=standalone -e JVM_XMS=512m -e JVM_XMX=512m -e JVM_XMN=256m -p 8848:8848 -p 9848:9848 -p 9849:9849 -d nacos/nacos-server:latest --restart=always
```

notes:

configClient.GetConfig方法的时候会访问grpc服务，nacos2添加了grpc通信方式，所以需要把grpc的端口也打开（9848 9849）

#### 访问地址

```http	
http://虚拟机地址:8848/nacos/index.html
```



#### 命名空间

隔离配置集

#### group

区分环境 dev test pro



```go
package main

import (
	"fmt"
	"github.com/nacos-group/nacos-sdk-go/v2/clients"
	"github.com/nacos-group/nacos-sdk-go/v2/common/constant"
	"github.com/nacos-group/nacos-sdk-go/v2/vo"
)

func main() {
	ip := "192.168.65.131"
	//创建nacos客户端配置
	clientCfg := constant.ClientConfig{
		NamespaceId:         "c33b746b-1df2-4a6a-a6c6-8da0901bdcde",
		TimeoutMs:           5000,
		LogDir:              "/nacos/log",
		CacheDir:            "/nacos/cache",
		LogLevel:            "debug",
		NotLoadCacheAtStart: true,
	}
	//服务端配置
	serverCfg := []constant.ServerConfig{
		*constant.NewServerConfig(ip, 8848),
	}
	//创建动态配置客户端
	client, err := clients.NewConfigClient(vo.NacosClientParam{
		ServerConfigs: serverCfg,
		ClientConfig:  &clientCfg,
	})
	if err != nil {
		panic(err)
	}
	//获取配置
	config, err := client.GetConfig(vo.ConfigParam{
		DataId: "user-web.yaml",
		Group:  "dev",
	})
	if err != nil {
		panic(err)
	}
	fmt.Println(config)
	//监听配置变化
	client.ListenConfig(vo.ConfigParam{
		DataId: "user-web.yaml",
		Group:  "dev",
		OnChange: func(namespace, group, dataId, data string) {
			fmt.Println("配置变化")
			fmt.Println(group, dataId, data, data)
		},
	})
	select {}
}

```



