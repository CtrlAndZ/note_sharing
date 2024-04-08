---
{"dg-publish":true,"permalink":"/知识库/中间件/nacos单机docker-compose部署/"}
---

创建一个nacos目录用作部署资料目录
``` shell
mkdir nacos
cd nacos # 进入该目录，后续工作在这个目录下展开
```

创建临时容器，拷贝挂载目录下的数据到宿主机映射目录下
``` shell
docker run -itd --name nacos nacos/nacos-server:latest bash
```

``` shell
docker cp nacos:/home/nacos/data .
docker cp nacos:/home/nacos/logs .
docker cp nacos:/home/nacos/conf .
```

删除临时容器
``` shell
docker rm -f nacos
```

编写docker-compose文件
``` shell 
vim docker-compose.yml
```

``` yml
version: '3'
services:
  nacos:
    image: nacos/nacos-server:latest
    container_name: nacos
    ports:
      - "8848:8848"
    networks:
      - my_network # docker网络，没有需要提前创建：`docker network create my_network`
    environment:
      - MODE=standalone
    volumes:
      - ./data:/home/nacos/data
      - ./logs:/home/nacos/logs
      - ./conf:/home/nacos/conf

networks:
  my_network:
    external: true
```

启动
``` shell
docker-compose up -d
```