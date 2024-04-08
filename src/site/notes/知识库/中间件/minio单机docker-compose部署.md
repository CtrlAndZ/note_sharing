---
{"dg-publish":true,"permalink":"/知识库/中间件/minio单机docker-compose部署/"}
---

创建一个minio目录用作部署资料目录
``` shell
mkdir minio
cd minio # 进入该目录，后续工作在这个目录下展开
```

创建需要挂载到宿主机的目录
``` shell
mkdir config
mkdir data
```

创建临时容器，拷贝挂载目录下的数据到宿主机映射目录下
``` shell
docker run -itd --name minio minio/minio bash
```

``` shell
docker cp minio:/data/ ./data
docker cp minio:/root/.minio/ ./config
```

删除临时容器
``` shell
docker rm -f minio
```

编写docker-compose文件
``` shell 
vim docker-compose.yml
```

``` yml
version: "3"
services:
  emqx:
    image: minio/minio
    hostname: "minio"
    ports:
      - 9000:9000 # api 端口
      - 9001:9001 # 控制台端口
    environment:
      MINIO_ACCESS_KEY: ztt    #管理后台用户名，这里是示例，按实际情况修改
      MINIO_SECRET_KEY: Ztt@123456 #管理后台密码，最小8个字符，这里是示例，按实际情况修改
      TZ: Asia/Shanghai
    volumes:
      - ./data:/data               #映射当前目录下的data目录至容器内/data目录
      - ./config:/root/.minio/     #映射配置目录
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
    command: server --console-address ':9001' /data  #指定容器中的目录 /data
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