---
layout: post
title: "Docker基础镜像-ubuntu下制作CentOS4.7"
description: ""
category: 笔记
tags: [Docker, CentOS]
---
{% include JB/setup %}

需要使用CentOS4.7 32位的Docker镜像，但是在[Docker hub](https://hub.docker.com)上未找到。看到[official的
CentOS镜像](https://hub.docker.com/r/library/centos/)(centos5+)是通过GitHub上[sig-cloud-instance-images](https://github.com/CentOS/sig-cloud-instance-images/)项目进行制作的，
并搜索了一些别人的方案制作了CentOS4.7的Docker image。

## 工作环境

**`Ubuntu14.04系统`**

#### 1. 安装Docker

    $ curl -fsSL https://get.docker.com/ | sh

#### 2. 配置docker命令权限

    $ sudo usermod -aG docker $your_user

**`PS`:** 需要退出当前用户重新登录或重启以使权限生效

## 制作

#### 1. 制作CentOS4.7-i386系统镜像

> CentOS6.x系统上可以使用febootstrap工具制作系统镜像，因此可以借助此工具快速制作

1. 下载CentOS6.x的Docker镜像

    我选用了centos6.8

        $ docker pull centos:6.8

2. 运行centos

        $ docker run -it --name centos6.8 centos:6.8

    `此时您将进入centos环境`

3. 安装febootstrap

        $ yum update -y
        $ yum install febootstrap -y

4. 制作文件镜像

        $ febootstrap -i bash -i yum -i wget -i iputils -i iproute -i man -i vim-enhanced \
            -i openssh-server -i openssh-clients -i tar -i gzip \
            centos4 \
            /tmp/centos4.7-i386-image \
            http://vault.centos.org/4.7/os/i386/

    命令解释：

      - `-i` 镜像所需要安装的工具：把-i后面的参数传递给yum来实现安装

      - `centos4.7-i386-image` 是指定镜像文件保存路径

      - `centos4` 是centos版本

      - `http://vault.centos.org/4.7/os/i386/` 源镜像地址，也可以使用iso文件挂载到本地， 在[http://vault.centos.org/](http://vault.centos.org/)上可以找到centos各个版本的镜像

5. 压缩打包镜像文件用于制作Docker镜像

        $ cd /tmp/centos4.7-i386-image/
        $ tar zcvf /tmp/centos4.7-i386-image.tar.gz .

6. 从Docker容器中拷贝出文件镜像到主机

    `重新开启一个终端窗口执行`

        $ docker cp {containerID}:/tmp/centos4.7-i386-image.tar.gz ~/

#### 2. 制作Docker镜像

1. Dockerfile

        FROM scratch

        MAINTAINER GreenWang(greenwangcl@gmail.com)

        ADD centos4.7-i386-image.tar.gz /

        #ENTRYPOINT ["linux32"]
        CMD ["/bin/bash"]

2. 执行docker build

    将centos4.7-i386-image.tar.gz文件放到Dockerfile同一目录下，然后执行

        $ docker build -t centos:4.7-i386 .

    **`至此就可以愉快的使用centos4.7的镜像啦！`**

---

#### 参考：

- [Docker基础镜像-从iso到image](http://blog.csdn.net/s1234567_89/article/details/50698111)

**在此对原文作者表示感谢！**

---
