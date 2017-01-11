---
layout: post
title: "DevOps学习笔记"
description: ""
category: 笔记
tags: [DevOps, CI/CD]
---
{% include JB/setup %}

最近一年来公司有谋求转型和提高效率的需求，在和同事的交流过程中想做一个软件开发服务类产品，能够服务于公司项目开发，
也想能让个人开发者能提高生产效率。让开发者专注与开发而不被一些其他事情所干扰。

这段时间接触了[Docker](https://www.docker.com/)， [Ansible](https://www.ansible.com/)等一些新技术，
也慢慢了解了DevOps的一些知识和理念，在此记录并分享下对DevOps的理解。主要涉及以下内容：

- [DevOps是什么](#devops是什么)

- [如何实施DevOps](#如何实施devops)

- [DevOps现状](#devops现状)

- [可能依然存在的问题](#可能依然存在的问题)

## DevOps是什么

一款应用在用户能使用之前都需要经历需求整理，开发，发布这三个阶段。开发团队需要不断满足用户的需求，并快速开发实现，
最终运维人员部署发布给用户使用。

然而在实际执行过程中会遇到一些挑战，一是如何快速开发出用户提出的需求，二是如何快速部署让用户使用并获得反馈。

![](/assets/img/dev-gap.png)

敏捷开发的出现缩小了上图所示的第一个隔阂，也就是商业需求和开发之间的隔阂，有效的加快了产品开发的周期和效率。

但是第二个隔阂，开发和运维之间的挑战还需要解决，于是DevOps的理念应用而生。

以前开发、测试、运维的工作模式是：

![](/assets/img/old-dev-workflow.png)

DevOps理念下开发、测试、运维的工作模式：

![](/assets/img/devops-workflow.png)

## 如何实施DevOps

DevOps只是一种理念，而在执行时只要符合这种理念可以采用适合自己的方法。通过深入了解自己的环境中需要解决的问题是什么，
再看看DevOps中的一些方法是不是可以起到事半功倍的作用。

实施上的一些建议：

- 标准化和自动化 - 使用模板和指定好的流程。自动化、尽量减少认为干预可以避免不必要的人为失误。使用像代码库一样的工具
记录并管理配置，在出现问题时可以快速找到问题所在。

- 独立的Dev/Test环境 - 在Dev环境中测试后在生产环境中实施。

- 测试自动化 - 方便在作出更改时能够更快的得到反馈，通过测试指标判断修改的影响。

- 开发人员能自己创建开发环境

而**持续集成(Continuous Integration)**、**持续交付(Continuous Delivery)**、**持续部署(Continuous Deployment)**
应该是DevOps理念具体实施方法。

**持续集成**

![](/assets/img/devops-ci.png)

持续集成强调开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。

**持续交付**

![](/assets/img/devops-cd.png)

持续交付在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境的「类生产环境」（production-like environments）中。比如，
我们完成单元测试后，可以把代码部署到连接数据库的 Staging 环境中更多的测试。如果代码没有问题，可以继续手动部署到生产环境中。

**持续部署**

![](/assets/img/devops-cdeploy.png)

持续部署则是在持续交付的基础上，把部署到生产环境的过程自动化。

## DevOps现状

我对DevOps的理解是：使用开发出一套适用于产品发布的自动化流程，避免人为操作，并可以持续的、高频率的交付。

因此实现DevOps应该是对现有工具的组装和流程的自动化。**`个人的认识，欢迎讨论`**

这次对DevOps现状的评价也基于以上个人的认识进行。

大家熟知的[GitHub](https://github.com)结合[Travis CI](https://travis-ci.org)/[Circle CI](https://circleci.com)等服务，
实现了从code hosting到CI、CD的一个公共平台，这为一些想将自身产品DevOps化的人提供了一个基础平台。使用者在具体使用时需要提供指定的build环境、
build操作（命令）、定义在不同阶段触发的操作。即可以进行DevOps化。

类似的平台还有[GitLab](https://gitlab.com)、[GoCD](https://www.gocd.io)、[Gogs](https://gogs.io)，这些可以自己搭建自己的
服务，便于一些比较在意信息/代码安全的用户使用。

使用这些平台（使用最多的是GitLab，GitHub，TravisCI）的过程中都需要对他们的工作流程、模式有较清晰的认识，并至少要掌握YAML基本语法进行相关配置，
GitLab还会涉及到Docker技术。所以在DevOps化的过程中依然需要学习和储备新知识，才能满足自身在发展过程中不断增加的复杂需求，也许这就是DevOps人才的
核心价值吧。

上述这些平台基本上都是国外的，而随着阿里云平台建立的[docker化持续交付](https://yq.aliyun.com/articles/32071)的构想，在不久的将来国内的用户
也会越来越接受和实施DevOps化。毕竟马大大为大家提供了这么多的工具。

## 可能依然存在的问题

1. 如何保证CI环境和Development环境保持一致，并可以统一的进行管理和更新？

2. 手机应用的Deploy无法向Web应用一样直接部署到自己的服务器上，如何解决市场发布的问题？

3. 如何可视化的让用户对自身进行DevOps化，或者说可视化的管理整个DevOps流程？

4. 是否可以将DevOps过程中使用者的积累让更多的用户可以使用？比如建立像GitHub这样的公共平台让使用者分享流程中的某个操作

    此处可以参考[Ansible Galaxy](https://galaxy.ansible.com/)对Ansible中`role`的做法

---

##### 参考资料
- [DevOps：从理念到实施](http://os.51cto.com/art/201404/436794.htm)
- [The Product Managers’ Guide to Continuous Delivery and DevOps](http://www.mindtheproduct.com/2016/02/what-the-hell-are-ci-cd-and-devops-a-cheatsheet-for-the-rest-of-us/)
- [云上应用docker化持续交付实践](https://yq.aliyun.com/articles/32071)
