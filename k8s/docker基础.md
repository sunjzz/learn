docker-ce 安装

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum list docker-ce --show-duplicates
sudo yum install -y docker-ce
sudo systemctl enable docker
sudo systemctl start docker
vim /etc/docker/daemon.json
```

配置

```
vim /etc/docker/daemon.json

{
    "graph": "data/docker",	#工作目录
    "storage-driver": "overlay2", #存储驱动
    "insecure-registries": ["registry.access.redhat.com", "quay.io"], #私有仓库
    "registry-mirrors": ["https://xfbr090w.mirror.aliyuncs.com"], # 阿里云加速镜像源
    "bip": "172.7.5.1/24",	# 172开头 + 宿主机的最后两位 + 1/24 生产经验，快速定位
    "exec-opts": ["native.cgroupdriver=systemd"], # cgroup 是 google2017写到内核里用于控制cpu和内存的方法, cgroup驱动是systemd
    "live-restore": true # docker容器引擎死掉 用容器引擎起来的容器还能存活 不依赖容器引擎服务
}
```

重载服务

```
sudo systemctl daemon-reload
sudo systemctl start docker

# 启动失败，仔细检查daemon.json文件是否有误。sudo systemctl reset-failed docker
```

检查服务

```
sudo docker info
sudo docker run hello-world
四步：
To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
```

仓库地址

```
https://hub.docker.com

sudo docker login docker.io

cat ~/.docker/config.json

里面auth base64加密的字符串  echo 字符串|base64 -d 明文


```

## 镜像管理

搜索镜像

```
sudo docker search alpine # alpine linux的发行版

```

下载镜像

```
sudo docker pull alpine # 拉取镜像
sudo docker pull alpine:3.10.3 # 拉取指定版本

docker.io/library/alpine:latest
镜像结构：registry_name/repository_name/image_name:tag_name
```

查看本地镜像

```
docker image
docker image ls
```

给镜像打标签

```
docker tag bf756fb1ae65 docker.io/jiangzhengzhong/alpine:v3.x
```

推送镜像

```
sudo docker push docker.io/jiangzhengzhong/alpine:v3.x
```

删除镜像

 ```
sudo docker rmi docker.io/jianghub/alpine:v3.x  # 删除tag
sudo docker rmi bf756fb1ae65
sudo docker rmi -f bf756fb1ae65 # 强制删除 
 ```

## 容器管理

查看本地容器

```
sudo docker ps -a
```

### 启动容器

```
sudo docker run [OPTION] IMAGE [COMMAND] [ARG...]
OPTION:
    -i: 表示启动可交互容器，并持续打开标准输入
    -t：表示使用终端关联到容器的标准输入输出
    -d：表示将容器放置后台运行
    -rm：退出后即删除容器
    -name：表示定义容器唯一名称
IMAGE： 表示要运行的镜像
COMMAND： 表示启动容器时要运行的命令


```

启动一个交互式容器

```
sudo docker run -it --name demo docker.io/library/alpine:latest /bin/sh
/ # cat /etc/issue 
Welcome to Alpine Linux 3.13
Kernel \r on an \m (\l)

/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:0b:0b:02 brd ff:ff:ff:ff:ff:ff
    inet 172.11.11.2/24 brd 172.11.11.255 scope global eth0
       valid_lft forever preferred_lft forever
```

加上 `--rm` 参数，容器结束后自动删除

```
sudo docker run --rm --name demo2 docker.io/library/alpine:latest /bin/echo hello
```

启动一个非交互后台容器

```
sudo docker run -d --name demo3 docker.io/library/alpine:latest
sudo docker run -d --name demo3 docker.io/library/alpine:latest /bin/sleep 60

[vagrant@node01 docker]$ sudo docker ps -a
CONTAINER ID   IMAGE           COMMAND           CREATED              STATUS                          PORTS     NAMES
8e8553dd2d86   alpine:latest   "/bin/sleep 60"   7 seconds ago        Up 6 seconds                              demo4
d05e83dc384c   alpine:latest   "/bin/sh"         About a minute ago   Exited (0) About a minute ago             demo3
bef89248bb20   alpine:latest   "/bin/sh"         9 minutes ago        Exited (0) 8 minutes ago                  demo
cf61e2a85f89   hello-world     "/hello"          2 hours ago          Exited (0) 2 hours ago                    gallant_mclaren
[vagrant@node01 docker]$ ps -ef|grep sleep
root     26639 26617  0 15:41 ?        00:00:00 /bin/sleep 60
vagrant  26712  2111  0 15:42 pts/0    00:00:00 grep --color=auto sleep

宿主机能看到sleep这个命令的进程
```

进入容器

```
[vagrant@node01 docker]$ sudo docker exec -it 8e8553dd2d86 /bin/sh
/ # 
/ # cat /etc/issue 
Welcome to Alpine Linux 3.13
Kernel \r on an \m (\l)
```

停止 启动 重启容器

```
sudo docker stop 76fd376b61cc
sudo docker start 76fd376b61cc
sudo docker restart 76fd376b61cc
```

删除容器

```
sudo docker rm 76fd376b61cc
sudo docker rm -f demo5
```

修改容器

```
sudo docker run -d --name demo jianghub/alpine:v3.x /bin/sleep 600
sudo docker exec -it demo /bin/sh
echo 'hello,docker' > info
exit
sudo docker commit -p demo jianghub/alpine:v3.x_with_info	# -p参数表示这个保持快照 不接受这个动作期间别的数据修改
sudo docker images
sudo docker run -it --name mypine jianghub/alpine:v3.x_with_info /bin/sh
/ # cat info 
hello,docker
文件存在，持久化成功

for i in $(sudo docker ps -a|grep -i exit|awk '{print $1}');do sudo docker rm -f $i;done
删除所有已退出的容器


```

导出容器

```
sudo sh -c 'docker save 697bfb335045 > alpine:v3.x_with_info.tar'
```

导入容器

```
sudo docker load < alpine\:v3.x_with_info.tar
sudo docker tag 697bfb335045 jianghub/alpine:v3.x_with_info
sudo docker run -it --rm --name myalpine jianghub/alpine:v3.x_with_info /bin/sh
/ # cat info 
hello,docker
```

查看日志

```
[vagrant@node01 docker]$ sudo docker run hello-world
[vagrant@node01 docker]$ 
[vagrant@node01 docker]$ sudo docker ps -a
CONTAINER ID   IMAGE                  COMMAND            CREATED          STATUS                      PORTS     NAMES
41e1f29764c1   hello-world            "/hello"           4 seconds ago    Exited (0) 3 seconds ago              determined_wu
b175664b7721   jianghub/alpine:v3.x   "/bin/sleep 600"   25 minutes ago   Exited (0) 15 minutes ago             demo
[vagrant@node01 docker]$ sudo docker logs -f 41e1f29764c1	# -f 参数动态输出

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

## 高级操作

映射端口

```
docker run -p 容器外端口:容器内端口

sudo docker pull nginx
sudo docker tag f6d0b4767a6c jianghub/nginx:v1.18
sudo docker run --rm -d --name mynginx -p 81:80 jianghub/nginx:v1.18 
```

挂载数据卷

```
docker run -v 容器外目录：容器内目录

sudo docker run -d --rm --name myweb -p 82:80 -v /home/vagrant/html/:/usr/share/nginx/html jianghub/nginx:v1.18
sudo docker inspect myweb	
```

传递环境变量

```
docker run -e 环境变量key=环境变量value

[vagrant@node01 ~]$ sudo docker run --rm --name myenv jianghub/alpine:v3.x printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=4d0c02dda572
HOME=/root
[vagrant@node01 ~]$ sudo docker run --rm --name myenv -e E_OPTS=abc jianghub/alpine:v3.x printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=8b134c7dbd26
E_OPTS=abc
HOME=/root
[vagrant@node01 ~]$ sudo docker run --rm --name myenv -e E_OPTS=abc -e C_OPTS=123 jianghub/alpine:v3.x printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=5b642210c285
E_OPTS=abc
C_OPTS=123
HOME=/root
```

容器内安装软件

```
yum/apt-get等


[vagrant@node01 ~]$ sudo docker exec -it myweb /bin/sh
# tee /etc/apt/sources.list <<EOF
> # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
> deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
> # deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
> deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
> # deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
> deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
> # deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
> deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free
> # deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free
> EOF
# apt update
# apt install vim
[vagrant@node01 ~]$ docker ps -a
CONTAINER ID   IMAGE                  COMMAND                  CREATED             STATUS                   PORTS                NAMES
1a99941f16ac   jianghub/nginx:v1.18   "/docker-entrypoint.…"   About an hour ago   Up About an hour         0.0.0.0:82->80/tcp   myweb
[vagrant@node01 ~]$ sudo docker commit -p 1a99941f16ac jianghub/nginx:with_vim
[vagrant@node01 ~]$ sudo docker push jianghub/nginx:with_vim 
The push refers to repository [docker.io/jianghub/nginx]
569a2dc3da0b: Pushed 
85fcec7ef3ef: Mounted from library/nginx 
3e5288f7a70f: Mounted from library/nginx 
56bc37de0858: Mounted from library/nginx 
1c91bf69a08b: Mounted from library/nginx 
cb42413394c4: Mounted from library/nginx 
with_vim: digest: sha256:adc895bd84a51ec1c8c19731cc8e032f7f7ab0e6affc6646bad27caf3121a064 size: 1574
```



