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

- [工作环境](#工作环境)
- [制作](#制作)
- [遇到的问题](#遇到的问题)

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

        $ febootstrap -g "Core" -i yum -i setarch \
            centos4 \
            /tmp/centos4.7-i386-image \
            http://vault.centos.org/4.7/os/i386/

    命令解释：

      - `-g` 镜像所需要安装的工具：把-g后面的参数传递给yum并使用*groupinstall*实现安装

      - `-i` 镜像所需要安装的工具：把-i后面的参数传递给yum并使用*install*实现安装

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
        ADD CentOS-Base.repo /etc/yum.repos.d/

        # Fix RPM database error
        RUN rm /var/lib/rpm/*
        RUN rpm --initdb

        # Clean yum cache
        RUN yum clean all

        ENTRYPOINT ["linux32"]
        CMD ["/bin/bash"]

2. 执行docker build

    复制[CentOS-Base.repo](http://vault.centos.org/4.9/CentOS-Base.repo)（目前在centos4上可用的下载源，之前系统自带的已经无法使用）
    内容并保存到Dockerfile同一目录下。

    将centos4.7-i386-image.tar.gz文件放到Dockerfile同一目录下，然后执行

        $ docker build -t centos:4.7-i386 .

    **`至此就可以愉快的使用centos4.7的镜像啦！`**

---

#### 遇到的问题

1. 安装vim时提示`No package vim available`

    需要改成`vim-enhanced` or `vim-common` or `vim-minimal`

    **`安装后使用的命令是vim，而不是vi`**

2. 使用yum命令时报错：

        bash-3.00# yum
        rpmdb: /var/lib/rpm/Packages: unsupported hash version: 9
        error: cannot open Packages index using db3 - Invalid argument (22)
        error: cannot open Packages database in /var/lib/rpm
        Traceback (most recent call last):
          File "/usr/bin/yum", line 29, in ?
            yummain.main(sys.argv[1:])
          File "/usr/share/yum-cli/yummain.py", line 80, in main
            base.getOptionsConfig(args)
          File "/usr/share/yum-cli/cli.py", line 170, in getOptionsConfig
            self.doConfigSetup(fn=opts.conffile, root=root)
          File "__init__.py", line 82, in doConfigSetup
          File "config.py", line 273, in __init__
          File "config.py", line 385, in _getsysver
        TypeError: rpmdb open failed

    看了[在CentOS6下制作CentOS5的镜像](http://www.opstool.com/article/318)发现别人也有遇到类似的问题，
    按照原作者的解决方法问题得到解决。

    `为什么会出现rpmdb的报错？`

    因为我们使用centos6的yum安装包，导致镜像里的 /var/lib/rpm/Packages 文件的DB格式不能被老的centos4的rpm读取。
    我们需要重建rpmdb，方法如下：

        # Fix up RPM database
        rm /var/lib/rpm/*
        rpm --initdb

    因此将Dockerfile做了修改。

3. 安装Core packages

    febootstrap可以使用`-g`选项安装groups内容，`-g Core`将安装包含bash, vi, tar, gzip等命令以及一些基本libs。

    因此也能解决问题1中的vim安装问题。**`而且不但可以使用vim命令，同时可以使用vi命令`**

    所以将

        $ febootstrap -i bash -i yum -i wget -i iputils -i iproute -i man -i vim-enhanced \
            -i openssh-server -i openssh-clients -i tar -i gzip \
            centos4 \
            /tmp/centos4.7-i386-image \
            http://vault.centos.org/4.7/os/i386/

    修改为：

        $ febootstrap -g "Core" -i yum -i setarch \
            centos4 \
            /tmp/centos4.7-i386-image \
            http://vault.centos.org/4.7/os/i386/

    如果没有`-i yum`，创建出来的镜像虽然有yum相关的lib，但是无法使用yum命令

    `-i setarch`是为了支持linux32命令

---

#### 参考：

- [Docker基础镜像-从iso到image](http://blog.csdn.net/s1234567_89/article/details/50698111)
- [在CentOS6下制作CentOS5的镜像](http://www.opstool.com/article/318)
- [febootstrap(8) - Linux man page](https://linux.die.net/man/8/febootstrap)

**在此对原文作者表示感谢！**

---
