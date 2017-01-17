---
title: 使用Gitlab Pages建立个人博客
date: 2016-12-12 10:50:36
category: 
- memo
tags:
- 备忘
- GIT
- GitLab
---

# 关于Gitlab Pages

与Github Pages相似，Gitlab Pages也是一个用来托管静态文件的服务，由Gitlab提供，通过与Gitlab CI和Gitlab Runner集成，将用户个人、组织以及项目的页面部署到静态文件服务当中。

# 开始使用Gitlab Pages

 通常来说，有两种类型的pages

 - 用户页面(`username.exmaple.io`)或者组织页面(`groupname.exmaple.io`)
 - 项目页面(`username.exmaple.io/projectname`)或者(`groupname.exmaple.io/projectname`)

在一个Gitlab实例中，用户名或者组织名是唯一的，所以我们可以将其当作命名空间使用，下面表格展示了不同类型的Gitlab Pages与他们的项目名称的关系以及最终URL的展现形式：

|Gitlab Pages类型|Gitlab中的项目名称|网站URL|
| --- | --- | --- |
|用户页面|`username.exmaple.io`|`http(s)://username.exmaple.io`|
|组织页面|`groupname.exmaple.io`|`http(s)://groupname.exmaple.io`|
|用户项目页面|`projectname`|`http(s)://username.exmaple.io/projectname`|
|组织项目页面|`projectname`|`http(s)://groupname.exmaple.io/projectname`|

# 使用Gitlab Pages的基本流程

简单的说，使用Gitlab Pages 需要以下几步：

- 从管理员处得到Gitlab Pages的域名(gitlab.com的对应域名为gitlab.io)，这一步非常重要，所以您必须确保首先获得正确的域名。
- 创建一个项目。
- 向该工程根目录中推送一个`.gitlab-ci.yml`文件。
- 建立一个Gitlab Runner用来构建您的服务。

# 搭建个人博客

下面要在Gitlab.com上搭建免费的个人博客，由于我的用户名是mlei，所以建立一个工程，名为mlei.gitlab.io。并将此工程克隆至本地。

将github pages工程的hexo分支内源码复制到该工程目录下，然后新建一个.gitlab-ci.yml文件，其内容如下：

```
image: node:4.2.2

pages:
  cache:
    paths:
    - node_modules/

  script:
  - npm install hexo-cli -g
  - npm install
  - hexo generate
  artifacts:
    paths:
    - public
  only:
  - master
```

提交所有更改，然后将代码库push到远程，Gitlab会自动识别.gitlab-ci.yml中的配置并执行其中的操作，执行完毕后，访问`http://mlei.gitlab.io`即可访问我的博客。

# 同步Github Pages内容

每次提交Github Pages之后都要将博客源代码文件复制到Gitlab Pages的工程下面，然后再提交，显然这种重复的体力劳动是不需要人力去完成的，我们可以在travis CI中设置相应的操作，让其自动将代码提交至Gitlab版本库中，但是Gitlab提供了一个更加简便快捷的方法：镜像版本库(Mirror repository)。

点击工程的设置按钮，找到`Mirror  Repository`菜单栏，点击进入，一共提供了两种选择，一共是将该版本库当作其他项目的镜像，每小时同步一次内容，另一种是将该版本库当作主版本库，每次提交之后，将其内容同步至其他版本库，我们此处用的是前者。

只要将Mirror repository的复选框勾选，然后填入我的Github pages的地址，确认即可，此时，自动同步功能已经开启。

