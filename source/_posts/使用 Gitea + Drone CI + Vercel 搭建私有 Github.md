---
title: 使用 Gitea + Drone CI + Vercel 搭建私有 Github
toc: true
categories: DevOps
tags: [ 'DevOps', 'Github', 'CI/CD', 'Gitea' , 'Drone' , 'Vercel' , 'Git' ]
excerpt: 我觉得是时候考虑自己部署一个类似 Github 的 Git 托管工具，周末尝试了使用 Gitea + Drone CI + Vercel 搭建私有的 Github
date: 2022-03-21 15:53:04
updated: 2022-03-21 15:53:04
---
### 介绍

在[阿里云流水线自动构建部署 Hexo 实践](https://aint.tech/2021/07/05/%E9%98%BF%E9%87%8C%E4%BA%91%E6%B5%81%E6%B0%B4%E7%BA%BF%E8%87%AA%E5%8A%A8%E6%9E%84%E5%BB%BA%E9%83%A8%E7%BD%B2-Hexo-%E5%AE%9E%E8%B7%B5/) 这篇文章里，曾讲过因众所周知的问题， 被迫把项目从 Github 迁出，并辗转迁移了好几个平台，最终使用阿里云 Codeup + flow 流水线实现了 Git 仓库可视化管理和 CI/CD 持续集成部署。

时近一年，flow 也暴露出了种种问题：

* 可视化的配置流程很棒，可是与项目解耦，没法直接在项目文件里查看到 flow 做了什么，常常需要打开 Codeup 的页面查看
* Codeup 的页面总给人一种卡顿不流畅的感觉，并且时不时就要重新登录
* 非阿里云的服务器需要安装一个脚本，但脚本稳定性不佳，不定期会掉线
* flow 虽然目前免费，但官方介绍以后会商业化，后期难免增加使用成本

综上所述，我觉得是时候考虑自己部署一个类似 Github 的 Git 托管工具，在[寿总](https://861204.xyz/)的安利下，周末尝试了使用 Gitea + Drone CI + Vercel 搭建私有的 Github。


### 要求

* 拥有公网 IP、域名、SSL 证书（如不满足要求，可尝试在本地使用 Gitea + Drone CI）
* 熟悉 `Docker` 及 `docker-compose`
* 熟悉 `Git` 基本命令
* 对 `CI/CD` 有一定了解


### 技术总结

#### 1、部署 Gitea

Gitea 看名字就大概能知道是做什么的，作为轻量级的 Git 托管工具，有着占用资源少，运行速度快的优点，同时也是知名项目 Gogs 的分支。

不过也正是因为 Gitea 的轻量级，CI/CD 持续集成部署之类功能缺失，需要和其他软件合作实现功能，不像 Gitlab 大而全一步到位，具体后面会提到。

Gitea 官方给出了和其他同类软件的[横向对比](https://docs.gitea.io/zh-cn/comparison/)，可以参考找到自己最合适的 Git 托管工具。  

<br />

Gitea 部署还是比较简单的，官方给出了 docker-compose [文件模板](https://docs.gitea.io/zh-cn/install-with-docker/)。

默认模板使用 SQLite 作为数据库，考虑到 SQLite 后期数据量大会有性能问题，改用了带 MySQL 数据库的 docker-compose。

```dockerfile
version: "3"

networks:
  gitea:
    external: false

services:
  server:
    image: gitea/gitea:1.16.4
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - DB_TYPE=mysql
      - DB_HOST=db:3306
      - DB_NAME=gitea
      - DB_USER=gitea
      - DB_PASSWD=gitea
    restart: always
    networks:
      - gitea
    volumes:
      - /home/appdata/gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3121:3000"
      - "3122:22"
    depends_on:
      - db
  
  db:
    image: mysql:8
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=gitea
      - MYSQL_DATABASE=gitea
    networks:
      - gitea
    volumes:
      - /home/appdata/gitea/mysql:/var/lib/mysql
```

首先，我把 WebUI `3000` 端口映射到了 `3121`，使用 Nginx 反向代理解析 `git.aint.tech` 域名到服务器备用；ssh 端口映射到了 `3122`，需要在云托管商的安全组（本地服务器为路由器或网关端口转发）开放 `3122` 端口。

同时把 Gitea `volumes` 的 `/data` 映射到了宿主机的 `/home/appdata/gitea` 目录，持久化 Gitea 产生的文件；把 MySQL `volumes` 的 `/var/lib/mysql` 映射到了宿主机的 `/home/appdata/gitea/mysql` 目录，持久化 MySQL 产生的文件。

PS： `gitea/gitea:latest` 拉下来并非最新版本，指定版本号即可；如果使用 ARM 架构，需把 db 的 `images` 替换为 `ubuntu/mysql`

<br />

使用 `docker-compose up -d` 运行后，访问 `git.aint.tech` 打开 `初始配置` 页面：

![初始配置](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220305758.png)

改动上图三个位置，公网解析过来的域名，以及映射的 SSH 服务端口，因为使用 Docker 构建，所以不需要改动 HTTP 服务端口。

点击安装后，顺利进入 Gitea 主界面。

<br />

因主要是我自己使用，还需修改些配置，打开 Gitea 目录下的 `conf/app.ini`（官方[配置说明](https://docs.gitea.io/zh-cn/config-cheat-sheet/)） ：

* 在 [server] 下添加 `LANDING_PAGE = explore`，指定默认首页为 explore
* 在 [service] 下把 `DISABLE_REGISTRATION` 值改为 true，禁止注册
* 在 [openid] 下把 `ENABLE_OPENID_SIGNIN` 和 `ENABLE_OPENID_SIGNUP` 值改为 false，禁止 OpenID 登录

修改后保存文件，重启 Gitea Docker 容器，部署完成。

<br />

#### 2、部署 Drone CI

Gitea 部署好后，迁移了几个项目使用了一番，体验不错。不过前面也提到 Gitea 比较轻量级，当我想要做 CI/CD 的时候，Gitea 就不支持了。上网检索解决方案，Drone CI 似乎是个不错的选择。

选定方案，当然是想尝试使用 Docker 和 Gitea 部署到同一台设备上，不过Drone CI 官网给出了一段[提示](https://docs.drone.io/server/provider/gitea/)：

> Please note we strongly recommend installing Drone on a dedicated instance. We do not recommend installing Drone and Gitea on the same machine due to network complications, and we definitely do not recommend installing Drone and Gitea on the same machine using docker-compose.

虽然官方建议物理安装不要使用 docker-compose，并且不要把  Gitea 和 Drone 跑在同一台服务器上，不过这两条友好的建议我都无视了，莽了再说。

<br />

Drone CI 包含了 drone-server 和 drone-runner，drone-server 负责 WebUI 及一些核心程序，drone-runner 负责部署时运行用例，弄清楚了依赖关系，尝试使用 docker-compose 来部署：

```dockerfile
version: '3'
services:
  drone-server:
    image: drone/drone:latest
    ports:
      - 9080:80
    volumes:
      - /home/appdata/drone:/data:rw
    restart: always
    environment:
      - DRONE_GITEA_SERVER=
      - DRONE_GITEA_CLIENT_ID=
      - DRONE_GITEA_CLIENT_SECRET=
      - DRONE_RPC_SECRET=
      - DRONE_SERVER_HOST=
      - DRONE_SERVER_PROTO=https

  drone-agent:
    image: drone/drone-runner-docker:latest
    restart: always
    depends_on:
      - drone-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:rw
    environment:
      - DRONE_RPC_PROTO=http
      - DRONE_RPC_HOST=drone-server
      - DRONE_RPC_SECRET=
      - DRONE_RUNNER_NAME=drone-runner
      - DRONE_RUNNER_CAPACITY=4
```

Drone CI 的部署参数比较多，讲解几个关键的地方（官方[配置说明](https://docs.drone.io/)） ：

1、把 `80` 端口映射到 `9080`，使用 Nginx 反向代理解析 `drone.aint.tech` 域名到服务器，在 `DRONE_SERVER_HOST` 填入域名

2、`DRONE_GITEA_SERVER` 填写 Gitea 域名 `https://git.aint.tech/`

3、`DRONE_GITEA_CLIENT_ID` 和 `DRONE_GITEA_CLIENT_SECRET` 需在 Gitra 里申请：

打开 `个人设置` -> `应用`，填写应用名称 `drone`，重定向 URL `https://drone.aint.tech/login`

创建应用后，把 `客户端ID` 和 `客户端密钥` 分别填入 `DRONE_GITEA_CLIENT_ID` 和 `DRONE_GITEA_CLIENT_SECRET`

![创建应用](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220309510.png)

4、`DRONE_RPC_SECRET` 为 Drone 共享密钥，这用于验证与服务器的 RPC 连接。必须为 drone-server 和 drone-runner 提供相同的密钥值，使用 `openssl rand -hex 16` 生成密钥后分别在 drone-server 和 drone-runner 填入

5、`DRONE_RUNNER_CAPACITY` 为限制 drone-runner 的并发数量，根据服务器性能评估填写

<br />

使用 `docker-compose up -d` 运行后，访问 `drone.aint.tech` 打开 Drone 控制台，跳转到 Gitea 授权并完善注册信息

![应用授权](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220309811.png)


进入并激活从 Gitea 同步过来的一个仓库

![激活仓库](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220310124.png)

回到 Gitea 点开对应仓库的 webhook，可以看到 Drone 已经创建好了一个钩子
![Web 钩子](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220310534.png)


向仓库根目录添加配置文件`.drone.yml`，并 git 提交

```sh
kind: pipeline
type: docker
name: default
   
steps:
- name: test
  image: alpine
  commands:
  - echo hello
  - echo world
```

回到 Drone 查看，成功触发了一次构建
![构建成功](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220311175.png)


#### 3、部署 Vercel

咕咕咕，下次再更
