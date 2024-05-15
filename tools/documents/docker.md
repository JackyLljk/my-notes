

## 1. Docker

### 1.1 介绍

> Docker 是一个应用打包、分发、部署的工具

**Docker 是一个轻量的虚拟机**

1. 支持绝大部分的系统，包括 Windows、Linux
2. 只虚拟软件所需的运行环境，性能好
3. 一个命令就可以自动部署好所需环境
4. 不同系统的部署方式相同，稳定性好

**打包**：把软件运行所需的依赖、第三方库、软件等打包到一起，变成一个「Docker镜像」（”安装包“）

**分发**：把打包好的镜像上传到一个「镜像仓库」，其他人可以非常方便的获取和安装

**部署**：使用镜像一个命令就可以运行应用，自动模拟一样的运行环境（无论是 Windows/Mac/Linux）

**镜像**：类似软件安装包，方便传播和安装

**容器**：软件安装后的状态，每个软件的运行环境都是独立的、隔离的，即称之为容器

<br>

### 1.2  基本用法

**部署流程**：`自己电脑上开发、测试` -> `打包为Docker镜像` ->` 在服务器上一个命令部署好`

**使用**

1. 开源软件和提供私有部署的应用，方便他人安装
2. 快速安装测试/学习软件，快速安装，减少配置时间
3. 多个版本软件共存，不污染系统
4. 跨平台使用



## 2. 常用命令









## 3. Docker 部署 Go 项目

#### MySQL

```bash
# 在本地 13306 端口运行一个名为 mysql830，root用户名密码为 root1234 的 MySQL 容器环境
docker run --name mysql830 -p 13306:3306 -e MYSQL_ROOT_PASSWORD=root1234 -d mysql:8.3.0

# 启动 MySQL client 连接上面的 MySQL 环境，密码为 root1234
docker run -it --network host --rm mysql mysql -h127.0.0.1 -P13306 --default-character-set=utf8mb4 -uroot -p
```

#### Redis

```bash
docker run --name redis724 -p 6379:6379 -d redis:7.2.4

docker run -it --network host --rm redis:7.2.4 redis-cli
```







## 参考

[Docker 快速入门](https://docker.easydoc.net/doc/81170005/cCewZWoN/lTKfePfP)