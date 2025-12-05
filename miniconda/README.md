## MiniConda Get Start

Miniconda 是一个轻量级的 Anaconda 发行版，主要包含 Conda 包管理器和 Python。它是管理 Python 环境和依赖包的强大工具。

以下是 Miniconda (Conda) 的常用命令和简要使用教程：

### 1. 环境管理 (Environment Management)
这是使用 Conda 最核心的功能，可以为不同项目创建隔离的 Python 环境。

*   **创建新环境**
    创建一个名为 `myenv` 的环境，并指定 Python 版本为 3.9：
    ```bash
    conda create -n myenv python=3.9
    ```

*   **激活环境**
    进入（激活）你创建的环境：
    ```bash
    conda activate myenv
    ```

*   **退出环境**
    回到基础环境 (base)：
    ```bash
    conda deactivate
    ```

*   **列出所有环境**
    查看当前已创建的所有虚拟环境：
    ```bash
    conda env list
    # 或者
    conda info --envs
    ```

*   **删除环境**
    删除名为 `myenv` 的环境：
    ```bash
    conda remove -n myenv --all
    ```

### 2. 包管理 (Package Management)
在激活的环境中安装、更新或删除软件包。

*   **安装包**
    安装 `numpy` 包：
    ```bash
    conda install numpy
    ```
    也可以指定版本：
    ```bash
    conda install numpy=1.21
    ```

*   **列出已安装的包**
    查看当前环境中安装了哪些包：
    ```bash
    conda list
    ```

*   **搜索包**
    在仓库中搜索名为 `scikit-learn` 的包：
    ```bash
    conda search scikit-learn
    ```

*   **更新包**
    更新 `numpy`：
    ```bash
    conda update numpy
    ```
    更新 Conda 自身：
    ```bash
    conda update conda
    ```

*   **删除包**
    卸载 `numpy`：
    ```bash
    conda remove numpy
    ```

### 3. 镜像源配置 (Channel Configuration)
在国内使用 Conda，通常需要配置镜像源以提高下载速度（例如清华源或中科大源）。

*   **添加镜像源** (以清华源为例)
    ```bash
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
    conda config --set show_channel_urls yes
    ```

*   **查看当前配置的源**
    ```bash
    conda config --show channels
    ```

*   **恢复默认源**
    ```bash
    conda config --remove-key channels
    ```

### 4. 导出与导入环境
当你需要分享你的环境配置给他人，或者在另一台机器上复现环境时非常有用。

*   **导出环境配置** (导出为 YAML 文件)
    ```bash
    conda env export > environment.yml
    ```

*   **通过配置文件创建环境**
    ```bash
    conda env create -f environment.yml
    ```

### 快速入门流程总结
1.  安装 Miniconda。
2.  打开终端 (Terminal) 或 Anaconda Prompt。
3.  创建环境：`conda create -n study python=3.10`
4.  激活环境：`conda activate study`
5.  安装需要的库：`conda install pandas matplotlib`
6.  开始编写代码。
