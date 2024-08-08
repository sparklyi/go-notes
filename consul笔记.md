## consul笔记

#### docker安装

```sh
docker run -d --name=consul -p 8500:8500 -p 8600:8600/udp hashicorp/consul agent -dev -client=0.0.0.0
```

notes:

可能健康检查会显示映射不对等

下载windows版consul即可

配置环境变量后，使用consul agent -dev启动开发模式

#### 访问地址

```http	
127.0.0.1:8021
```







```go
package main

import (
	"fmt"
	capi "github.com/hashicorp/consul/api"
	"net/http"
)

var (
	config capi.Config
	client *capi.Client
)

func init() {
	//配置consul地址
	config = capi.Config{
		Address: "127.0.0.1:8500",
	}
	client, _ = capi.NewClient(&config)

}
func registerService(name string) {
	//服务注册
	err := client.Agent().ServiceRegister(&capi.AgentServiceRegistration{
		ID:      name,
		Name:    name,
		Address: "127.0.0.1",
		Port:    8021,
		//健康检查
		Check: &capi.AgentServiceCheck{
			CheckID:                        name,
			HTTP:                           "http://127.0.0.1:8021/" + name,
			Interval:                       "10s",
			Timeout:                        "5s",
			DeregisterCriticalServiceAfter: "10s",
		},
	})
	if err != nil {
		panic(err)
	}

}
func allService() {

	services, err := client.Agent().Services()
	if err != nil {
		panic(err)
	}
	for _, service := range services {
		fmt.Println(service.Service)

	}
}

// filterService 过滤服务
func filterService() {
	filter, err := client.Agent().ServicesWithFilter(`Service=="demo"`)
	if err != nil {
		return
	}
	for _, service := range filter {
		fmt.Println(service.Service)
	}
}

func hello(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello World!"))
}
func setupHTTP() {
	http.HandleFunc("/health", hello)
	http.HandleFunc("/demo", hello)
	http.HandleFunc("/demo1", hello)
	http.HandleFunc("/demo2", hello)
	http.ListenAndServe(":8021", nil)

}
func main() {
	//需要监听对应端口 给健康检查返回心跳
	go setupHTTP()
	registerService("demo1")
	registerService("demo2")
	//allService()
	filterService()
	select {}
}

```

