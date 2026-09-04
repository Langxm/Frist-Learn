# dockers知识整理

本文整理 Docker 入门知识。软件的正式名称是 **Docker**，命令写作 `docker`。学习环境为 Windows + WSL 2 + Ubuntu，使用 Docker Desktop 管理 Linux 容器。

## 学习资料

- [Docker 菜鸟教程（中文）](https://www.runoob.com/docker/docker-tutorial.html)
- [Docker 官方入门](https://docs.docker.com/get-started/)
- [Docker Desktop 与 WSL 2 集成](https://docs.docker.com/desktop/features/wsl/)
- [Docker 官方：什么是容器](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/)
- [Docker 官方：什么是镜像](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/)

菜鸟教程用于中文入门；安装步骤、版本兼容性和命令行为以对应版本的官方文档为准。

## 1. Linux、Ubuntu、WSL 2 与 Docker

| 名称 | 含义 | 在当前环境中的作用 |
| --- | --- | --- |
| Linux | 严格说是内核；日常也用来统称 Linux 操作系统 | 管理进程、内存和设备等资源 |
| Ubuntu | Linux 发行版 | 提供终端、系统工具、软件包管理和开发环境 |
| WSL 2 | Windows 中运行 Linux 环境的平台，使用真实 Linux 内核 | 让 Windows 和 Ubuntu 开发环境共存 |
| Docker Desktop | 桌面应用，集成 Docker 引擎、客户端和管理界面等 | 管理镜像、容器、网络和存储，并提供 WSL 集成 |
| Docker Engine | 执行容器管理工作的后台引擎 | 根据客户端请求下载镜像、创建和运行容器 |
| ROS 2 / Gazebo | 机器人软件框架 / 仿真软件 | 可以安装在 Ubuntu 或合适的容器环境里 |

当前使用 **Docker Desktop 的 WSL 2 后端与 Ubuntu 集成**。在 PowerShell 和 Ubuntu 里都可以调用 Docker 客户端，连接同一个 Docker Desktop 引擎。

```text
PowerShell中的Docker客户端 ─┐
                          ├─ Docker Desktop管理的Linux引擎
Ubuntu中的Docker客户端 ────┘       （WSL 2后端）
                                      ↓
                                  镜像和容器
```

这与“在 Ubuntu 中另外安装一个独立 Docker Engine”是不同的安装方式。当前无需再重复安装一套引擎。

Linux 容器共享运行它们的 Linux 内核，但拥有隔离的运行环境。容器中的 Ubuntu 软件版本可以不同于 WSL 发行版；这不代表容器启动了自己的独立内核。图形显示、GPU、设备和网络仍需要按用途配置。

## 2. 四个核心概念

| 概念 | 理解方式 | 示例 |
| --- | --- | --- |
| 镜像 Image | 用于创建容器的只读模板，包含程序、文件和依赖 | `hello-world:latest` |
| 容器 Container | 根据镜像创建的实例，可以运行、停止或删除 | 执行 hello-world 程序的容器 |
| 客户端 Client | 接收命令，并向引擎发送请求 | 终端中的 `docker` 命令 |
| 服务端 Server / Engine | 真正管理和执行容器操作 | Docker Desktop 中运行的引擎 |

补充概念：

- **Registry**：分发镜像的服务，例如 Docker Hub。
- **Tag**：镜像标签，例如 `latest`。它是可变标签，不保证永远代表最新版本。
- **Dockerfile**：描述镜像构建步骤的文本文件，方便复现环境。
- **Volume（数据卷）**：由 Docker 管理的持久数据存储。
- **Bind mount（绑定挂载）**：把宿主环境中的指定路径挂载到容器中，适合保存和编辑代码。

同一个镜像可以创建多个容器。容器停止后仍可能存在；“停止容器”和“删除容器”是两件事。

## 3. 第一次运行：hello-world

在 Windows PowerShell 中执行：

```powershell
docker run --rm hello-world
```

| 命令片段 | 含义 |
| --- | --- |
| `docker` | 调用 Docker 客户端 |
| `run` | 创建并启动一个新容器 |
| `--rm` | 容器退出后自动删除该容器 |
| `hello-world` | 镜像名称，未指定标签时默认使用 `latest` |

第一次执行时的过程：

```text
本机没有镜像 → 从Docker Hub下载 → 创建容器
           → 执行测试程序 → 输出文字 → 退出并删除测试容器
```

常见输出：

| 输出 | 含义 |
| --- | --- |
| `Unable to find image ... locally` | 本地尚无该镜像，随后会尝试拉取；单独这一行不表示操作失败 |
| `Pull complete` | 对应镜像层拉取完成 |
| `Downloaded newer image` | 镜像已下载 |
| `Hello from Docker!` | 容器中的测试程序成功执行 |

`--rm` 不会删除下载好的镜像。hello-world 打印文字后结束，因此不会一直出现在运行中的容器列表里。

这项测试证明镜像下载和基本容器运行成功，尚不能证明 ROS 2、Gazebo 或 GPU 已经可用。

## 4. 验证 WSL 与 Docker 集成

在 **Windows PowerShell** 中执行：

```powershell
wsl -d Ubuntu -- docker version
```

- `wsl`：从 Windows 调用 WSL。
- `-d Ubuntu`：选择名称为 Ubuntu 的发行版。
- `--`：后面的内容作为要执行的 Linux 命令。
- `docker version`：在 Ubuntu 里查询 Docker 客户端和服务端版本。

这条命令执行结束后会回到 PowerShell。如果已经进入 Ubuntu 终端，直接执行 `docker version` 即可。

| 输出字段 | 含义 |
| --- | --- |
| `Client` | 客户端信息；仅有它不足以证明已连接引擎 |
| `Server` | 服务端信息；能正常返回说明客户端已经连接引擎 |
| `API version` | 客户端与服务端通信使用的接口版本 |
| `OS/Arch: linux/amd64` | Linux 环境、x86-64 架构；amd64 不限定 AMD 品牌 |
| `containerd` / `runc` | 底层容器管理与执行组件，入门阶段先了解名称即可 |

若出现“localhost 代理配置未镜像到 WSL”的提示，但 Server 信息正常返回，说明这次 Docker 连接成功。该结果不能证明 Ubuntu 中所有程序都能正常联网。

## 5. 常用命令速查

以下是后续学习参考，并非都已执行。含 `<容器名>` 的命令需要替换占位符。

| 命令 | 作用 |
| --- | --- |
| `docker version` | 查看客户端和服务端版本 |
| `docker info` | 查看引擎信息 |
| `docker image ls` | 查看本地镜像 |
| `docker ps` | 查看运行中的容器 |
| `docker ps -a` | 查看包括已停止容器在内的容器列表 |
| `docker pull hello-world` | 只下载镜像，不创建容器 |
| `docker logs <容器名>` | 查看容器输出的日志 |
| `docker stop <容器名>` | 停止容器 |
| `docker start <容器名>` | 重新启动已有容器 |
| `docker exec -it <容器名> sh` | 在运行中的容器里启动终端，镜像需要包含 sh |

`docker run` 创建新容器；`docker start` 启动已有容器；`docker exec` 在正在运行的容器里执行另一条命令。

## 6. 文件为什么要持久保存

在容器里安装软件或修改文件，通常会写入该容器的可写层：

- 停止并重新启动同一个容器，修改通常还在。
- 删除容器，其可写层会被删除。
- 根据原镜像重新创建容器，不会自动带上旧容器的修改。
- 代码可以通过绑定挂载保存；其他持久数据可以使用数据卷。
- 数据卷有独立生命周期，但仍可能被明确删除。
- 可复现的软件安装过程应写进 Dockerfile，不能只依赖手工修改某一个容器。

## 7. 当前进度与后续顺序

已完成：

- [x] 确认使用 WSL 2 与 Ubuntu 26.04。
- [x] 安装并启动 Docker Desktop。
- [x] 成功下载并运行 hello-world。
- [x] 从 Ubuntu 获取 Docker Client 和 Server 信息。

待学习：

- [ ] 区分镜像、容器、停止与删除。
- [ ] 创建学习容器并配置代码持久保存。
- [ ] 学习 Dockerfile 和镜像构建。
- [ ] 配置并验证 ROS 2 通信。
- [ ] 验证图形显示，再运行 RViz 与 Gazebo。

这里记录的是 Docker 入门进度。ROS 2 镜像下载、学习容器创建和仿真运行尚未作为已完成事项记录。
