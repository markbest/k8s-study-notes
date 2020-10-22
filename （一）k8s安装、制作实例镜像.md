最近买了本《Kubernetes in Action中文版》，准备在工作之余学习Kubernetes，增加自己的知识储备，这里简单做下笔记。

## k8s安装配置
 - mac系统中docker自带Kubernetes，所以只安装docker即可。docker最新版本的下载地址[https://www.docker.com/get-started](https://www.docker.com/get-started)，选择mac版本下载，下载文件为：Docker.dmg，安装dmg文件后修改配置就可以开启Kubernetes，如下图：  
![](https://github.com/markbest/k8s-study-notes/blob/main/images/d158d8a2-014e-11eb-b281-7eeb6dbfe843.png "")  
 - windows系统具体安装可以自行百度。

## 制作实例镜像
书中的实例采用的是node.js案例，从制作镜像到推送到docker hub，从编写yaml，到创建node和service，完整的实现一个小型服务的部署流程，鉴于我们后端没有用node.js所以这里换成golang来实例操作。
- main.go
```golang
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
)

func sayHello(w http.ResponseWriter, r *http.Request) {
	hostName, _ := os.Hostname()
	_, _ = fmt.Fprintf(w, "hello world, " + hostName + "\n")
}

func main() {
	http.HandleFunc("/", sayHello)

	log.Println("服务启动成功，监听端口：8001")
	if er := http.ListenAndServe("0.0.0.0:8001", nil); er != nil {
		log.Fatal("ListenAndServe: ", er)
	}
}
```
- build.sh
```shell
#!/usr/bin/env bash
cd /go/src/app/ && ./main
```
- Dockerfile
```
FROM golang
MAINTAINER markbest
WORKDIR /go/src/
COPY . .
EXPOSE 8001
CMD ["/bin/bash", "/go/src/script/build.sh"]
```
- 整个目录结构如下：  
go-example  
|-- app  
|  |-- main  
|  |-- Dockerfile  
|-- script  
|  |-- build.sh  
