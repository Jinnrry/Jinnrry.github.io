---
layout:     post
title:      "个人服务器架构介绍"
subtitle:   " 合理使用高射炮打蚊子"
date:       2020-07-12 12:00:00
author:     "Jinnrry"
header-img: "img/25.jpg"
catalog: true
tags:
    - 架构
---
### 背景
服务器硬件：BWG 洛杉矶机房 10G+512M+1核

截止目前，我服务器（你现在看的这个页面就是在这个服务器上面）上面运行的服务有：Nginx、PHP、MySQL、Redis、Log收集器、Ngork 。这些服务全部部署在一台机器上面。但是由于我服务器经常拿来做一些测试，导致经常需要
重装系统。因此每次重装系统后的服务部署都格外头疼。基本上把这些所有的服务部署一般，半天时间就没了。因此，一个快速稳定的部署方案十分有必要。

刚开始的想法是把所有部署操作写成一个Shell脚本，每次部署只用执行一遍Shell脚本就行了。但是Shell脚本维护成本太高，而且不同操作系统，不同版本的系统，很容易出现问题。因此
放弃了这个思路。

第二个想法是一步到位，上Docker + K8S，但是考虑到机器性能，K8S实在是太重了，可能K8S的性能消耗比我所有服务的消耗还要高。但是容器化部署是一个非常好的思路，于是想到了
使用轻量一点的方案docker+docker-compose。这套方案优势在于

1、docker容器带来的跨平台性
2、docker-compose进行容器编排足够轻量，不会额外消耗机器资源（我服务器配置太低，这点非常重要）

### 实现过程

一、打包Dokcerfile

打包过程中基础镜像选择了各个官方镜像，另外由于服务器磁盘只有10G，需要非常注意Docker镜像的尺寸，基本上每一步操作的缓存都得清理干净，比如apt-get、gcc build这些操作带来的
缓存可能非常大，不清理干净最终容器将会非常大。

另外，基础镜像尽可能小，比如golang build完后只需要一个alpine镜像运行即可，能选alpine就不用用ubuntu，保证最终镜像尽可能小，毕竟10G磁盘

最后，golang、c++之类的编译型语言全部采用分阶段构建，编译阶段在已经容器中，编译完以后拷贝到新容器中运行，这样可以连源码，编译器的大小都省去了。还是那句话，10G磁盘，迫不得已啊

二、编写docker-compose.yml

为了尽可能节约磁盘空间，基本上所有服务都关闭了日志记录。但是必要的日志还是得收集，不能丢。因此自己写了一个日志收集工具（其实本来不想自己写的，想用ELK一套，但是，机器性能
摆在这里，没办法，只能自己动手）。工具使用golang编写，功能只有一个，解析各个日志，然后存进MySQL。

工具日志也单独部署在一个容器中，日志数据写入到统一的日志数据卷中，然后日志服务读取日志数据卷的内容，写入MySQL。

其中，Nginx日志为了方便解析，将其格式改成了JSON。配置如下

```

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error_local.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

   # log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
   #                   '$status $body_bytes_sent "$http_referer" '
   #                   '"$http_user_agent" "$http_x_forwarded_for"';

    log_format  main '{"timestamp":"$time_iso8601",'
        '"host":"$server_addr",'
        ' "clientip" : "$remote_addr",'
        ' "size" : "$body_bytes_sent" ,'
        '"respnsetime":"$request_time",'
        '"upstremtime":"$upstream_response_time",'
        '"upstremhost":"$upstream_addr",'
        '"httphost":"$host",'
        '"referer":"$http_referer",'
        '"xff":"$http_x_forwarded_for",'
        '"agent":"$http_user_agent",'
        '"clientip":"$remote_addr",'
        '"request":"$request",'
        '"uri":"$uri",'
        '"status":"$status"}';
    access_log  /var/log/nginx/access_local.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}


```


### 自动CI/CD

整个项目全部使用docker-compose管理以后，更新代码，重新编译运行整个项目就只需要两行命令了

```
git pull
docerk-compose up -d --build
```

但是，懒人改变世界。我连这两行命令都懒得敲。因为要敲这2行命令的话我还得ssh到服务器，还是挺麻烦的。于是思考能不能搞一套自动CI/CD。

自动CI\CD第一想法肯定是Jenkins。但是！都512M内存了，Java系的东西就别想了，Jvm都起不来。于是开始寻找其他的方案，找来找去，基本上没啥合适的，要么太吃资源，要么功能不合适。

（其实有一种稍微合适点的，使用Github的Action+Docker Hub实现，但是我的项目是私有的，Docker hub私有项目得给钱，本着能白嫖绝不给钱的原则，这个方案也放弃了）

因此，最后，还是自己动手了。

因为我项目全是使用docker-compose管理的，为了管理和重装系统方便我并不想在物理机上面部署任何服务，但是，自动CI\CD又怎么能放得进docker容器里面呢。
这不就相当于完成自举（自己编译自己）了么。

最后，参考了Jenkins的docker部署实现方案，Jenkins代码运行在docker容器当中，遇到容器编译操作的时候，Jenkins在容器中通过SSH连接宿主机，然后发送Shell命令，
从而完成编译以及后续的运行操作。因此，我借鉴了Jenkins的实现方案，也将程序运行在Docker容器中，然后通过SSH连接到宿主机，从而实现docker的重编译。

然后就是如何触发Build的问题了，由于我代码全部放在Github上面，因此，触发毫无疑问选择了Github的webhook。

最终代码使用Golang实现，逻辑非常简单，开启一个http服务用于监听github的webhook，收到push事件的时候使用ssh协议连接到宿主机，运行rebuild命令。完整代码不足100行

```
package main

import (
	"fmt"
	"github.com/mitchellh/go-homedir"
	"golang.org/x/crypto/ssh"
	"gopkg.in/go-playground/webhooks.v5/github"
	"io/ioutil"
	"log"
	"net/http"
	"time"
)

const (
	path = "/"
)

func main() {
	hook, _ := github.New(github.Options.Secret("_____"))  // 校验Web hook的秘钥

	http.HandleFunc(path, func(w http.ResponseWriter, r *http.Request) {
		payload, err := hook.Parse(r, github.PushEvent)
		if err != nil {
			if err == github.ErrEventNotFound {
				// ok event wasn;t one of the ones asked to be parsed
			}
		}
		switch payload.(type) {

		case github.PushPayload:
			rebuild()
		}
	})
	http.ListenAndServe(":80", nil)
}



func rebuild(){
	sshHost := "___"
	sshUser := "___"
	sshPassword := "____"
	sshType := "____"//password 或者 key
	sshKeyPath := ""//ssh id_rsa.id 路径"
	sshPort := 22


	//创建sshp登陆配置
	config := &ssh.ClientConfig{
		Timeout:         time.Second,//ssh 连接time out 时间一秒钟, 如果ssh验证错误 会在一秒内返回
		User:            sshUser,
		HostKeyCallback: ssh.InsecureIgnoreHostKey(), //这个可以， 但是不够安全
		//HostKeyCallback: hostKeyCallBackFunc(h.Host),
	}
	if sshType == "password" {
		config.Auth = []ssh.AuthMethod{ssh.Password(sshPassword)}
	} else {
		config.Auth = []ssh.AuthMethod{publicKeyAuthFunc(sshKeyPath)}
	}



	//dial 获取ssh client
	addr := fmt.Sprintf("%s:%d", sshHost, sshPort)
	sshClient, err := ssh.Dial("tcp", addr, config)
	if err != nil {
		log.Fatal("创建ssh client 失败",err)
	}
	defer sshClient.Close()


	//创建ssh-session
	session, err := sshClient.NewSession()
	if err != nil {
		log.Fatal("创建ssh session 失败",err)
	}
	defer session.Close()
	//到宿主机执行命令
	combo,err := session.CombinedOutput("cd privateServer; bash ./rebuild.sh &")
	if err != nil {
		log.Fatal("远程执行cmd 失败",err)
	}
	log.Println("命令输出:",string(combo))

}

func publicKeyAuthFunc(kPath string) ssh.AuthMethod {
	keyPath, err := homedir.Expand(kPath)
	if err != nil {
		log.Fatal("find key's home dir failed", err)
	}
	key, err := ioutil.ReadFile(keyPath)
	if err != nil {
		log.Fatal("ssh key file read failed", err)
	}
	// Create the Signer for this private key.
	signer, err := ssh.ParsePrivateKey(key)
	if err != nil {
		log.Fatal("ssh key signer failed", err)
	}
	return ssh.PublicKeys(signer)
}
```

rebuil.sh 脚本内容(我为了节约磁盘空间，每次编译后都把多余镜像删掉了)

```
nohup sh -c 'git pull && docker-compose up -d --build  && docker system prune --all --force ' > build.log 2>&1 &
```

容器Dockerfile内容

```
FROM golang:latest as builder

WORKDIR /app/webhook

ENV GO111MODULE "on"
ENV GOPROXY "https://goproxy.cn,direct"

COPY src /app/webhook/

RUN go mod download

#RUN GOOS=linux go build main.go
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .


FROM alpine:latest


# 设置时区
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
RUN apk add --no-cache tzdata \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    &&rm -rf /var/cache/apk/* /tmp/* /var/tmp/* $HOME/.cache

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy the Pre-built binary file from the previous stage
COPY --from=builder /app/webhook/main .

#
#
## Command to run the executable
CMD ["./main"]

#CMD tail -f conf.yaml

```


最后，加入docker-compose，完美实现自动CI\CD，以后只需要push代码到github，服务器就会自动拉取最新代码，然后重新编译运行