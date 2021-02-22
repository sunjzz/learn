4组核心的Dockerfile指令

- USER/WORKDIR 指令

  docker里面进程使用的用户

  容器启动后所在的工作目录

  ```
  FROM jianghub/nginx:v1.18
  USER nginx
  WORKDIR /usr/share/nginx/html
  ```

- ADD/EXPOSE 指令

  ```
  FROM jianghub/nginx:v1.18
  ADD index.html /usr/share/nginx/html/index.html
  EXPOSE 80	# EXPOSE指令只能配合-P使用才有效果
  
  docker build . -t jianghub/nginx:v1.18_with_add
  docker run --rm -it --name mynginx -P jianghub/nginx:v1.18_with_add /bin/bash
  nginx -g "daemon off;"
  docker run --rm -d --name mynginx -P jianghub/nginx:v1.18_with_add
  ```

- RUN/ENV 指令

  ```
  FROM centos:7
  ENV VER 9.11.4-26.P2.el7_9.3
  RUN yum install bind-$VER -y
  
  docker build . -t centos7:with_run
  docker run --rm -it --name mycentos centos7:with_run /bin/bash
  [root@463b50876ff4 /]# rpm -qa|grep bind
  bind-libs-lite-9.11.4-26.P2.el7_9.3.x86_64
  bind-libs-9.11.4-26.P2.el7_9.3.x86_64
  bind-license-9.11.4-26.P2.el7_9.3.noarch
  bind-9.11.4-26.P2.el7_9.3.x86_64
  [root@463b50876ff4 /]# printenv
  PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
  PWD=/
  SHLVL=1
  HOME=/root
  VER=9.11.4-26.P2.el7_9.3
  ```

- CMD/ENTRYPOINT 指令

  ```
  FROM centos:7
  RUN yum install httpd -y
  CMD ["httpd", "-D", "FOREGROUND"]
  
  docker build . -t jianghub/centos:with_httpd
  docker run --rm -d --name myhttpd -p83:80 jianghub/centos:with_httpd
  
  
  FROM centos:7
  ADD entrypoint.sh /entrypoint.sh
  RUN yum install epel-release -q -y && yum install nginx -y
  ENTRYPOINT  /entrypoint.sh
  
  docker run --rm -d --name mynginx -p 84:80 jianghub/nginx:with_entrypoint
  docker exec -it mynginx /bin/bash
  
  ```

  

## Docker 网络模型

--net 指定

NAT（默认）

None

Host

联合网络 	--net=container:*ID*















意义：

内核3.8以上

统一了基础设施环境 硬件，操作系统版本

统一了程序打包方式

统一了程序部署方式

- java -jar --> docker run
- python manage.py runserver --> docker run
- npm run dev --> docker run



k8s 1.15版本

四组基本概念

- Pod/Pod控制器
- Name/Namespace
- Label/Label选择器
- Service/Ingress

Pod控制器

保证pod按照预期运行（副本数，生命周期，健康状态检查）

- Deployment
- DaemonSet
- ReplicaSet
- StatefulSet
- Job
- Cronjob

### 核心组件

配置存储中心 etc服务

主控（master）节点

- kube-apiserver服务
- kube-controller-manager服务
- kube-scheduler服务

运算（node）节点

- kube-kubelet服务

- kube-proxy服务

  三种流量转发方式

  - USerspace 废弃
  - Iptables 濒临废弃 net_filter
  - Ipvs 推荐 lvs性能比iptables要高效得多

CLI客户端

- kubectl

核心附件 9

- CNI网络插件 -->flannel/calico
- 服务发现用插件 -->coredns
- 服务暴露用插件 -->traefik
- GUI管理插件 -->Dashboard

网络 Pod网络 Service网络 Node网络

