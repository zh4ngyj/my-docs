## Windows配置WSL2

通过适用于Linux的Windows子系统（WSL2），可以直接在Windows上运行你最喜爱的Linux工具、实用工具、应用程
序和工作流。

### 介绍

#### 1.什么是 WSL2?

**WSL2（Windows Subsystem for Linux 2）** 是微软推出的**第二代 Windows Linux 子系统**，是一个能让 Windows 10/11 直接运行完整 Linux 二进制程序的兼容层，核心架构是**基于 Hyper-V 的轻量级虚拟机**。

它和 WSL1 的本质区别在于：

- WSL1 没有真正的 Linux 内核，靠**系统调用翻译层**将 Linux 的系统调用转换成 Windows 的系统调用，属于 “模拟” 运行；
- WSL2 内置了**完整的 Linux 内核**（由微软编译维护，与主线 Linux 内核同步更新），通过轻量级 Hyper-V 虚拟机实现，既保留了 Linux 的原生功能，又能和 Windows 系统深度集成（比如文件互通、网络共享）。

WSL2 的核心特点：

1. **完整 Linux 兼容性**：支持所有 Linux 系统调用、内核模块（如 Docker 需要的内核特性）、原生 Linux 工具链；
2. **高性能**：文件 IO 速度比 WSL1 提升数倍，接近原生 Linux 水平；
3. **硬件直通支持**：支持 GPU（NVIDIA CUDA/AMD ROCm）、USB 设备直通，适合 AI 模型训练 / 推理、硬件开发；
4. **无缝集成 Windows**：Windows 和 WSL2 可以互相访问文件系统，共享网络端口，直接调用对方的程序（比如在 WSL2 里运行`notepad.exe`打开 Windows 记事本）。



#### 2.为什么要安装WSL2，而不安装WSL1?

WSL2 在**性能、功能兼容性、硬件支持**上全面碾压 WSL1，核心优势如下：

| 对比维度           | WSL1                                                         | WSL2                                                         |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **内核架构**       | 无真实内核，系统调用翻译层                                   | 完整 Linux 内核，轻量级 Hyper-V 虚拟机                       |
| **文件 IO 性能**   | 慢（尤其是对 Linux 文件系统的随机读写）                      | 快（接近原生 Linux，大文件操作无压力）                       |
| **Docker 支持**    | 仅支持 Docker Desktop 的 “WSL1 后端”，功能受限，性能差       | 原生支持 Docker Desktop 的 “WSL2 后端”，容器启动快，资源占用低，是目前 Docker 在 Windows 的最佳实践 |
| **GPU / 硬件直通** | 不支持 CUDA/ROCm，无法用于 AI 模型推理训练                   | 支持 NVIDIA CUDA、AMD ROCm、DirectML，可直接调用你的 RTX 4070 显卡进行模型量化、推理 |
| **系统调用完整性** | 仅支持部分 Linux 系统调用，部分工具（如`iptables`、`systemd`）无法使用 | 支持全部 Linux 系统调用，可运行`systemd`、`kubelet`等复杂服务 |
| **跨系统交互**     | 支持，但文件 IO 延迟高                                       | 支持，且文件互通速度更快，网络端口共享更稳定                 |



**补充：WSL2 的少量局限性及解决办法**

1. **依赖 Hyper-V**：需要开启 Windows 的 Hyper-V 功能（部分老旧 CPU 不支持），但你的 Windows 11+RTX 4070 平台完全满足；
2. **内存占用**：默认会动态分配内存，可通过`%USERPROFILE%\.wslconfig`配置最大内存上限，避免占用过多 Windows 资源。

### 安装步骤

#### 1.前置条件检查

~~~powershell
1. 系统版本要求
- Windows 10：2004 版本（内部版本 19041）及以上
- Windows 11：任意版本
- 查看版本：按 Win+R 输入 winver 即可查看

2. 启用 CPU 虚拟化（必须）
	1.按 Ctrl+Shift+Esc 打开任务管理器 → 性能 → CPU
	2.查看 “虚拟化” 是否为 已启用
	3.若未启用：重启电脑，按 Del/F2/F10 进入 BIOS，找到 Intel VT-x/AMD-V 选项，设为 Enabled，保存重启
~~~



#### 2.启用 WSL 和虚拟机平台功能

~~~powershell
# 以管理员身份打开 PowerShell（Win+X → 选择 “Windows PowerShell (管理员)”）PowerShell (管理员)”
# 启用适用于 Linux 的 Windows 子系统
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 启用虚拟机平台（WSL2 必需，缺一不可）
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 重启电脑
~~~



#### 3.安装 WSL2

~~~powershell
# 执行命令将 WSL2 设为默认版本
wsl --set-default-version 2
~~~



#### 4.安装Linux发行版

这里提供一种简单方式：通过去微软商店安装，以Ubuntu22.04为例

![store-install](store-install.png)



#### 5.更新和升级软件包

建议使用发行版的首选包管理器定期更新和升级包。 对于 Ubuntu 或 Debian，请使用以下命令：

~~~bash
sudo apt update && sudo apt upgrade
~~~



#### 其他可选项

1.系统默认安装在C盘，可以通过命令迁移到其他盘，释放C盘空间

~~~powershell
# 关闭所有 WSL 实例
wsl --shutdown

# 导出系统为备份文件
wsl --export 原始名称 D:\wsl-backup\my-distro.tar

# 注销原系统
wsl --unregister 原始名称

用新名称重新导入系统
wsl --import 新名称 D:\wsl\新名称 D:\wsl-backup\my-distro.tar

# 直接以 root 身份登录 新名称
wsl -d 新名称 -u root
~~~



2.创建用户

~~~powershell
# 直接以 root 身份登录
wsl -d  -u root

# 检查 : 在 root 终端中执行以下命令，检查 {username} 是否存在
id {username}

# 如果输出 no such user,继续执行下面的创建流程
# 创建用户（-m 自动创建用户主目录，-s 指定默认 shell 为 bash）
useradd -m -s /bin/bash {username}

# 设置用户密码（执行后会提示输入两次密码，建议记好）
passwd {username}

# （可选）给用户添加 sudo 权限（否则无法执行 sudo 命令）
usermod -aG sudo {username}

# 执行完后，再用 id zh4ngyj 验证，能看到 UID（通常是 1000）、GID 信息，说明用户创建成功。
~~~



3.若要添加其他 Linux 发行版，可以通过 [Microsoft Store](ms-windows-store://collection?CollectionId=LinuxDistros)、通过 [--import 命令](https://learn.microsoft.com/zh-cn/windows/wsl/use-custom-distro)或通过[旁加载你自己的自定义发行版](https://learn.microsoft.com/zh-cn/windows/wsl/build-custom-distro)进行安装。 你可能还希望设置自定义 WSL 映像，以便在企业公司中分发。



### 教程

#### VS Code

若要从 WSL 分发版打开项目，请打开分发的命令行并输入： `code .`



#### C/C++环境搭建

1.假设你的发行版使用 `apt`（本演练使用 Ubuntu），此时请使用以下命令在 WSL 2 发行版上安装所需的生成工具：

~~~bash
sudo apt update
sudo apt install g++ gdb make ninja-build rsync zip
~~~

上述 `apt` 命令会安装：

- C++ 编译器
- `gdb`
- `CMake`
- `rsync`
- `zip`
- 基础生成系统生成器

2....



#### 设置 GPU 加速(NVIDIA CUDA/DirectML)

https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/gpu-compute



#### 接入 Docker 远程容器

1. 下载 [Docker Desktop](https://docs.docker.com/desktop/features/wsl/#turn-on-docker-desktop-wsl-2) 并按照安装说明进行作。
2. 安装后，从 Windows“开始”菜单启动 Docker Desktop，然后从任务栏的隐藏图标菜单中选择 Docker 图标。 右键单击图标以显示 Docker 命令菜单，然后选择“设置”。
3. 确保在**“设置**>”中选中“使用基于 WSL 2 的引擎”。
4. 通过转到 **“设置**>**资源**>**WSL 集成**”，从要启用 Docker 集成的已安装 WSL 2 分发版中进行选择。
5. 要确认 Docker 是否已安装，请打开 WSL 发行版（如 Ubuntu），并输入以下命令以显示版本及内部版本号：`docker --version`
6. 使用以下方法测试安装是否正常工作：运行简单的内置 Docker 映像： `docker run hello-world`



### 常见指令

**核心基础指令**：`wsl -l -v`（查系统）、`wsl -d <系统名>`（启动指定系统）、`wsl --shutdown`（关闭所有 WSL）是日常使用频率最高的三个指令；

**关键参数记忆**：`-d`指定系统、`-u`指定用户、`-v`查看详情、`--set-default`设置默认系统；

**操作注意**：修改 WSL 配置（重命名、迁移、卸载）前，务必先执行`wsl --shutdown`关闭所有运行的 WSL 实例，避免数据损坏。

#### 1.基础信息查看类

这类指令用于查看 WSL2 的版本、已安装系统、运行状态等核心信息。

| 指令                                  | 说明                                      | 示例 / 补充                                                  |
| ------------------------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| `wsl --version` 或 `wsl -v`           | 查看 WSL 内核版本、WSL 版本（1/2）        | 可确认是否安装 WSL2 及内核是否最新                           |
| `wsl --list --verbose` 或 `wsl -l -v` | 查看所有已安装的 WSL 系统（含状态、版本） | 输出列：`NAME`（系统名）、`STATE`（运行 / 停止）、`VERSION`（1/2） |
| `wsl --list --online` 或 `wsl -l -o`  | 查看微软官方可安装的 WSL 发行版           | 比如 Ubuntu、Debian、SUSE 等                                 |
| `wsl --status`                        | 查看 WSL 的整体状态（默认版本、内核等）   | 快速确认 WSL2 是否为默认版本                                 |

示例:

~~~powershell
# 查看已安装的WSL系统详情
wsl -l -v
# 输出示例：
#  NAME                   STATE           VERSION
#* Ubuntu2204             Running         2
#  Debian                 Stopped         2
~~~



#### 2.系统启停 / 管理类

这类指令用于启动、关闭、切换 WSL2 系统，是最频繁使用的操作。

| 指令                                             | 说明                                  | 示例 / 补充                                            |
| ------------------------------------------------ | ------------------------------------- | ------------------------------------------------------ |
| wsl` 或 `wsl.exe                                 | 启动默认 WSL 系统（进入交互终端）     | 直接回车即可，默认以设置的默认用户登录                 |
| wsl -d <系统名>` 或 `wsl --distribution <系统名> | 启动指定名称的 WSL 系统               | `wsl -d Ubuntu2204`（启动 Ubuntu2204）                 |
| wsl -u <用户名>` 或 `wsl --user <用户名>         | 以指定用户启动 WSL                    | `wsl -d Ubuntu2204 -u root`（以 root 登录 Ubuntu2204） |
| wsl --shutdown                                   | 关闭所有运行中的 WSL 实例（强制停止） | 重命名、修改配置前必须执行，避免数据损坏               |
| wsl --terminate <系统名>` 或 `wsl -t <系统名>    | 仅关闭指定的 WSL 系统                 | `wsl -t Ubuntu2204`（只停止 Ubuntu2204，不影响其他）   |
| wsl --set-default <系统名>` 或 `wsl -s <系统名>  | 设置默认启动的 WSL 系统               | `wsl -s Ubuntu2204`（后续输入`wsl`直接启动该系统）     |

#### 3.安装 / 卸载 / 迁移类

这类指令用于安装新系统、卸载无用系统，或迁移 WSL2 系统到其他磁盘（解决 C 盘占用）

| 指令                                          | 说明                                     | 示例 / 补充                                                  |
| --------------------------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| wsl --install                                 | 一键安装 WSL2（默认 Ubuntu）             | 需管理员权限，首次安装会自动启用虚拟机平台、下载内核         |
| wsl --install -d <系统名>                     | 安装指定的 WSL 发行版                    | `wsl --install -d Debian`（安装 Debian）                     |
| wsl --unregister <系统名>                     | 卸载 / 注销指定 WSL 系统（删除所有数据） | 谨慎使用！会彻底删除该系统的所有文件，建议先导出备份         |
| wsl --export <系统名> <备份路径.tar>          | 导出 WSL 系统为备份文件（.tar 格式）     | wsl --export Ubuntu2204 D:\wsl-backup\ubuntu.tar             |
| wsl --import <新系统名> <安装路径> <备份.tar> | 导入备份文件为新的 WSL 系统              | wsl --import Ubuntu2204 D:\wsl\Ubuntu2204 D:\wsl-backup\ubuntu.tar |

**迁移系统示例**（解决 C 盘占用）：

~~~
# 1. 关闭所有WSL
wsl --shutdown

# 2. 导出原系统到D盘
wsl --export Ubuntu2204 D:\wsl-backup\ubuntu2204.tar

# 3. 注销C盘的原系统
wsl --unregister Ubuntu2204

# 4. 导入到D盘并命名为Ubuntu2204
wsl --import Ubuntu2204 D:\wsl\Ubuntu2204 D:\wsl-backup\ubuntu2204.tar

# 5. 设置默认用户（导入后默认root）
wsl -d Ubuntu2204 -u zh4ngyj
~~~



#### 4.版本切换 / 配置类

这类指令用于设置 WSL 默认版本（1/2）、升级系统版本等。

| 指令                             | 说明                                    | 示例 / 补充                                                  |
| -------------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| wsl --set-default-version <1/2>  | 设置新建 WSL 系统的默认版本             | `wsl --set-default-version 2`（推荐，所有新安装的系统默认 WSL2） |
| wsl --set-version <系统名> <1/2> | 将已安装的 WSL 系统从 1 升级 / 降级到 2 | `wsl --set-version Ubuntu2204 2`（把 Ubuntu2204 切换为 WSL2） |
| wsl --update                     | 更新 WSL 内核和 WSL 组件                | 解决 WSL2 兼容性问题的首选，需管理员权限                     |
| `wsl --update --rollback`        | 回滚 WSL 到上一个版本                   | 若更新后出现问题，可回滚                                     |

#### 5.文件操作 / 跨系统交互类

WSL2 支持 Windows 和 Linux 文件互通，这类指令用于快速访问文件。

| 指令                                     | 说明                                                         | 示例 / 补充                                                  |
| ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| wsl ls /mnt/c/Users/<你的Windows用户名>/ | 在 WSL 中访问 Windows 的 C 盘用户目录                        | `/mnt/盘符/` 是 WSL 访问 Windows 磁盘的固定路径（C 盘 =/mnt/c，D 盘 =/mnt/d） |
| explorer.exe .                           | 在 WSL 终端中打开当前目录的 Windows 文件资源管理器           | WSL 中输入该指令，直接弹出对应路径的文件夹，无需手动找路径   |
| wsl <Linux指令>                          | 在 Windows 终端中直接执行 Linux 指令（无需进入 WSL 交互模式） | `wsl ls -l /home`（查看 WSL 的 home 目录文件）、`wsl mkdir /mnt/d/wsl-test`（在 D 盘创建文件夹） |

#### 6.高级 / 排障类

用于排查 WSL2 故障、重置配置等（新手少用，需谨慎）。

| 指令              | 说明                                            | 示例 / 补充                                 |
| ----------------- | ----------------------------------------------- | ------------------------------------------- |
| wsl --cleanup     | 清理 WSL 的缓存、无用资源                       | 解决 WSL 占用磁盘空间过大、启动卡顿问题     |
| wsl --repair      | 自动修复 WSL 的常见故障（如无法启动、版本异常） | 管理员权限执行，会重置 WSL 配置但不删除数据 |
| wsl --debug-shell | 启动 WSL 调试 shell（仅排障用）                 | 普通用户无需使用                            |



