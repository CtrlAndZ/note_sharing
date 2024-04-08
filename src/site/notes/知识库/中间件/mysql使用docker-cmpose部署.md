---
{"dg-publish":true,"permalink":"/知识库/中间件/mysql使用docker-cmpose部署/"}
---

创建一个mysql目录用作部署资料目录
``` shell
mkdir mysql
cd mysql # 进入该目录，后续工作在这个目录下展开
```

创建需要挂载到宿主机的目录
``` shell
mkdir data
```

创建临时容器，拷贝挂载目录下的数据到宿主机映射目录下
``` shell
docker run -itd --name mysql mysql:latest bash
```

``` shell
docker cp mysql:/var/lib/mysql ./data
```

删除临时容器
``` shell
docker rm -f mysql
```

编写docker-compose文件
``` shell 
vim docker-compose.yml
```

``` yml
version: '3'
services:
  mysql:
    image: mysql:latest
    restart: always
    hostname: mysql
    container_name: mysql
    ports:
      - 3306:3306
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: Ztt@123456 # 数据库密码，按需修改
    networks:
      - my_network # docker网络，没有需要提前创建：`docker network create my_network`
    volumes:
      - ./data:/var/lib/mysql
    command:
      --max_connections=1000
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --default-authentication-plugin=mysql_native_password

networks:
  my_network:
    external: true
```

启动
``` shell
docker-compose up -d
```