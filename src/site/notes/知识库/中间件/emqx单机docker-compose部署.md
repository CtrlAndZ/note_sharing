---
{"dg-publish":true,"permalink":"/知识库/中间件/emqx单机docker-compose部署/"}
---

创建一个emqx目录用作部署资料目录
``` shell
mkdir emqx
cd emqx # 进入该目录，后续工作在这个目录下展开
```

运行一个临时容器，从容器中拷贝需要挂载到宿主机的数据到部署目录下
``` shell
docker run -itd --name emqx emqx/emqx:latest bash
```

``` shell
docker cp emqx:/opt/emqx/data .
docker cp emqx:/opt/emqx/etc .
docker cp emqx:/opt/emqx/log .
```

删除临时容器
``` shell
docker rm -f emqx
```

编写docker-compose文件
``` shell 
vim docker-compose.yml
```

``` yml
version: "3"
services:
  emqx:
    image: "emqx/emqx:latest"
    container_name: "emqx"
    ports:
     - 18083:18083 # web管理页面端口
     - 1883:1883 # mqtt协议端口
     - 8083:8083 # ws协议端口
    volumes:
      - ./conf:/opt/emqx/etc
      - ./data:/opt/emqx/data
      - ./logs:/opt/emqx/log
    networks:
      - my_network # docker网络，没有需要提前创建：`docker network create my_network`

networks:
  my_network:
    external: true
```

启动
``` shell
docker-compose up -d
```