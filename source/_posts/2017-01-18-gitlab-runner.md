---
title: 安装Gitlab Runner
date: 2017-01-18 11:56:32
category: 
- memo
tags:
- 备忘
- GIT
- 持续集成
---

# Gitlab CI 简介

Gitlab中集成了CI (Continuous Integration：持续集成) 和CD (Continuous Delivery：持续交付) 来方便用户测试、构建、部署代码。它是Gitlab的一部分，用户可以在 [Gitlab.com](https://gitlab.com/) 上免费使用，同时也包含在了开源的Gitlab社区版和付费的Gitlab企业版中。

## Gitlab CI具有如下特性：

- **多平台**：您可以在任何支持Go语言的平台上运行，例如：Unix、Windows、OSX等。
- **多语言**：构建脚本是通过命令行驱动的，可以支持诸如Java、PHP、Ruby、C等任何语言。
- **稳定**：您的构建操作可以运行在其他机器上，而不是Gitlab上。
- **并行构建**：为了加快构建速度，Gitlab CI将任务分别运行在多个不同机器上。
- **实时日志**：可以通过链接查看到实时更新的构建日志。
- **版本化的测试**：每个人都可以对`.gitlab-ci.yml`文件提交修改以确保每个分支都能够得到所需要的测试。
- **管道**：您可以为每个阶段定义多个任务并且触发其他构建操作。
- **弹性运行**：可以自动增加或者减少虚拟机的数量以保证所有的构建操作都立即执行并且代价最小。
- **编译好的文件**：您可以上传二进制代码以及其他编译好的文件，并且能够浏览以及下载他们。
- **本地测试**：Gitlab中有多个线程池，您可以利用它们运行本地测试。
- **Docker支持**：您可以很容易的将其他的Docker容器服务集成进来作为测试的一部分，并且能够构建docker镜像。


Gitlab CI是Gitlab的一部分，它是一个带有api的web应用程序，可以将运行状态都保存在数据库中，除了Gitlab的功能外，它也能管理项目的构建过程，并且提供了一个很友好的用户界面。

Gitlab Runner是一个执行构建操作的应用程序，他能够被单独部署到其他机器上，通过API与Gitlab CI协作运行。Gitlab Runner可以被部署到任何支持Go语言二进制文件运行的环境中，例如Linux、OSX、Windows、FreeBSD以及Docker。它也能够测试任何编程语言书写的代码，例如.Net、Java、Python、C、PHP等等。

为了运行测试功能，您需要至少一个Gitlab实例和一个Gitlab Runner。

# 安装Gitlab Runner

安装Gitlab Runner可以有三种方法，通过Docker安装、下载二进制文件手动安装以及使用Gitlab提供的rpm/deb包通过包管理系统进行安装。我们使用第三种方式，也是Gitlab官方推荐的安装方式，目前Gitlab支持Debian、Ubuntu、RHEL以及CentOS的包管理工具安装。

如果您想使用Docker runner，那么必须要在安装Gitlab Runner之前安装它。
```
curl -sSL https://get.docker.com/ | sh
```

通过 apt-get 或者 yum 添加Gitlab官方软件源。
```
# For Debian/Ubuntu
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.deb.sh | sudo bash

# For RHEL/CentOS
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
```

### 覆盖APT设置-仅Debian系统

自从Debian Stretch开始，Debian维护者添加了一个和我们软件包具有相同名称的一个他们的官方软件包，并且默认情况下官方的软件仓库拥有优先权，如果您想使用Gitlab的软件包，那么必须手动设置软件包的安装源，最佳方法就是添加一个额外的配置文件覆盖掉原来的配置。接下来的每一个版本的更新，无论是手动更新还是自动更新，都将会使用同样的软件源。

```
cat > /etc/apt/preferences.d/pin-gitlab-runner.pref <<EOF
Explanation: Prefer GitLab provided packages over the Debian native ones
Package: gitlab-ci-multi-runner
Pin: origin packages.gitlab.com
Pin-Priority: 1001
EOF
```

# 

### 安装`gitlab-ci-multi-runner`：
```
# For Debian/Ubuntu
sudo apt-get install gitlab-ci-multi-runner

# For RHEL/CentOS
sudo yum install gitlab-ci-multi-runner
```

### 注册runner

此处需要使用一个token，使用管理员账号登陆gitlab并到`http://gitlab.exmaple.com/admin/runners` 可以查到，详细的说明可以通过查阅 [runner文档](http://doc.gitlab.com/ce/ci/runners/README.html) 学习。

```
sudo gitlab-ci-multi-runner register

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://gitlab.com
Please enter the gitlab-ci token for this runner
xxx
Please enter the gitlab-ci description for this runner
my-runner
INFO[0034] fcf5c619 Registering runner... succeeded
Please enter the executor: shell, docker, docker-ssh, ssh?
docker
Please enter the Docker image (eg. ruby:2.1):
ruby:2.1
INFO[0037] Runner registered successfully. Feel free to start it, but if it's
running already the config should be automatically reloaded!
```

执行完上述命令之后，使用管理员账号登陆gitlab并到`http://gitlab.exmaple.com/admin/runners`即可查看到已经注册好的gitlab runner。

