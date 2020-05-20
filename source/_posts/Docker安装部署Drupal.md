---
title: Docker安装部署Drupal
index_img: /img/2020-05-19/drupal.jpg
banner_img: /img/2020-05-19/drupal-bg.jpg
banner_img_height: 60
banner_mask_alpha: 0.5
date: 2020-05-19 15:01:35
categories: 
- drupal
tags:
- cms
- php
---

### Docker安装部署Drupal

---

#### 需要的环境

* docker
* docker-compose

---

#### 介绍

* Docker

> 什么是Docker

docker允许你将你的应用和它依赖的环境打包到一个叫做容器的标准单元内。

你的应用，比如说druapl网站，这个应用依赖于web服务器（nginx/apache）和数据库（mysql/mariadb/postgresql）。

通常来说，如果你想要基于现有的网站重新部署一个新的网站副本，你需要通过操作系统的包管理器比如apt-get或者yum来手动部署依赖并且配置环境，安装扩展等等，这些都是很繁琐很复杂的过程。

当然，你也可以使用一些譬如Puppet, Ansible, Salt, Chef等编排工具来自动化你的安装和配置过程。

或者你也能可以使用完全不一样的方法：容器虚拟化Docker。通过使用docker，你可以打包你的应用到容器里面，并且可以部署在几乎所有的机器上，你可以把应用部署在你的手提电脑上，或者生产服务器上，或者你个人的vps上，甚至在vagrant里面。
  Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中,然后发布到任何流行的Linux机器或Windows 机器上,也可以实现虚拟化,容器是完全使用沙箱机制,相互之间不会有任何接口。

![容器让你从细节中抽象出来](/img/2020-05-19/this_is_docker.png)

  一个完整的Docker有以下几个部分组成：
  
1. DockerClient客户端
2. Docker Daemon守护进程
3. Docker Image镜像
4. DockerContainer容器

---

* Drupal

  Drupal是一套开源的内容管理平台，是一个基于PHP语言编写的开发型CMF（内容管理框架），拥有多种功能，可以用来建设从个人网站到大型社区网站。Drupal的架构由三大部分组成：内核、模块、主题。三者通过Hook机制紧密的联系起来。
  
  Drupal包括以下的功能：
  
  * Blog
  * 协同写作平台
  * 论坛
  * 电子报
  * 相册
  * 文件的上传与下载
  * 全文搜索
  * 多角色权限管理
  * 模块化
  * 主题引擎
  * 多语言支持
  
> 6个理由使用docker部署drupal

1. 托管无关

万一你想要把网站迁移到其他的主机提供商，使用docker的话就很容易做到，因为你的网站都是一些容器集合，而这些容器可以运行在各种机器上面。

2. 克隆 

如果你想要启动一个新的环境，例如开发环境或者测试环境，那么使用docker很容易实现，你只需要使用同一个镜像启动另外的容器就行了。

3. 环境一致

我们都知道这么一类问题，你在开发环境中不能重现某一些神秘的bug，最终发现是因为缺失某一些扩展或者扩展的版本不一致导致的。如果使用docker的话，诸如此类的环境问题就不存在了。

4. 隔离

如果你有多个网站或者一个网站的多个实例部署在同一台服务器的花，那么将它们隔离是非常重要的。你也不想因为其中一个应用的高负载影响到其他的应用。尽管隔离的实现比较棘手，但是docker的容器之间是相互隔离的。

5. 优化基础设施

毫无疑问的是我们通过配置nginx，php，mysql来优化我们的应用性能，如果你不擅长这些，那么你能够从docker hub中获取别人已经制作好的容器镜像。

6. 可扩展 

每个容器都可以被纵向扩展，比如增加更多的资源，也可以被横向扩展，比如克隆更多的容器和使用负载均衡。
  
---
####  准备

![容器部署项目目录结构图](/img/2020-05-19/drupal_project.png)

* 创建指定项目目录 DrupalProject
* 在 DrupalProject 下新建 .env docker-compose.yml **文件** 及 drupal **文件夹**
* 在 drupal 下新建  drupal-install.sh **文件** 及 mysql nginx.conf web **文件夹**
* 在 nginx.conf 下新建 drupal.test.conf【根据域名自定义以.conf结尾文件名文件】

#### 下载drupal8安装文件

基于Docker-compose正常情况下上面的drupal8 container会下载安装文件并放到对应的文件夹里，但是有时候会下载出错或者并没有下载文件，所以创建我们可以手动下载drupal并放到对应的目录下。
大家可以到[Drupal官网](https://www.drupal.org/download)下载，也可以通过执行下面到文件下载。

> 创建 drupal-install.sh 文件

```
# drupal-install.sh
#! /bin/bash
[[ -f .env ]] && source .env
if [ $DRUPAL_VERSION ]
then 
    echo "Start with Drupal version ${DRUPAL_VERSION}"
else
    echo "Drupal version is not defined. Set the default version to 8.6.12"
    DRUPAL_VERSION='8.6.12'
fi
mkdir web
curl -fSL "https://ftp.drupal.org/files/projects/drupal-${DRUPAL_VERSION}.tar.gz" -o drupal.tar.gz
mv drupal.tar.gz web
cd web
tar -zx --strip-components=1 -f drupal.tar.gz
rm drupal.tar.gz
```

> 执行上面的文件 ，下载drupal

```$xslt
sh drupal-install.sh
```


#### 创建docker-compose.yml配置文件

```

# docker-compose.yml
version: "3"

services:
  web:
    image: nginx:$NGINX_TAG
    container_name: "${PROJECT_NAME}_nginx"
    ports:
      - $NGINX_PORT
    volumes:
      - "./drupal/web:/var/www/html"
      - "./drupal/nginx.conf/drupal.test.conf:/etc/nginx/conf.d/default.conf"    
    depends_on:
      - php 
    networks:
      drupal-net:
        ipv4_address: 192.158.0.4
  php:
    image: drupal:$DRUPAL_TAG
    container_name: "${PROJECT_NAME}_drupal"
    volumes:
      - "./drupal/web:/var/www/html"
    restart: always
    privileged: true 
    depends_on:
      - mysql
    networks:
      drupal-net:
        ipv4_address: 192.158.0.3
  mysql:
    image: mysql:$MYSQL_TAG
    container_name: "${PROJECT_NAME}_mysql"
    command: mysqld --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_ROOT_PASSWORD: $DB_ROOT_PASSWORD
      MYSQL_USER: $DB_USER
      MYSQL_PASSWORD: $DB_PASSWORD
      MYSQL_DATABASE: $DB_NAME
    volumes:
      - "./drupal/mysql:/var/lib/mysql"
    ports:
      - $DB_PORT
    restart: always
    networks:
      drupal-net:
        ipv4_address: 192.158.0.2
      
networks:
  drupal-net:
    ipam:
      config:
        - subnet: 192.158.0.0/16
```

#### 创建.env文件

```$xslt
# .env
PROJECT_NAME=drupal8
DRUPAL_VERSION=8.8.3
DRUPAL_TAG=latest
NGINX_TAG=alpine
NGINX_PORT=80:80

###  database ###
MYSQL_TAG=8
DB_NAME=drupal
DB_USER=admin
DB_PASSWORD=admin
DB_ROOT_PASSWORD=root
DB_PORT=33060:3306
```

#### 创建 drupal.test.conf 【nginx】配置文件

```
upstream drupal_php
{
    server 192.158.0.3:80;
    #server 192.158.0.5:80;
    #server 192.158.0.6:80;
}
server {
    listen       80;
    server_name  drupal.test;
	
	location / {
	    proxy_set_header Host $host;
        proxy_pass http://drupal_php/;
    }
}

```

#### 容器服务命令 [在DrupalProject目录下执行]

> 启动容器

```
docker-compose up -d
```

容器启动以后，便可使用 drupal.test 【nginx配置域名及端口】 成功访问！

> 停止容器

```
docker-compose stop
```

> 删除容器

```
docker-compose rm
```

---

### Drupal安装

> 访问域名 drupal.test 跳转至安装页面

![容器部署项目目录结构图](/img/2020-05-20/drupal_install_1.png)

> 安装语言 选择 English ,选择中国将会出现以下（图）错误提示

![容器部署项目目录结构图](/img/2020-05-20/drupal_install_q_1.png)

> 基于标准（Standard）/极简(Minimal)/案例(Demo) 选择安装

![容器部署项目目录结构图](/img/2020-05-20/drupal_install_2.png)

> 数据库配置，基于 .env 文件配置 

```
* 数据库名 【Database name】  => drupal
* 数据库用户名 【Database username】  => admin
* 数据库密码 【Database password】  => admin
```

```
* 数据库访问地址 【Host】  => 192.158.0.2 
* 数据库访问端口 【Port number】  => 3306
* 数据库表前缀  【Table name prefix】 => （可留空）
```

![容器部署项目目录结构图](/img/2020-05-20/drupal_install_3.png)

> 站点信息配置

* Site name 【站点名称】
* Site email address 【站点邮箱地址】
* Username 【管理员用户名】
* Password 【管理员用户密码】
* Confirm password 【确认管理员用户密码】
* Email address 【管理员邮箱地址】
* Default country 【默认国家】
* Default time zone 【默认时区】


![容器部署项目目录结构图](/img/2020-05-20/drupal_install_4.png)

> Drupal安装完成！
![容器部署项目目录结构图](/img/2020-05-20/drupal_install_5.png)