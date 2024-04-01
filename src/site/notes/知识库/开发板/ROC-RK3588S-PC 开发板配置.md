---
{"dg-publish":true,"permalink":"/知识库/开发板/ROC-RK3588S-PC 开发板配置/"}
---


### 1. 烧录系统
https://wiki.t-firefly.com/zh_CN/ROC-RK3588S-PC/upgrade_firmware.html#jin-ru-sheng-ji-mo-shi

### 2.配置ssh密码
``` shell
sudo passwd root
```

#####  修改sshd配置文件

```plaintext
sudo vi /etc/ssh/sshd_config
```

- ##### 在文件中搜索定位PermitRootLogin
    

```plaintext
/PermitRootLogin
```

- ##### 将#PermitRootLogin prohibit-password 改为： PermitRootLogin yes
    

- ##### 重启sshd，使配置文件生效
    

```plaintext
systemctl restart sshd
```
### 3.修改软件源
``` shell
vim /etc/apt/sources.list
```
替换为下面内容：
``` shell
deb http://wiki.t-firefly.com/firefly-ubuntu-repo focal main
deb http://wiki.t-firefly.com/firefly-rk3588-repo focal main
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://repo.huaweicloud.com/ubuntu-ports/ focal main restricted
# deb-src http://repo.huaweicloud.com/ubuntu-ports/ focal main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://repo.huaweicloud.com/ubuntu-ports/ focal-updates main restricted
# deb-src http://repo.huaweicloud.com/ubuntu-ports/ focal-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://repo.huaweicloud.com/ubuntu-ports/ focal universe
# deb-src http://repo.huaweicloud.com/ubuntu-ports/ focal universe
deb http://repo.huaweicloud.com/ubuntu-ports/ focal-updates universe
# deb-src http://repo.huaweicloud.com/ubuntu-ports/ focal-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://repo.huaweicloud.com/ubuntu-ports/ focal multiverse
# deb-src http://repo.huaweicloud.com/ubuntu-ports/ focal multiverse
deb http://repo.huaweicloud.com/ubuntu-ports/ focal-updates multiverse
# deb-src http://repo.huaweicloud.com/ubuntu-ports/ focal-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://repo.huaweicloud.com/ubuntu-ports/ focal-backports main restricted universe multiverse
# deb-src http://repo.huaweicloud.com/ubuntu-ports/ focal-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu focal partner
# deb-src http://archive.canonical.com/ubuntu focal partner

deb http://repo.huaweicloud.com/ubuntu-ports/ focal-security main restricted
# deb-src http://repo.huaweicloud.com/ubuntu-ports/ focal-security main restricted
deb http://repo.huaweicloud.com/ubuntu-ports/ focal-security universe
# deb-src http://repo.huaweicloud.com/ubuntu-ports/ focal-security universe
deb http://repo.huaweicloud.com/ubuntu-ports/ focal-security multiverse
# deb-src http://repo.huaweicloud.com/ubuntu-ports/ focal-security multiverse
deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable
# deb-src [arch=amd64] https://download.docker.com/linux/ubuntu focal stable
deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu focal stable
# deb-src [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu focal stable
```

### 4.安装docker环境

在 Ubuntu 上安装 Docker 非常直接。我们将会启用 Docker 软件源，导入 GPG key，并且安装软件包。

首先，更新软件包索引，并且安装必要的依赖软件，来添加一个新的 HTTPS 软件源：

``` shell
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```

使用下面的 `curl` 导入源仓库的 GPG key：

``` shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

将 Docker APT 软件源添加到你的系统：

``` shell
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

现在，Docker 软件源被启用了，你可以安装软件源中任何可用的 Docker 版本。

01.想要安装 Docker 最新版本，运行下面的命令。如果你想安装指定版本，跳过这个步骤，并且跳到下一步。

``` shell
sudo apt update
sudo apt install docker-ce
```

#### 安装docker-compose

``` shell
sudo apt install docker-compose
```

#### 修改镜像源
1. 在/etc/docker下，新建daemon.json文件

``` shell
sudo touch daemon.json
```

2. 开daemon.json权限

``` shell
sudo chmod 777 daemon.json
```

3. 进入daemon.json输入镜像地址：

``` shell
vim daemon.json
```

然后输入（加速地址是我自己的）

``` yml
{
  "registry-mirrors": ["https://w9zcgj26.mirror.aliyuncs.com"]
}
```

并保存退出即可。

4. 重启

``` shell
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl status docker
```

### 5.固定本机ip

`ifconfig` 查看本机ip对应的网络名称

![image.png](https://cdn.jsdelivr.net/gh/CtrlAndZ/imgded@main/blog/20230821141713.png)

编写对应名称的配置文件，命名规则如下：
``` shell
vim /etc/netplan/eth0-network.yaml
```

内容：
``` yml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.110.98/24]
      gateway4: 192.168.110.200
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
```

保存更改：
``` shell
sudo netplan apply
```

重启：
``` shell
reboot restart
```

### 6.安装emqx

``` shell
mkdir /home/docker/emqx /home/docker/emqx/logs
```

``` shell
cd /home/docker/emqx
vim docker-compose.yml
```

``` yml
version: "3"
services:
  emqx:
    image: "emqx/emqx:5.0.20"
    container_name: "emqx_2"
    ports:
     - 18084:18083
     - 9002:1883
     - 8088:8081
     - 8886:8883
    volumes:
     - ./logs:/opt/emqx/log
    restart: always
```

启动:
``` shell
docker-compose up -d
```

### 7.安装中控程序（已弃用，选下面 2024/03/29 更新方式）

安装配置文件:
``` shell
mkdir /home/docker/ztt-main/data/config
cd /home/docker/ztt-main/data/config
vim config.properties
```

config.properties:
``` properties
# 机场sn码
sn=37B2613H8
# 型号
model=ZT30
```

安装java程序:
``` shell
mkdir /home/docker/ztt-main /home/docker/ztt-main/data /home/docker/ztt-main/data/tmp
```

``` shell
cd /home/docker/ztt-main
vim docker-compose.yml
```

```yml
version: "3"
services:
  ztt-main:
    image: ztt-main
    container_name: ztt-main
    ports:
      - "9003:9003"
    volumes:
      - ./data:/data
    restart: always
    privileged: true
```

Dockerfile:
``` shell
vim Dockerfile
```

```shell
FROM adoptopenjdk/openjdk8
MAINTAINER bingo
ADD ztt-main.jar app.jar
EXPOSE 9003
ENV TZ=Asia/Shanghai
RUN sh -c 'touch /app.jar' && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
ENTRYPOINT ["java","-jar","app.jar"]
```

构建docker镜像：
``` shell
docker build -t ztt-main .
```

启动:
``` shell
docker-compose up -d
```


### 8.安装rtty

安装依赖

``` shell
sudo apt install -y libev-dev libssl-dev
```

克隆rtty代码
``` shell
cd /root
git clone --recursive https://github.com/zhaojh329/rtty.git
```

编译

``` shell
cd rtty && mkdir build && cd build
cmake .. && make install
```

配置rtty开机自启动

rtty_start.sh：
``` shell
cd /root
vim rtty_start.sh
```

```shell
#!/bin/bash
sudo rtty -I 'drone_nest_003' -h '218.77.59.1' -p 5912 -a -v -d '第三台设备' -t 004fb10edc9549f86a7f90b5eaed14ac
```

``` shell
chmod 755 rtty_start.sh
```

rtty.service：
``` shell
cd /etc/systemd/system
vim rtty.service
```

``` shell
[Unit]
Description=rtty
After=network.target

[Service]
ExecStart=/root/rtty_start.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

``` shell
chmod 644 rtty.service
```

``` shell
systemctl daemon-reload
systemctl enable rtty.service
reboot
```


### 9.设置时区

``` shell
sudo apt-get install chrony
sudo systemctl stop chronyd
sudo timedatectl set-timezone Asia/Shanghai
sudo systemctl start chronyd
sudo systemctl enable chronyd
```


> 2024/03/29 更新

### 中控程序安装方式调整

安装配置文件:
``` shell
mkdir /home/docker/ztt-main/data/config
cd /home/docker/ztt-main/data/config
vim config.properties
```

config.properties:
``` properties
# 机场sn码
sn=37B2613H8
# 型号
model=ZT30
# 本地mqtt地址
mqttHost=127.0.0.1
# 本地mqtt端口
mqttPort=9002
```

安装java程序:
``` shell
mkdir /home/docker/ztt-main /home/docker/ztt-main/data /home/docker/ztt-main/data/tmp
```

``` shell
cd /home/docker/ztt-main
vim run.sh
```

run.sh是手动启动脚本，一般情况不使用，使用命令为`./run.sh start`、`./run.sh stop`
``` bash
#!/bin/bash

# 设置Java的路径，确保根据你的安装路径进行修改
JAVA_PATH="java"

# 设置JAR包的路径，替换成你的JAR包的实际路径
JAR_PATH="/home/docker/ztt-main/ztt-main.jar"

# 设置日志文件路径，替换成你希望保存日志的实际路径
LOG_FILE="/logs/ztt-main/log.log"

check_status() {
    if pgrep -f "$JAR_PATH" > /dev/null; then
        echo "Application is running."
        return 0
    else
        echo "Application is not running."
        return 1
    fi
}

case "$1" in
    "start")
        check_status
        if [ $? -eq 0 ]; then
            echo "Application is already running. Skipping start."
        else
            echo "Starting the application..."
            nohup $JAVA_PATH -jar $JAR_PATH > $LOG_FILE 2>&1 &
        fi
        ;;
    "restart")
        check_status
        if [ $? -eq 0 ]; then
            echo "Restarting the application..."
            pkill -f $JAR_PATH
            nohup $JAVA_PATH -jar $JAR_PATH > $LOG_FILE 2>&1 &
        else
            echo "Application is not running. Skipping restart."
        fi
        ;;
    "stop")
        check_status
        if [ $? -eq 0 ]; then
            echo "Stopping the application..."
            pkill -f $JAR_PATH
        else
            echo "Application is not running. Skipping stop."
        fi
        ;;
    "status")
        check_status
        ;;
    *)
        echo "Usage: $0 {start|restart|stop|status}"
        exit 1
        ;;
esac

exit 0

```

设置中控程序自启动
``` shell
cd /etc/systemd/system/
vim ztt-main.service
```

``` properties
[Unit]
Description=ztt-main
After=network.target

[Service]
ExecStart=/usr/bin/nohup /usr/bin/java -jar /home/docker/ztt-main/ztt-main.jar > /logs/ztt-main/log.log 2>&1 &
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

设置权限
``` shell
chmod 644 ztt-main.service
```

``` shell
systemctl daemon-reload
systemctl enable ztt-main.service
reboot
```


### RTK模块接入

#### BT-468 RTK 模块

设置波特率为38400（临时有效）
``` shell
sudo stty -F /dev/ttyS7 38400
```

数据读取测试
``` shell
cat /dev/ttyS7
```