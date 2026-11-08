# Docker 教程（按主题重组版）

> 用途：学习笔记 · 定位：把 95 集视频按"知识结构"重新组织，不是按集数流水账
> 覆盖：01–95 集（黑马 Docker 课程）
> **标注说明**：
> - ✅ = 看文字就够，不用看视频
> - 🎬 = **必须看视频**（GUI 操作、界面演示、纯图解）
> 本机是 Arch Linux，未装 Docker；安装部分会同时给 Arch 和视频里的 CentOS 两套命令。

---

## 0. 先看全貌：95 集到底在讲什么

**整个课程就四层能力，从下往上：**

```
┌─────────────────────────────────────┐
│ 第四层：编排与监控                     │
│   Compose（多容器）· Portainer · CIG  │
├─────────────────────────────────────┤
│ 第三层：构建自己的镜像                 │
│   Dockerfile · commit · 推送到仓库     │
├─────────────────────────────────────┤
│ 第二层：跑起来、管起来                 │
│   容器命令 · 数据卷 · 网络             │
├─────────────────────────────────────┤
│ 第一层：知道它是什么                   │
│   镜像/容器/仓库 · 为什么比 VM 快      │
└─────────────────────────────────────┘
```

**95 集的知识地图：**

| 集数 | 主题 | 建议 |
|---|---|---|
| 01–08 | Docker 是什么、三要素 | ✅ 文字够 |
| 09–10 | 平台入门、架构图解 | 🎬 看视频（图解为主） |
| 11–12 | 安装、镜像加速 | ✅ 文字够（命令照抄） |
| 13–16 | hello-world、镜像命令 | ✅ 文字够 |
| 17 | ubuntu 容器说明 | 🎬 看视频（演示） |
| 18–22 | **容器命令 A–E** | ✅ 文字够（重点，命令表） |
| 23–25 | 镜像分层、commit | ✅ 文字够 |
| 26 | 发布到阿里云 | 🎬 看视频（网页操作） |
| 27–28 | 私有库 | 🎬 看视频（演示） |
| 29–33 | **数据卷** | ✅ 文字够（要点清楚） |
| 34–39 | 装 tomcat/mysql/redis | ✅ 文字够（命令照抄） |
| 40 | 高级篇简介 | ⏭️ 跳过 |
| 41–44 | 主从复制、分布式存储算法 | ✅ 文字够（概念） |
| 45–47 | redis 集群配置 | 🎬 看视频（步骤长） |
| 48–50 | 集群错误、路由、cluster check | ✅ 文字够 |
| 51–55 | 主从容错、扩缩容 | 🎬 看视频（演示） |
| 56 | 分布式存储总结 | ✅ 文字够 |
| 57–62 | **Dockerfile** | ✅ 文字够（指令表） |
| 63–65 | 微服务打包部署 | 🎬 看视频（演示） |
| 66–74 | **网络**（5 种模式） | ✅ 文字够 |
| 75–77 | 自定义网络 | 🎬 看视频（演示） |
| 78–86 | **Compose** | ✅ 文字够（yml 结构）+ 🎬 编排演示 |
| 87–94 | **Portainer / CIG** | 🎬 **必须看视频**（全 GUI） |
| 95 | 总结 | ⏭️ 可跳 |

---

# 第一部分：Docker 是什么（01–08）

## 1. 它解决什么问题

### 1.1 一个真实的痛点

你写了一个 Java 项目，在自己电脑上跑得好好的。发给同事：

```
同事：运行报错
你：我这儿好好的啊
同事：你什么版本？
你：JDK 17、MySQL 8.0.35、Redis 7.2、Maven 3.9...
同事：我 JDK 8、MySQL 5.7...
```

**这就是"环境不一致"问题。** 部署到服务器时更麻烦——服务器上缺这缺那。

### 1.2 Docker 的答案

**把你的程序 + 它依赖的所有环境，打包成一个"集装箱"。**

```
传统方式：服务器上装 JDK、装 MySQL、装 Redis、配环境变量...
          → 换台机器再来一遍，还可能版本对不上

Docker：  把程序和环境一起打包成镜像 → 任何装了 Docker 的机器
          一条命令就能跑起来，环境完全一致
```

**"一次构建，到处运行"**——这就是 Docker 最大的卖点。

## 2. 三要素（必背）

| 要素 | 是什么 | 类比 |
|---|---|---|
| **镜像 Image** | 一个只读的模板，包含程序和运行环境 | 安装包 / 菜谱 |
| **容器 Container** | 镜像运行起来的实例 | 已安装的软件 / 做好的菜 |
| **仓库 Registry** | 存放镜像的地方 | 应用商店 / 网盘 |

**关系**：`镜像 --run--> 容器`，`镜像 <--push/pull--> 仓库`。

```bash
docker pull nginx              # 从仓库拉镜像
docker run -d nginx            # 用镜像启动容器
docker push myimage:v1         # 把镜像推到仓库
```

## 3. 和虚拟机的对比

| | 传统虚拟机 | Docker 容器 |
|---|---|---|
| 虚拟化什么 | 整套硬件（CPU/内存/硬盘） | 只虚拟操作系统层 |
| 有没有完整 OS | 有（每个 VM 一个完整系统） | 没有（共享宿主机内核） |
| 启动速度 | 分钟级 | **秒级** |
| 体积 | GB 级 | MB 级 |
| 隔离性 | 强 | 较弱（但够用） |
| 资源占用 | 高 | 低 |

**类比**：
- **虚拟机** = 每个应用住一栋独立别墅（有自己完整的房子、水电、地基）
- **容器** = 每个应用住一个隔间（共用同一栋楼的地基和水电，但各自独立）

**为什么 Docker 比 VM 快？** 因为它不需要启动一个完整的操作系统，只是启动一个进程，并给它套上"隔离"的壳（namespace + cgroups）。这就是第 14 集讲的。

## 4. 架构（Client–Server）

```
┌─────────┐     REST API     ┌──────────────────┐
│ docker  │ ───────────────► │  dockerd 守护进程  │
│ (CLI)   │                  │  ├─ 容器运行时      │
└─────────┘                  │  ├─ 镜像管理        │
                             │  └─ 网络/存储       │
                             └──────────────────┘
```

**你敲的 `docker run` 只是一个客户端命令，真正干活的是后台的 `dockerd` 守护进程。**

**记住两件事**：
1. Docker 是 C/S 架构
2. 容器本质是宿主机上的一个**进程**（被隔离了）

> 🎬 **第 09–10 集**讲的是平台入门和架构图解，有大量示意图，**建议看视频**，文字描述不如动画直观。

---

# 第二部分：安装（11–12）

## 5. Arch Linux 安装（本机）

```bash
# 1. 安装
sudo pacman -S docker docker-compose

# 2. 启动并设置开机自启
sudo systemctl enable --now docker

# 3. 把当前用户加入 docker 组（这样不用每次 sudo）
sudo usermod -aG docker $USER

# 4. 重新登录（或执行）
newgrp docker

# 5. 验证
docker run hello-world
```

**Step 3 很关键**：不加进 docker 组，每次都得 `sudo docker`，很烦。

## 6. CentOS 7 安装（视频里的命令）

```bash
# 1. 安装依赖
sudo yum install -y yum-utils

# 2. 加 Docker 官方源
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 3. 安装
sudo yum install -y docker-ce docker-ce-cli containerd.io

# 4. 启动
sudo systemctl start docker
sudo systemctl enable docker
```

**两套命令的区别只是包管理器**（pacman vs yum），后面的 docker 命令完全一样。

## 7. 镜像加速器（国内必配）

**默认从 Docker Hub 拉镜像，国内很慢。** 配置国内镜像源：

```bash
# 1. 创建/编辑配置
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<'EOF'
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.nju.edu.cn"
  ]
}
EOF

# 2. 重启生效
sudo systemctl daemon-reload
sudo systemctl restart docker

# 3. 验证配置生效
docker info | grep -A 5 "Registry Mirrors"
```

**注意**：镜像源经常挂，如果一个不通就换下一个。**2024 年后很多公共镜像源已下线**，实际使用时可能需要自己找可用的。

> 🎬 **第 12 集**演示了配置文件怎么写、怎么验证，**看一眼视频**比看文字快。

---

# 第三部分：核心操作（13–22）

## 8. hello-world 到底做了什么

```bash
docker run hello-world
```

**这一行背后发生了 4 件事：**

```
① 在本地找 hello-world 镜像 → 没有
② 去远程仓库拉（pull）到本地
③ 用镜像创建并启动一个容器
④ 容器执行完就退出，输出提示信息
```

**输出里的三句话**：
1. "Hello from Docker!" —— 说明容器跑起来了
2. "This message shows that your installation appears to be working correctly." —— 安装成功
3. 介绍了 Docker 的基本工作流程

## 9. 镜像命令

```bash
# 查看本地镜像
docker images
docker images -a          # 显示所有（含中间层）

# 搜索（Docker Hub 上搜，用的少）
docker search nginx

# 拉取镜像
docker pull nginx               # 最新版
docker pull nginx:1.25          # 指定版本（推荐，别用 latest）
docker pull ubuntu:22.04

# 删除镜像
docker rmi nginx                # 删单个
docker rmi -f nginx             # 强制删
docker rmi -f $(docker images -aq)   # 删全部（危险）

# 查看镜像详情
docker inspect nginx

# 导出/导入镜像（离线传输用）
docker save -o nginx.tar nginx:latest
docker load -i nginx.tar
```

**一个坑**：镜像被容器占用时 `rmi` 会失败，要先删容器再删镜像。

## 10. 容器命令（最重要的一节）

### 10.1 创建并运行

```bash
# 最常用：交互式运行（进去玩）
docker run -it --name myubuntu ubuntu /bin/bash

# 后台运行
docker run -d --name mynginx -p 8080:80 nginx

# 运行完自动删除
docker run --rm -it ubuntu /bin/bash
```

**参数拆解：**

| 参数 | 含义 | 记法 |
|---|---|---|
| `-i` | 保持标准输入打开 | interactive |
| `-t` | 分配一个终端 | tty |
| `-it` | 两个一起用，交互式 | 最常用 |
| `-d` | 后台运行，返回容器 ID | daemon |
| `--name` | 给容器起名字 | 不起名会随机分配 |
| `-p 宿主机:容器` | 端口映射 | `-p 8080:80` 外面 8080 → 里面 80 |
| `-P` | 随机端口 | 大写 P |
| `-v 宿主:容器` | 挂载数据卷 | volume |
| `-e KEY=VALUE` | 环境变量 | environment |
| `--rm` | 容器退出后自动删除 | 临时容器用 |

**`-p 8080:80` 的映射关系：**

```
你的浏览器访问 localhost:8080
        ↓
宿主机 8080 端口
        ↓（docker 转发）
容器内 80 端口（nginx 在监听）
```

### 10.2 查看容器

```bash
docker ps              # 只看运行中的
docker ps -a           # 看所有（含已停止）
docker ps -q           # 只显示 ID
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

### 10.3 启动 / 停止 / 重启

```bash
docker start 容器名或ID      # 启动已停止的容器
docker stop 容器名或ID       # 停止（发送 SIGTERM，优雅）
docker kill 容器名或ID       # 强制停止（SIGKILL）
docker restart 容器名或ID    # 重启
```

**区别**：`stop` 会等容器自己收尾，`kill` 直接干掉。**一般用 stop。**

### 10.4 进入容器（两种方式）

**方式一：exec（推荐）**

```bash
docker exec -it 容器名 /bin/bash
```

**exec 是"新开一个终端进程"**，退出不会影响容器运行。

**方式二：attach（不推荐）**

```bash
docker attach 容器名
```

**attach 是"连到容器的主进程"**，退出（Ctrl+C）会**把容器一起停掉**。

**记住：日常用 `exec`，别用 `attach`。**

### 10.5 看日志

```bash
docker logs 容器名              # 看全部日志
docker logs -f 容器名           # 实时跟踪（follow）
docker logs --tail 100 容器名   # 最后 100 行
docker logs -t 容器名           # 带时间戳
```

### 10.6 文件拷贝

```bash
docker cp 容器名:/路径/文件 /宿主机/路径/     # 从容器拷出来
docker cp /宿主机/文件 容器名:/路径/         # 拷进容器
```

**注意**：容器重启后，容器内的文件修改会丢失（除非用了数据卷）。`docker cp` 只适合临时操作。

### 10.7 删除容器

```bash
docker rm 容器名          # 删除已停止的容器
docker rm -f 容器名       # 强制删除运行中的
docker rm -f $(docker ps -aq)   # 删所有容器（危险）
```

## 11. 命令速查表

| 操作 | 命令 |
|---|---|
| 拉镜像 | `docker pull 镜像:版本` |
| 看镜像 | `docker images` |
| 删镜像 | `docker rmi 镜像` |
| 跑容器 | `docker run -it --name 名字 镜像` |
| 后台跑 | `docker run -d -p 8080:80 镜像` |
| 看运行中 | `docker ps` |
| 看全部 | `docker ps -a` |
| 启动 | `docker start 名字` |
| 停止 | `docker stop 名字` |
| 进入 | `docker exec -it 名字 /bin/bash` |
| 日志 | `docker logs -f 名字` |
| 删容器 | `docker rm 名字` |

---

# 第四部分：镜像分层与 commit（23–25）

## 12. 镜像分层（理解了这个，后面全通）

**镜像不是一整块，而是"一层一层叠起来的"。**

```
┌─────────────────────────────┐
│ 层 4：你的程序（jar 包）      │  ← 每层只记录"变化"
├─────────────────────────────┤
│ 层 3：配置、环境变量          │
├─────────────────────────────┤
│ 层 2：JDK 安装                │
├─────────────────────────────┤
│ 层 1：基础系统（centos）      │  ← 所有镜像共享
└─────────────────────────────┘
```

**关键特性：**

| 特性 | 说明 |
|---|---|
| 只读 | 每一层都是只读的 |
| 共享 | 多个镜像可以共用底层（省空间、拉取快） |
| 写时复制 | 容器启动后，要改文件才复制一份到可写层 |

**为什么这个设计重要**：
- 你从 `ubuntu` 拉一个镜像，另一个镜像也用 `ubuntu` 底层 → 只拉一次
- 容器启动瞬间完成，因为不需要复制整个系统

## 13. commit：把容器保存成新镜像

**场景**：在 ubuntu 里装了 vim，想保存下来，下次直接用。

```bash
# 1. 跑一个 ubuntu 容器
docker run -it --name myubuntu ubuntu /bin/bash

# 2. 在容器里装 vim
apt-get update
apt-get install -y vim
exit

# 3. 把容器提交成新镜像
docker commit -m "装了vim的ubuntu" -a "你的名字" myubuntu myubuntu-vim:1.0

# 4. 用新镜像启动，vim 还在
docker run -it myubuntu-vim:1.0 /bin/bash
```

**commit 的缺点**：
- 镜像里会带上"操作历史"，越来越大
- 不可重复（换个人操作，结果可能不一样）
- **生产环境推荐用 Dockerfile 构建**（第 57 集开始讲）

**commit 适合**：临时实验、快速保存现场。

---

# 第五部分：镜像发布（26–28）

## 14. 发布到阿里云

```bash
# 1. 登录（阿里云控制台生成密码）
docker login --username=你的用户名 registry.cn-hangzhou.aliyuncs.com

# 2. 给镜像打标签（必须符合仓库路径格式）
docker tag myubuntu-vim:1.0 registry.cn-hangzhou.aliyuncs.com/你的命名空间/myubuntu-vim:1.0

# 3. 推送
docker push registry.cn-hangzhou.aliyuncs.com/你的命名空间/myubuntu-vim:1.0

# 4. 在别的机器上拉取
docker pull registry.cn-hangzhou.aliyuncs.com/你的命名空间/myubuntu-vim:1.0
```

> 🎬 **第 26 集**：阿里云控制台网页操作（创建命名空间、看仓库地址），**这部分必须看视频**，纯网页操作。

## 15. 私有库

**公司内部不想公开镜像**，自己搭一个 Registry：

```bash
# 1. 起一个 registry 容器
docker run -d -p 5000:5000 --name registry registry:2

# 2. 给镜像打标签（指向本地 registry）
docker tag myubuntu-vim:1.0 127.0.0.1:5000/myubuntu-vim:1.0

# 3. 推送
docker push 127.0.0.1:5000/myubuntu-vim:1.0

# 4. 拉取
docker pull 127.0.0.1:5000/myubuntu-vim:1.0
```

**坑**：Docker 默认不允许用 HTTP 推送非 localhost 的仓库，要加 `insecure-registries` 配置。

> 🎬 **第 27–28 集**：私有库演示，**看视频**（有配置文件的细节）。

---

# 第六部分：数据卷（29–33）

## 16. 为什么需要数据卷

**问题**：容器删了，里面写的数据也没了。

```bash
docker run -it --name test ubuntu /bin/bash
echo "重要数据" > /data.txt     # 容器里的文件
exit
docker rm test                   # 删容器
# /data.txt 没了！
```

**容器是"用完即弃"的，数据要持久化，就得挂到宿主机上。**

## 17. 数据卷的三种挂载方式

```bash
# 方式 1：指定宿主机目录（最常用）
docker run -it -v /宿主机/路径:/容器内/路径 ubuntu /bin/bash

# 方式 2：匿名卷（Docker 自动分配宿主机目录）
docker run -it -v /容器内/路径 ubuntu /bin/bash

# 方式 3：具名卷（推荐，方便管理）
docker run -it -v 卷名:/容器内/路径 ubuntu /bin/bash
```

**实际例子**：

```bash
# 把宿主机 ~/data 挂到容器的 /data
docker run -it -v ~/data:/data ubuntu /bin/bash

# 容器里建文件，宿主机也能看到
echo "hello" > /data/test.txt
# 宿主机 ~/data/test.txt 出现了
```

**这就是第 31 集说的"容器卷和主机互通"。**

## 18. ro / rw 读写规则

```bash
# 默认是 rw（读写）
docker run -it -v ~/data:/data:rw ubuntu /bin/bash

# ro 表示"容器内只读"，宿主机可以改，容器只能读
docker run -it -v ~/data:/data:ro ubuntu /bin/bash
```

**`ro` 适合**：配置文件、证书——容器只读不写，防止误改。

## 19. 容器卷继承（--volumes-from）

**场景**：多个容器要共享同一个数据卷。

```bash
# 容器 A 挂载了数据卷
docker run -it --name A -v ~/data:/data ubuntu /bin/bash

# 容器 B 继承 A 的挂载
docker run -it --name B --volumes-from A ubuntu /bin/bash

# 现在 A 和 B 看到的是同一个 /data
```

**用途**：多容器共享配置、日志、数据。

**注意**：`--volumes-from` 是"继承挂载配置"，**不是拷贝数据**——改一处，两边都变。

---

# 第七部分：网络（66–77）

## 20. 五种网络模式

| 模式 | 命令 | 特点 |
|---|---|---|
| **bridge** | 默认 | 每个容器有独立 IP，通过 docker0 网桥通信 |
| **host** | `--network host` | 容器直接用宿主机网络，端口不隔离 |
| **none** | `--network none` | 没有网络，只有 lo |
| **container** | `--network container:名字` | 共用另一个容器的网络 |
| **自定义** | `--network 网络名` | 自己建的 bridge，**推荐** |

## 21. bridge 模式（默认）

```
宿主机
  ├─ docker0（虚拟网桥，默认 172.17.0.1）
  │    ├─ 容器 A（172.17.0.2）
  │    ├─ 容器 B（172.17.0.3）
  │    └─ 容器 C（172.17.0.4）
  └─ 网卡 eth0（真实网络）
```

**特点**：
- 每个容器分配一个 172.17.x.x 的 IP
- 容器之间可以通过 IP 互通
- **容器之间不能用名字互相访问**（这是默认 bridge 的缺陷）

**解决**：自定义网络（下一节）。

## 22. 自定义网络（推荐）

```bash
# 1. 创建一个自定义网络
docker network create mynet

# 2. 启动容器时指定网络
docker run -d --name nginx1 --network mynet nginx
docker run -d --name nginx2 --network mynet nginx

# 3. 两个容器可以直接用名字互访
docker exec -it nginx1 ping nginx2    # ✅ 通
```

**自定义网络的好处**：
- **自动 DNS**：容器间可以用名字访问
- 与默认 bridge 隔离，互不干扰
- 适合多容器项目（Compose 默认就用自定义网络）

## 23. 网络命令

```bash
docker network ls                      # 列出所有网络
docker network inspect 网络名          # 看详情
docker network create 网络名           # 创建
docker network rm 网络名               # 删除
docker network connect 网络 容器       # 把容器连到网络
docker network disconnect 网络 容器    # 断开
```

> 🎬 **第 75–77 集**：自定义网络的完整演示（创建、跨网络测试），**看视频**更直观。

---

# 第八部分：Dockerfile（57–65）

## 24. 是什么

**Dockerfile 就是一个"构建镜像的脚本"**，一行一个指令，把"装环境"的过程写成代码。

**和 commit 的区别**：
- `commit`：手动操作，不可重复
- `Dockerfile`：**代码化、可重复、可版本管理**

## 25. 一个完整的例子

```dockerfile
# 基础镜像
FROM openjdk:17-jdk-slim

# 维护者信息（已废弃，用 LABEL 替代）
LABEL maintainer="you@example.com"

# 设置工作目录
WORKDIR /app

# 复制 jar 包进去
COPY app.jar /app/app.jar

# 声明暴露的端口（只是声明，不真的映射）
EXPOSE 8080

# 容器启动时执行的命令
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

**构建**：

```bash
docker build -t myapp:1.0 .
```

**最后的 `.` 是构建上下文**（把当前目录发给 Docker 引擎）。

## 26. 常用指令

| 指令 | 作用 | 例子 |
|---|---|---|
| `FROM` | 基础镜像（必须第一行） | `FROM ubuntu:22.04` |
| `RUN` | **构建时**执行命令 | `RUN apt-get update` |
| `CMD` | **容器启动时**的默认命令 | `CMD ["java","-jar","app.jar"]` |
| `ENTRYPOINT` | 容器启动时的入口（可追加参数） | `ENTRYPOINT ["java","-jar"]` |
| `COPY` | 复制文件（推荐） | `COPY app.jar /app/` |
| `ADD` | 复制 + 自动解压 tar | `ADD app.tar.gz /app/` |
| `WORKDIR` | 工作目录 | `WORKDIR /app` |
| `EXPOSE` | 声明端口 | `EXPOSE 8080` |
| `ENV` | 环境变量 | `ENV JAVA_HOME=/opt/jdk` |
| `VOLUME` | 声明数据卷 | `VOLUME ["/data"]` |
| `USER` | 指定运行用户 | `USER appuser` |
| `ARG` | 构建参数 | `ARG VERSION=1.0` |

**CMD vs ENTRYPOINT 的区别**（面试常问）：

```dockerfile
# CMD：可以被 docker run 后面的参数覆盖
CMD ["java", "-jar", "app.jar"]
docker run myapp sh        # 实际执行 sh，不是 java

# ENTRYPOINT：固定入口，docker run 后面的参数是"追加参数"
ENTRYPOINT ["java", "-jar", "app.jar"]
docker run myapp --debug   # 实际执行 java -jar app.jar --debug
```

## 27. 构建过程（分层）

**Dockerfile 的每条指令都会生成一层镜像。**

```dockerfile
FROM ubuntu:22.04        # 层 1
RUN apt-get update       # 层 2
RUN apt-get install vim  # 层 3
COPY app.jar /app/       # 层 4
```

**优化技巧**：
- 把"不常变的"放前面，"常变的"放后面（利用缓存）
- 多个 `RUN` 合并成一行（减少层数）

```dockerfile
# 优化前：3 层
RUN apt-get update
RUN apt-get install -y vim
RUN apt-get install -y curl

# 优化后：1 层
RUN apt-get update && apt-get install -y vim curl
```

## 28. 虚悬镜像

**没有标签（`<none>`）的镜像**，通常是重新构建同一个 tag 后留下的旧版本。

```bash
# 查看
docker images -f dangling=true

# 清理
docker image prune
```

> 🎬 **第 60–61 集**：Dockerfile 实战演示（从编写到构建到运行），**看视频**跟一遍。
> 🎬 **第 63–65 集**：微服务打包 → 生成 jar → 构建镜像 → 部署，**看视频**。

---

# 第九部分：Compose（78–86）

## 29. 是什么

**Compose 用来"一次性管理多个容器"。**

**问题**：一个项目要起 mysql + redis + 你的应用，得敲三条 `docker run`，还要记住每个的参数。

**Compose 的答案**：写一个 `docker-compose.yml`，一条命令全起来。

```bash
docker compose up -d      # 全部启动
docker compose down       # 全部停止并删除
```

## 30. 核心概念

| 概念 | 对应 |
|---|---|
| **服务 service** | 一个容器 |
| **项目 project** | 一组服务（一个 yml 文件 = 一个项目） |
| **网络 network** | Compose 自动创建一个默认网络 |
| **卷 volume** | 数据持久化 |

## 31. yml 结构

```yaml
version: '3.8'          # 版本（新版本可省略）

services:               # 服务列表
  mysql:                # 服务名（也是容器名前缀）
    image: mysql:8.0
    container_name: my-mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: testdb
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - mynet

  redis:
    image: redis:7
    container_name: my-redis
    ports:
      - "6379:6379"
    networks:
      - mynet

  app:
    build: .              # 用当前目录的 Dockerfile 构建
    container_name: my-app
    ports:
      - "8080:8080"
    depends_on:
      - mysql
      - redis
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/testdb
    networks:
      - mynet

volumes:
  mysql-data:

networks:
  mynet:
    driver: bridge
```

**关键点**：
- **服务名可以直接当主机名**（`jdbc:mysql://mysql:3306` 里的 `mysql` 就是服务名）
- `depends_on` 只保证启动顺序，**不保证服务就绪**
- 数据卷要用"具名卷"才能在 `down` 后保留

## 32. 常用命令

```bash
docker compose up -d          # 后台启动
docker compose up --build     # 重新构建后启动
docker compose down           # 停止并删除容器/网络
docker compose down -v        # 连数据卷一起删（危险）
docker compose ps             # 查看状态
docker compose logs -f        # 跟踪日志
docker compose restart        # 重启
docker compose config         # 校验并展示配置
```

**注意**：新版本是 `docker compose`（空格），老版本是 `docker-compose`（横线）。

> 🎬 **第 82–86 集**：不用 Compose vs 用 Compose 的对比演示，**看视频**能直观感受到方便在哪。

---

# 第十部分：进阶实战（34–56）

## 33. 装常用软件（命令照抄）

### MySQL

```bash
docker run -d --name mysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -v ~/mysql/data:/var/lib/mysql \
  mysql:8.0
```

### Redis

```bash
docker run -d --name redis \
  -p 6379:6379 \
  -v ~/redis/data:/data \
  redis:7 redis-server --appendonly yes
```

### Tomcat

```bash
docker run -d --name tomcat \
  -p 8080:8080 \
  -v ~/tomcat/webapps:/usr/local/tomcat/webapps \
  tomcat:9
```

**要点**：
- 数据一定要挂出来（`-v`），否则删容器数据就没了
- 密码用 `-e` 传，别写死在镜像里

## 34. 主从复制（41 集）

**MySQL 主从 = 主库写，从库读，主库的数据同步到从库。**

```bash
# 主库
docker run -d --name mysql-master \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root \
  mysql:8.0

# 从库
docker run -d --name mysql-slave \
  -p 3307:3306 \
  -e MYSQL_ROOT_PASSWORD=root \
  mysql:8.0
```

**核心配置**：
- 主库开 `log-bin`（二进制日志）
- 从库配置 `server-id` + `relay-log`
- 从库执行 `CHANGE MASTER TO` 指向主库

**原理**：
```
主库写 binlog → 从库 IO 线程拉取 → 写入 relay log → SQL 线程重放
```

## 35. 分布式存储三种算法（42–44）

| 算法 | 原理 | 问题 |
|---|---|---|
| **哈希取余** | `hash(key) % 节点数` | **节点增减时大量数据要迁移** |
| **一致性哈希** | 哈希环，key 找顺时针第一个节点 | 节点少时分布不均（数据倾斜） |
| **哈希槽** | 预分 16384 个槽，槽分配给节点 | Redis Cluster 用的方案，最均衡 |

**记忆**：
- 取余：简单但扩缩容痛
- 一致性哈希：扩缩容好，但可能倾斜
- 哈希槽：**Redis 集群的选择**，兼顾两者

## 36. Redis 集群（45–56）

**3 主 3 从**：6 个节点，数据分 16384 个槽，每个主节点管一部分槽，从节点做备份。

```bash
# 创建集群（简化版）
redis-cli --cluster create \
  172.17.0.2:6379 172.17.0.3:6379 172.17.0.4:6379 \
  172.17.0.5:6379 172.17.0.6:6379 172.17.0.7:6379 \
  --cluster-replicas 1
```

**常见错误**（48 集）：
- `CLUSTERDOWN`：集群不可用（槽没分配完）
- `MOVED`：key 不在当前节点，要重定向
- `CROSSSLOT`：一次操作涉及多个槽

**扩缩容**（52–55 集）：
- 扩容：加节点 → 迁移槽 → 完成
- 缩容：迁移槽走 → 删节点

> 🎬 **第 45–47、51–55 集**：集群配置和扩缩容都是**多步演示**，**强烈建议看视频**，文字步骤容易漏。

---

# 第十一部分：图形化工具（87–94）—— 必须看视频

## 37. Portainer

**Web 界面的 Docker 管理工具**：看容器、镜像、网络、卷，点鼠标就能操作。

```bash
docker run -d -p 9000:9000 \
  --name portainer \
  --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```

访问 `http://localhost:9000` 设置管理员密码。

> 🎬 **第 87–89 集**：Portainer 安装和**界面操作**，**必须看视频**——纯 GUI，文字描述不了。

## 38. CIG（CAdvisor + InfluxDB + Grafana）

**容器监控三件套**：

```
CAdvisor（采集数据）→ InfluxDB（存数据）→ Grafana（画图）
```

```yaml
# docker-compose.yml
services:
  influxdb:
    image: influxdb:1.8
    ports:
      - "8086:8086"
    environment:
      - INFLUXDB_DB=cadvisor
      - INFLUXDB_HTTP_AUTH_ENABLED=false

  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
      - influxdb

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    depends_on:
      - influxdb
```

> 🎬 **第 90–94 集**：CIG 平台搭建、**Grafana 界面配置**、加 panel、配监控规则，**全部必须看视频**——Grafana 全是鼠标点点点。

---

# 附录：看视频清单（按优先级）

## 🔴 必看（文字替代不了）

| 集数 | 内容 | 原因 |
|---|---|---|
| 09–10 | 平台入门、架构图解 | 动画图解 |
| 26 | 发布到阿里云 | 网页操作 |
| 87–89 | Portainer | 全 GUI |
| 90–94 | CIG + Grafana | 全 GUI |

## 🟡 建议看（步骤长/演示类）

| 集数 | 内容 |
|---|---|
| 17 | ubuntu 容器演示 |
| 27–28 | 私有库推送演示 |
| 45–47 | Redis 集群配置 |
| 51–55 | 主从容错、扩缩容 |
| 60–61 | Dockerfile 案例 |
| 63–65 | 微服务部署 |
| 75–77 | 自定义网络 |
| 82–86 | Compose 编排对比 |

## 🟢 文字够用（可跳过视频）

| 集数 | 内容 |
|---|---|
| 01–08 | 概念、三要素 |
| 11–12 | 安装、加速器 |
| 13–16 | hello-world、镜像命令 |
| 18–22 | 容器命令 |
| 23–25 | 分层、commit |
| 29–33 | 数据卷 |
| 34–39 | 装软件 |
| 41–44 | 主从原理、存储算法 |
| 48–50 | 集群错误、路由 |
| 57–59 | Dockerfile 指令 |
| 66–74 | 网络模式 |

---

# 附：一页速查

## 镜像

```bash
docker pull nginx:1.25        # 拉
docker images                 # 看
docker rmi nginx              # 删
docker build -t app:1.0 .     # 构建
docker save -o a.tar nginx    # 导出
docker load -i a.tar          # 导入
```

## 容器

```bash
docker run -it --name c ubuntu /bin/bash   # 交互式
docker run -d -p 8080:80 nginx             # 后台
docker ps -a                               # 全部
docker start/stop/rm 名字                   # 启停删
docker exec -it 名字 /bin/bash             # 进入
docker logs -f 名字                        # 日志
docker cp 名字:/a /b                       # 拷贝
```

## 数据卷

```bash
-v ~/data:/data              # 挂目录
-v ~/data:/data:ro           # 只读
-v 卷名:/data                # 具名卷
--volumes-from 别的容器       # 继承
```

## 网络

```bash
docker network ls
docker network create mynet
docker run --network mynet ...
```

## Compose

```bash
docker compose up -d
docker compose down
docker compose ps
docker compose logs -f
```

---

## 记忆口诀

> **镜像模板容器跑，仓库存镜像；**
> **-it 进容器，-d 后台跑，-p 映射端口；**
> **exec 进容器，attach 会送命；**
> **数据要持久化，-v 挂到宿主机；**
> **默认 bridge 不能互 ping，自定义网络走名字；**
> **Dockerfile 构建镜像，Compose 编排多容器；**
> **Portainer/CIG 是 GUI，必须看视频。**
