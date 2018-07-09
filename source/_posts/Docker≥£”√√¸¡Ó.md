title: Docker常用命令
date: 2016-08-07 21:31:39
tags: 
		- Docker
---

> ##### 作为Docker入门练手时整理



#### docker daemon
* `status docker`		/查看docker守护进程状态
* `stop/start docker` 	/关闭、开启docker守护进程

#### docker image
* `docker pull`		/拉取镜像(默认从docker hub拉取)
* `docker images`		/列出本地镜像<!-- more -->
```
REPOSITORY       TAG      IMAGE ID      CREATED      VIRTUAL SIZE
ubuntu           12.04    74fe38d11401  4 weeks ago  209.6 MB
ubuntu           precise  74fe38d11401  4 weeks ago  209.6 MB
ubuntu           14.04    99ec81b80c55  4 weeks ago  266 MB
ubuntu           latest   99ec81b80c55  4 weeks ago  266 MB
ubuntu           trusty   99ec81b80c55  4 weeks ago  266 MB
```
* `docker rmi`		/删除镜像
* `docker build`		/构建镜像
* `docker history`	/列出镜像层构建历史

#### docker container
* `docker run`		/启动容器

	```
	  -i	开启容器内stdin
  	  -t	分配伪tty终端(便于运行交互式shell)
  	  -p　80	宿主机随机端口映射容器80端口
  	  -p 8080:80 宿主机制定端口映射容器80端口
	```
* `docker ps`		/列出docker进程
* `docker restart/start/stop containerId/name`		/重启、启动、停止容器
* `docker logs`		/查看容器内日志

	```
	  -f	实时查看日志
	  -t	时间戳
	```
* `docker top`		/列出容器内进程
* `docker exec`		/容器内运行进程
* `docker inspect`	/查看容器内部信息
* `docker rm`		/删除容器

	> 必须通过docker stop/kill停止容器

#### Dockerfile
* **FROM**		/设置基础镜像
* **MAINTAINER**	/主要维护者(邮件通知)
* **ENV**		/设置容器环境变量
* **RUN** <command>/["exec","param1","param2"]	/构建镜像时命令

	> <command> 在shell中运行　bin/bash -c
	["exec","param1","param2"] 指定运行终端

* **CMD**		/容器启动时命令
  * `CMD ["executable","param1","param2"] 使用 exec 执行，推荐方式`
  * `CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应用`
  * `CMD ["param1","param2"] 提供给 ENTRYPOINT 的默认参数`

	> 每个Dockerfile中只能有一个CMD命令，多条时最后一条执行

* **EXPOSE**	/暴露端口号
* **ADD** <src> <dest>	/复制src到容器的dest路径

	> <src>可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）

* **COPY** <src> <dest>	／复制本机src到容器dest

	> 在使用本地源目录时，推荐使用COPY

* **ENTRYPOINT**	/容器启动后命令
  * `ENTRYPOINT ["executable", "param1", "param2"]`
  * `ENTRYPOINT command param1 param2（shell中执行）`

	> 配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。
	每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，执行最后一个

* **VOLUMN** ["/data"]	/挂载点
* **WORKDIR** dest		/为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录



#### 参考
* 《第一本Docker书》
* [《Docker从入门到实战》](http://udn.yyuap.com/doc/docker_practice)

