# Docker Hub 镜像加速

国内从 Docker Hub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。

## 安装Docker
官方安装脚本：

```
curl -fsSL https://get.docker.com | sh
```

国内阿里云镜像

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

<details>
  <summary>安装Docker Compose</summary>
  
  ###  下载 Docker Compose:



运行以下命令来下载 Docker Compose：

```
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
添加可执行权限:
```
chmod +x /usr/local/bin/docker-compose
```
验证安装:
```
docker-compose --version
```

---
（可选）国内环境可[手动下载文件](https://github.com/docker/compose/releases)上传到`/usr/local/bin`目录，并重命名为`docker-compose`，然后增加执行权限。

---

</details>

##  Docker常用命令:

| 功能    | 命令 | 说明 |
|-------------|-------------------|----------------|
| 编译镜像  | `docker build -t 镜像名 .`      |   先`docker login`登录docker hub        |
| 推送镜像  | `docker push 用户名/镜像名`      |   需先标记镜像 `docker 本地镜像名 用户名/镜像名`        |
| 查看容器  | `docker ps`      |   `-a`查看包括已停止的容器         |
| 容器资源占用  | `docker stats`      |   查看所有容器资源占用         |
| 容器详细信息  | `docker inspect`      |  挂载看`Mounts`网络看`Networks`       |
| 进入容器内部  | `docker exec -it 容器名 /bin/sh`      |   结尾使用`/bash`也行        |
| 停止容器  | `docker stop`      |   `docker stop 容器名或ID`             |
| 启动容器  | `docker start`      | `docker start 容器名或ID`           |
| 重启容器  | `docker restart`      |  `docker restart 容器名或ID`          |
| 删除容器 | `docker rm`       |  `docker rm 容器名或ID`              |
| 查看镜像 | `docker images`   | `docker images 镜像名或ID`            |
| 删除镜像  | `docker rmi -f`   |  `docker rmi -f 镜像名或ID`          |
| 清除资源  | `docker system prune`   |  清除所有未使用资源`容器 网络 镜像 缓存`    |
| 删除所有镜像  | `docker rmi -f $(docker images -aq)`  |   删除所有镜像         |
| 删除所有容器  | `docker container prune -f`  |   删除所有已停止容器         |
| 停止所有容器  | `docker stop $(docker ps -aq)`  |   停止所有容器         |

---
[官方仓库](https://hub.docker.com/)

[官方文档](https://docs.docker.com/build/building/packaging/)

---

### vim编辑器

安装：`apt-get install vim`或者`apk add vim`

常用命令：

| 功能    | 命令 | 说明 |
|-------------|-------------------|----------------|
| 退出并保存  | `:wq`      |   命令模式中执行         |
| 只退出不保存  | `:q!`      |    命令模式中执行      |
| 清空所有内容  | `:%d`      |   命令模式中执行        |
| 粘贴时保持代码格式  | `:set paste`      |   执行后按`i`进入编辑模式             |
| 进入编辑模式  | `i`      | 命令模式中执行          |
| 退出编辑模式  | `esc`      |  编辑模式中执行         |
| 更改编码适配中文 | `:set encoding=utf-8`       |  执行后按`i`进入编辑模式              |


---

## 配置加速地址

> Ubuntu 16.04+、Debian 8+、CentOS 7+

创建或修改 `/etc/docker/daemon.json`：

```
sudo mkdir -p /etc/docker
```
```
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://dockerproxy.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.nju.edu.cn"
    ]
}
EOF
```
```
sudo systemctl daemon-reload
```
```
sudo systemctl restart docker
```


### 检查加速是否生效

查看docker系统信息 `docker info`，如果从结果中看到了如下内容，说明配置成功。

```console
Registry Mirrors:
 [...]
 https://docker.m.daocloud.io
```

对于 Mac 和 Windows 用户，直接在 Docker Desktop 系统设置中，配置 registry-mirrors 即可。

---
## 使用代理拉取镜像

#### 创建配置文件
```
sudo mkdir -p /etc/systemd/system/docker.service.d
```
```
sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf
```
#### 在文件中添加代理
```
[Service]
Environment="HTTP_PROXY=socks5://user:pass@127.0.0.1:1080"
Environment="HTTPS_PROXY=socks5://user:pass@127.0.0.1:1080"
```
#### 重启Docker
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

---
## 备用方法：直接传送镜像
国外服务器拉取镜像后打包压缩到本地，然后传输到国内服务器，`myimage`为镜像名
#### A服务器保存Docker镜像
```
docker save myimage > myimage.tar
```
#### 传送到B服务器
```
scp myimage.tar root@192.0.2.0:/home
```
然后输入B服务器root密码

#### B服务器加载Docker镜像

```
cd /home
```

```
docker load < myimage.tar
```
查看镜像
```
docker images
```
---

## Docker Hub 镜像测速

拉取镜像时，可使用 `time` 统计所花费的总时间。测速前记得移除本地的镜像。

例如：`time docker pull node:latest`

## 卸载Docker
```
sudo systemctl stop docker
sudo apt-get purge docker-ce docker-ce-cli containerd.io
sudo rm -rf /etc/docker /var/lib/docker
```

---
## Docker Hub 镜像加速列表
以下镜像站来源于互联网（感谢热心网友），可能出现宕机、转内网、关停等情况，建议同时配置多个镜像源。

#### 目前可用镜像加速

镜像 | 镜像加速地址 | 说明 | 其它加速
--- | --- | --- | ---
[DaoCloud 镜像站](https://github.com/DaoCloud/public-image-mirror) | `https://docker.m.daocloud.io` | |  Docker Hub、GCR、K8S、GHCR、Quay、NVCR 等
[Docker 镜像代理](https://dockerproxy.com) | `https://dockerproxy.com` | | Docker Hub、GCR、K8S、GHCR
[百度云](https://cloud.baidu.com/doc/CCE/s/Yjxppt74z#%E4%BD%BF%E7%94%A8dockerhub%E5%8A%A0%E9%80%9F%E5%99%A8) | `https://mirror.baidubce.com` | | Docker Hub
[南京大学镜像站](https://doc.nju.edu.cn/books/35f4a) | `https://docker.nju.edu.cn` | | Docker Hub、GCR、GHCR、Quay、NVCR 等
[中科院软件所镜像站](https://mirror.iscas.ac.cn/mirror/docker.html) | `https://mirror.iscas.ac.cn` | | Docker Hub
[阿里云](https://cr.console.aliyun.com/) | `https://<your_code>.mirror.aliyuncs.com` | 需登录分配 | Docker Hub 未同步最新源镜像

## 参考链接

+ https://docs.docker.com/registry/recipes/mirror/
+ https://gist.github.com/y0ngb1n/7e8f16af3242c7815e7ca2f0833d3ea6
