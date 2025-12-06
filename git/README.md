# Git 使用指南

本文档涵盖 Git 的基本操作、分支管理与合并、暂存功能（Stash）以及代理配置。

---

## 目录

1. [Git 基本操作](#1-git-基本操作)
2. [分支管理与合并](#2-分支管理与合并)
3. [Git Stash 暂存功能](#3-git-stash-暂存功能)
4. [配置 Git 代理](#4-配置-git-代理)

---

## 1. Git 基本操作

Git 是 GitHub 的核心版本控制系统。以下是常见的配置与操作流程：

### 1.1 安装 Git

请访问 [git-scm.com/downloads](https://git-scm.com/downloads) 下载并安装适合您操作系统的最新版本。

### 1.2 配置用户信息

在终端中设置用户名：
```bash
git config --global user.name "你的用户名"
```

设置提交邮箱（需与 GitHub 账户邮箱一致）：
```bash
git config --global user.email "你的邮箱@example.com"
```

### 1.3 常用命令

| 命令 | 说明 |
|------|------|
| `git init` | 在当前目录初始化一个新的 Git 仓库 |
| `git clone [url]` | 克隆远程仓库到本地 |
| `git add [file]` | 将文件添加到暂存区 |
| `git commit -m "message"` | 提交暂存区的更改到本地仓库 |
| `git push` | 将本地提交推送到远程仓库 |
| `git pull` | 从远程仓库拉取并合并最新的更改 |

更多详细信息，请参阅官方文档 [Set up Git](https://docs.github.com/en/get-started/git-basics/set-up-git)。

---

## 2. 分支管理与合并

分支是 Git 中用于并行开发的核心功能。通过分支，您可以在不影响主线代码的情况下开发新功能或修复 Bug。

### 2.1 分支基本操作

```bash
# 查看所有本地分支（当前分支前有 * 标记）
git branch

# 查看所有分支（包括远程分支）
git branch -a

# 创建新分支
git branch <分支名>

# 切换到指定分支
git checkout <分支名>
# 或使用新命令（Git 2.23+）
git switch <分支名>

# 创建并切换到新分支（常用）
git checkout -b <分支名>
# 或
git switch -c <分支名>

# 删除本地分支
git branch -d <分支名>      # 安全删除（已合并的分支）
git branch -D <分支名>      # 强制删除（未合并的分支）

# 删除远程分支
git push origin --delete <分支名>

# 重命名当前分支
git branch -m <新分支名>
```

### 2.2 分支合并（Merge）

将一个分支的更改合并到当前分支：

```bash
# 先切换到目标分支（如 main）
git checkout main

# 合并指定分支到当前分支
git merge <要合并的分支名>

# 合并时添加合并信息
git merge <分支名> -m "合并说明"

# 取消正在进行的合并
git merge --abort
```

**合并类型：**

| 类型 | 说明 |
|------|------|
| Fast-forward | 当前分支没有新提交时，直接移动指针，不产生合并提交 |
| 三方合并 | 两个分支都有新提交时，创建一个新的合并提交 |

```bash
# 禁用 fast-forward，强制创建合并提交（保留分支历史）
git merge --no-ff <分支名>
```

### 2.3 变基（Rebase）

变基是另一种整合分支的方式，可以产生更线性的提交历史：

```bash
# 将当前分支变基到目标分支
git rebase <目标分支>

# 交互式变基（可以修改、合并、删除提交）
git rebase -i <commit-id>
git rebase -i HEAD~3   # 操作最近3个提交

# 取消正在进行的变基
git rebase --abort

# 解决冲突后继续变基
git rebase --continue
```

> ⚠️ **注意**：不要对已推送到远程的公共分支进行变基，这会导致其他协作者的历史混乱。

**Merge vs Rebase：**

| 特性 | Merge | Rebase |
|------|-------|--------|
| 提交历史 | 保留完整分支历史 | 产生线性历史 |
| 合并提交 | 会产生额外的合并提交 | 不产生合并提交 |
| 适用场景 | 公共分支、保留开发轨迹 | 个人分支、整理提交历史 |

### 2.4 解决合并冲突

当两个分支修改了同一文件的同一部分时，会产生冲突：

```bash
# 查看冲突文件
git status

# 冲突文件中的标记格式：
<<<<<<< HEAD
当前分支的内容
=======
要合并分支的内容
>>>>>>> feature-branch
```

**解决步骤：**

1. 打开冲突文件，手动编辑保留需要的内容
2. 删除冲突标记（`<<<<<<<`、`=======`、`>>>>>>>`）
3. 保存文件后添加到暂存区：
   ```bash
   git add <冲突文件>
   ```
4. 完成合并：
   ```bash
   git commit   # Merge 冲突
   # 或
   git rebase --continue   # Rebase 冲突
   ```

### 2.5 Cherry-pick（拣选提交）

将指定的提交应用到当前分支（不是整个分支合并）：

```bash
# 拣选单个提交
git cherry-pick <commit-id>

# 拣选多个提交
git cherry-pick <commit-id1> <commit-id2>

# 拣选一个范围的提交（不包含起始提交）
git cherry-pick <起始commit-id>..<结束commit-id>

# 只应用更改，不自动提交
git cherry-pick -n <commit-id>

# 取消 cherry-pick
git cherry-pick --abort
```

### 2.6 常用分支工作流

#### 功能分支工作流

```bash
# 1. 从 main 创建功能分支
git checkout main
git pull origin main
git checkout -b feature/新功能

# 2. 在功能分支上开发并提交
git add .
git commit -m "完成新功能"

# 3. 推送功能分支到远程
git push -u origin feature/新功能

# 4. 开发完成后，合并回 main
git checkout main
git pull origin main
git merge feature/新功能

# 5. 推送并删除功能分支
git push origin main
git branch -d feature/新功能
git push origin --delete feature/新功能
```

#### 分支命名规范建议

| 类型 | 命名格式 | 示例 |
|------|----------|------|
| 功能分支 | `feature/功能名` | `feature/user-login` |
| 修复分支 | `bugfix/问题描述` | `bugfix/fix-login-error` |
| 热修复分支 | `hotfix/问题描述` | `hotfix/security-patch` |
| 发布分支 | `release/版本号` | `release/v1.2.0` |

---

## 3. Git Stash 暂存功能

`git stash` 用于临时保存当前工作区的修改，让你可以切换到其他分支处理紧急任务，之后再恢复之前的工作状态。

### 3.1 基本使用

```bash
# 暂存当前修改（包括已跟踪文件的修改和暂存区的内容）
git stash

# 暂存时添加描述信息（推荐）
git stash save "正在开发的新功能"

# 或使用 push 命令（新版本推荐）
git stash push -m "正在开发的新功能"
```

### 3.2 查看暂存列表

```bash
# 查看所有暂存记录
git stash list
```

输出示例：
```
stash@{0}: On main: 正在开发的新功能
stash@{1}: WIP on main: abc1234 上一次提交信息
```

### 3.3 恢复暂存内容

```bash
# 恢复最近一次暂存的内容，并保留 stash 记录
git stash apply

# 恢复最近一次暂存的内容，并删除该 stash 记录
git stash pop

# 恢复指定的暂存内容
git stash apply stash@{1}
git stash pop stash@{1}
```

### 3.4 删除暂存记录

```bash
# 删除最近一次暂存记录
git stash drop

# 删除指定的暂存记录
git stash drop stash@{1}

# 清空所有暂存记录
git stash clear
```

### 3.5 高级用法

```bash
# 暂存所有文件，包括未跟踪的新文件
git stash -u
# 或
git stash --include-untracked

# 暂存所有文件，包括被 .gitignore 忽略的文件
git stash -a
# 或
git stash --all

# 查看暂存内容的详细差异
git stash show -p stash@{0}

# 从暂存创建新分支（避免恢复时产生冲突）
git stash branch <新分支名> stash@{0}
```

### 3.6 常用场景

| 场景 | 命令 |
|------|------|
| 临时切换分支修复 Bug | `git stash` → 切换分支 → 修复 → `git stash pop` |
| 保存多个工作进度 | `git stash push -m "功能A"` 和 `git stash push -m "功能B"` |
| 拉取远程更新前暂存本地修改 | `git stash` → `git pull` → `git stash pop` |

---

## 4. 配置 Git 代理

如果您在受限网络环境中（如公司内网或需要通过 VPN 访问），可能需要配置代理以便 Git 能正常连接 GitHub。

### 4.1 全局代理配置

#### 方法 A：配置 Git 全局代理（推荐用于本地开发）

在终端中运行以下命令（请将 `127.0.0.1:7890` 替换为您的代理地址和端口）：

```bash
# 设置 HTTP 代理
git config --global http.proxy http://127.0.0.1:7890

# 设置 HTTPS 代理
git config --global https.proxy http://127.0.0.1:7890
```

取消代理：
```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

#### 方法 B：使用环境变量（适用于 GitHub Actions Runner 或临时会话）

**Linux/macOS：**
```bash
export http_proxy=http://proxy.local:8080
export https_proxy=http://proxy.local:8080
```

**Windows PowerShell：**
```powershell
$env:http_proxy="http://proxy.local:8080"
$env:https_proxy="http://proxy.local:8080"
```

更多关于 Runner 和代理的高级配置，请参阅 [Using proxy servers with a runner](https://docs.github.com/en/actions/how-tos/manage-runners/use-proxy-servers)。

### 4.2 单个项目代理配置（Local 配置）

如果您只想让当前这一个特定的项目走代理，而不影响电脑上其他的 Git 项目，可以使用 `--local` 参数。

**步骤：**

1. 在终端中，先进入该项目的根目录：
    ```bash
    cd /path/to/your/project
    ```

2. 执行以下命令：
    ```bash
    # 设置 HTTP 代理
    git config --local http.proxy http://127.0.0.1:7890

    # 设置 HTTPS 代理
    git config --local https.proxy http://127.0.0.1:7890
    ```

**验证配置：**
```bash
git config --local --list
```

**取消配置：**
```bash
git config --local --unset http.proxy
git config --local --unset https.proxy
```

### 4.3 仅针对 GitHub 设置代理（特定域名配置）

如果您希望**仅让 GitHub 操作走代理**，而公司内部的代码库（如 GitLab、Gitee）直连不走代理，可以使用 Git 的按 URL 配置功能。

**设置命令：**
```bash
# 仅让 https://github.com 开头的仓库走代理
git config --global http.https://github.com.proxy http://127.0.0.1:7890
```

**原理解析：**
- 当 Git 访问 `github.com` 时，使用该代理
- 当 Git 访问 `gitlab.company.com` 或 `gitee.com` 时，没有匹配到规则，直接连接

**验证配置：**
```bash
git config --global --get-regexp http.*
```

输出示例：`http.https://github.com.proxy http://127.0.0.1:7890`

### 4.4 SSH 协议代理配置

> ⚠️ **重要提示**：上述 `git config` 设置**仅适用于 HTTPS 协议**的仓库地址（如：`https://github.com/username/repo.git`）。

如果您使用的是 **SSH 协议**（如：`git@github.com:username/repo.git`），需要修改 SSH 配置文件。

**配置文件位置：**
- Linux/macOS: `~/.ssh/config`
- Windows: `C:\Users\用户名\.ssh\config`

**配置内容：**
```ssh
Host github.com
    User git
    # macOS/Linux 示例 (假设代理端口 7890):
    ProxyCommand nc -X 5 -x 127.0.0.1:7890 %h %p
    
    # Windows 用户可使用 connect.exe:
    # ProxyCommand connect -S 127.0.0.1:7890 %h %p
```

---

## 附录：快速参考表

### Git 常用命令速查

| 命令 | 说明 |
|------|------|
| `git init` | 初始化仓库 |
| `git clone [url]` | 克隆仓库 |
| `git add .` | 暂存所有修改 |
| `git commit -m "msg"` | 提交更改 |
| `git push` | 推送到远程 |
| `git pull` | 拉取远程更新 |
| `git stash` | 暂存工作区 |
| `git stash pop` | 恢复暂存内容 |

### 分支操作速查

| 命令 | 说明 |
|------|------|
| `git branch` | 查看本地分支 |
| `git branch -a` | 查看所有分支（含远程） |
| `git checkout -b <分支名>` | 创建并切换分支 |
| `git switch <分支名>` | 切换分支 |
| `git merge <分支名>` | 合并分支 |
| `git rebase <分支名>` | 变基到指定分支 |
| `git branch -d <分支名>` | 删除本地分支 |
| `git cherry-pick <commit>` | 拣选指定提交 |

### 代理配置速查

| 场景 | 命令 |
|------|------|
| 设置全局代理 | `git config --global http.proxy http://127.0.0.1:7890` |
| 取消全局代理 | `git config --global --unset http.proxy` |
| 设置项目代理 | `git config --local http.proxy http://127.0.0.1:7890` |
| 仅 GitHub 代理 | `git config --global http.https://github.com.proxy http://127.0.0.1:7890` |