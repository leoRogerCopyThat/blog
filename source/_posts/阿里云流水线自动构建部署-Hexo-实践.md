---
title: 阿里云流水线自动构建部署 Hexo 实践
date: 2021-07-05 09:47:42
updated: 2021-07-05 16:38:00
toc: true
categories: DevOps
tags: [ 'Hexo', 'DevOps', '阿里云', '云效', '流水线' ]
excerpt: 访问比 Github pages 快多了
---
Hexo 创建博客十分方便，接触过前端的同学基本都能快速上手，大佬们做的主题也比较丰富，所以断断续续一直在用，文章编写与发布体验也不错。但因为部署在 Github pages 上，在国内因为各种你懂的原因，访问体验一直不佳（因为懒，不想改程序适配 jsDelivr），便有了把博客迁移回国内的想法。

# 令人失望的 Coding pages
起初把目光聚焦在了国内闲置的阿里云小鸡，但要想使用 hexo deploy 发布到小鸡，得配置 git hooks 等一堆东西，用着没有 Gtihub actions 方便直观，并且还是得做一遍 git push 再  hexo deploy，不够优雅。老底子的 hexo generate 本地编译再 ftp 上传的方法就更不优雅了。

直接部署到小鸡方案被否决，便改变思路，找了找国内类似 Github pages 的服务，Gitee、腾讯工蜂、阿里云Code、Coding... 国内看似提供 Git 服务的网站众多，但提供 pages 服务仅有 Coding 一家，就试着迁移了过来。

刚开始的体验着实还不错，和 Github pages 的使用逻辑无异，迁移到了国内，访问速度也快了很多。但接下来 Coding 的一列骚操作，着实恶心：和腾讯云合并期间 pages 服务时断时续，代码审查服务导致经常部署失败，最致命的是加入了强制实名和把 pages 服务独立迁移到了香港，访问速度又堪忧了，无奈抛弃。

![Coding pages](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220315738.png)

# 阿里云版 Github actions
国内的 pages 服务走不通，便把目光又放回了那台闲置的小鸡，
此时我的博客建站需求已经十分明确了：
1、不使用访问贼慢的 Github pages（因为懒，不想改程序适配 jsDelivr）
2、不使用国内瞎搞的 pages 服务，部署在自己的小鸡
3、需要类似 Gtihub actions 服务实现 git push 后自动构建部署

这两天碰巧看到了阿里云的流水线服务，心头一动，搞不好这就是阿里云版的 actions，赶紧尝试一波。

## 选择模板
![流水线模板](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220316589.png)
流水线提供了各类语言的自动化部署模板，Hexo 基于 node，并且我的小鸡也在阿里云，就选择了这个模板先试试水。

## 流程配置
![流程配置](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220316944.png)
进入流程配置，顺序从左至右，整体逻辑清晰明了，添加代码源 -> 测试代码 -> 构建代码 -> 部署。
觉得个人静态博客的没测试代码的必要，直接改为 添加代码源 -> 构建 -> 部署，简单粗暴。

### 代码源配置
![添加代码源](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220323086.png)
流水线能从 Github、Gitee 甚至自建的 git 等各种地方选择代码源，兼容性不错。
我提前把代码先 git push 到了阿里云的 Codeup 上，在同一账号下，便可直接选择到代码仓库及分支，无缝衔接。
同时在选择代码源的界面，也可配置触发部署等条件，比如我设置了代码合并到 master 分支后，自动触发构建部署。

### 构建代码配置
```bash 流水线构建命令
# input your command here
yarn
yarn build
```
Node.js 构建的本质其实就是装好包依赖，并且编译代码，只不过这些操作从本地变成了在云上完成。
流水线会自动拉取前面设置好的代码源，并根据给定的命令进行编译。

```json hexo package.json
..."scripts": {
  "build": "hexo generate",...
..."dependencies": {
    "hexo": "^5.0.0",...
```
因为我在 Hexo 项目的 package.json 的 dependencies 里配置了 Hexo 及其所需依赖，所以无需再在构建环境里全局安装 hoxe，yarn 命令安装完依赖后直接 yarn build 即可。
最后设置好打包路径，Hexo 默认编译生成是在 public 下，所以只需要打包这个目录，然后上传构建物，给下一步的部署做准备。

### 部署配置
#### 添加主机
![主机部署](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220320125.png)


选择刚才生成的构建物，添加主机，填写构建物下载到小鸡内路径。因为我的小鸡是同一账号下的的ECS，直接选择即可。

![添加自有主机](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220320262.png)
如果是自建或其他云服务商的小鸡，则需要先根据提示安装一个脚本。

```bash production-install.sh 脚本片段
function check_python() {
  ...output=`python -V 2>&1`...
  ...if [[ ! $output == *"2.7"* ]]; then...
}
```
注：此处脚本有个坑，脚本会先使用 `python -V 2>&1` 来检查小鸡是否安装了 python，但我们一般用 yum 等包管理器安装的 python 创建的软链默认是带版本号的，就会导致脚本检测 python 失败，此时需要手动创建符合脚本检测的软链即可。
```bash 手动创建 python 软链
ln -s /usr/bin/python2 /usr/bin/python 
```
#### 部署脚本
```shell 部署脚本
rm -rf /home/appdata/nginx/www/blog.leo.red/*
tar zxvf /home/admin/app/LeoBlog.tgz -C /home/appdata/nginx/www/blog.leo.red/
docker restart nginx
```
接着来编写部署脚本，部署静态页面的操作比较简单，思路就是删除原来的静态内容，然后把下载的构建物解压到 nginx 的虚拟主机静态目录，然后重启 nginx。

至此，流水线的配置就全部完成。

# 铛铛铛，流水线开动！
配置好流水线后，就可以测试啦。

![部署概览](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220324911.png)

![构建日志](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220321920.png)

![主机部署](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220322647.png)

根据上面的配置，只要我把代码合并到 master 分支，自动化构建部署就被触发。

# 解放生产力
![博客构建部署成功](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203220322454.png)

至此，流水线开动完成~
以后只要提交博客代码，后续的工作完全自动化运行，解放生产力。
