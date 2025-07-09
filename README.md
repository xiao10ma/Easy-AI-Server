<h1 align="center">AI服务器极简入门 Easy-AI-Server</h1>

<p align="center"> </p>

> 本文将综合介绍AI科研服务器的一些工具的使用，供大家入门。如果在学习过程中，遇到一些问题，可以问问GPT/Deepseek这类AI工具或Google。若文章有错误，欢迎提issue指正 :D

# Contents - 目录

<nav>
  <ul>
    <li><a href="#-linux-常用命令与基础知识">1. Linux 常用命令与基础知识 - Basic Linux Commands</a>
      <ul>
        <li><a href="#-一远程登录">1.1 远程登录 - Remote Login</a></li>
        <li><a href="#-二文件与目录操作">1.2 文件与目录操作 - File Operations</a></li>
        <li><a href="#-三服务器资源查看">1.3 服务器资源查看 - Resource Monitoring</a></li>
        <li><a href="#-四实用技巧">1.4 实用技巧 - Useful Tips</a></li>
      </ul>
    </li>
    <li><a href="#-anaconda--miniconda-安装与配置">2. Anaconda/Miniconda 安装与配置 - Installation & Setup</a>
      <ul>
        <li><a href="#-安装位置建议">2.1 安装位置建议 - Installation Location</a></li>
        <li><a href="#-安装步骤以-miniconda-为例">2.2 安装步骤 - Installation Steps</a></li>
        <li><a href="#-加速安装配置国内镜像源">2.3 加速安装：配置国内镜像源 - Speed Up with Mirrors</a></li>
      </ul>
    </li>
  </ul>
</nav>

## 🐧 Linux 常用命令与基础知识

为便于说明，以下示例中使用了通用占位符，如 {username}、{dirname} 等，表示你需要根据自己的实际情况进行替换。例如，如果你的用户名是 mazipei，则应将 {username} 替换为 mazipei。

为提升可读性，我会用花括号 {} 标注需要替换的内容，请根据实际情况进行修改。

### 🌐 一、远程登录

```
ssh {username}@{ip}
# example: ssh mazipei@10.10.10.10
```

如果没有设置过钥匙，第一次登陆时通常会有：

```bash
❯ ssh mazipei@10.10.10.10
no such identity: /Users/mazipei/.ssh/id_rsa: No such file or directory
mazipei@10.28.5.13's password:
```

输入密码即可（linux在输入密码时不会显示）

<details>
<summary>（可选）服务器免密登陆</summary>

每次登录服务器都需要输入密码，比较麻烦。可以通过设置 SSH 密钥实现免密登录：

1. 在本地生成 SSH 密钥对

   ```bash
   ssh-keygen -t rsa -b 4096 -C {"your_email@example.com"}

   # example: ssh-keygen -t rsa -b 4096 -C 123456@qq.com
   ```

   回车后系统会提示你保存路径，默认是：

   ```
   /Users/{username}/.ssh/id_rsa
   ```

   可以直接回车使用默认路径，或者输入你自定义的路径。接着提示你设置密码，可以直接回车跳过。

   生成成功后，会看到两份文件：

   - /Users/{username}/.ssh/id_rsa：私钥（不要泄露）
   - /Users/{username}/.ssh/id_rsa.pub：公钥（可以共享给服务器）
2. 将公钥上传到服务器
   运行以下命令，把公钥追加到服务器的 `/Users/{username}/.ssh/authorized_keys` 文件中：

   ```
   ssh-copy-id {username}@{ip}
   ```

   如果你使用的是自定义密钥路径，可以加 `-i` 参数：

   ```
   ssh-copy-id -i {your_path}/id_rsa.pub {username}@{ip}
   ```
3. 测试免密登录

   ```bash
   ssh {username}@{ip}
   ```

   如果不再提示输入密码，则说明配置成功。

**🧠 附加技巧：多服务器共享一个 SSH 密钥**
如果你有多台服务器，其实无需为每台都生成不同的密钥。只要将同一个公钥添加到每台服务器的 `~/.ssh/authorized_keys` 中，即可通过 同一个私钥实现免密登录多个服务器。
为方便管理和免密登录，可以在本地的 ~/.ssh/config 文件中添加如下配置：

```bash
# ~/.ssh/config

Host {alias1}
    HostName {ip_address1}
    User {username1}
    IdentityFile {private_key_path}

Host {alias2}
    HostName {ip_address2}
    User {username2}
    IdentityFile {private_key_path}
```

例如：

```bash
Host 3090
    HostName 10.10.10.10
    User mazipei
    IdentityFile ~/.ssh/id_rsa

Host 4090
    HostName 10.10.10.11
    User zipeima
    IdentityFile ~/.ssh/id_rsa
```

然后你就可以通过以下命令直接登录：

```
ssh 3090
ssh 4090
```

或者用vscode：

https://github.com/user-attachments/assets/4e5686ab-5496-44ee-83c3-c5b160802701

</details>

---

### 📁 二、文件与目录操作

这一部分建议大家可以跟着一起在终端敲一遍。

| 操作         | 命令                    | 说明                                                                                                     |
| ------------ | ----------------------- | -------------------------------------------------------------------------------------------------------- |
| 查看当前路径 | `pwd`                 | 显示当前所在的目录（print working directory）                                                            |
| 列出文件     | `ls`                  | 列出当前目录下的文件和文件夹                                                                             |
| 列出详细信息 | `ls -l`               | 显示包含权限、大小、时间等信息的文件列表                                                                 |
| 切换目录     | `cd {dirname/}`       | 进入某个目录，比如 `cd data/` <br />👉 注意：这里的 `dirname` 是你服务器实际文件夹名字，不是固定写法 |
| 返回上级目录 | `cd ..`               | 返回到当前目录的上一级目录（父目录）                                                                     |
| 当前目录     | `.`                   | 表示当前目录，常用于路径组合或执行脚本，例如 `./run.sh`                                                |
| 上级目录     | `..`                  | 表示当前目录的上一级，例如 `../data/` 表示父目录下的 `data` 文件夹                                   |
| 创建目录     | `mkdir {new_folder/}` | 创建一个文件夹，名字可以自己起<br />👉 例如：`mkdir results`                                           |
| 删除文件     | `rm {file.txt}`       | 删除文件（不可恢复）                                                                                     |
| 删除目录     | `rm -r {folder/}`     | 递归删除目录及其内容                                                                                     |
| 复制文件     | `cp {a.txt} {b.txt}`  | 将 `a.txt` 复制为 `b.txt`                                                                            |
| 移动/重命名  | `mv {a.txt} {b.txt}`  | 改名或移动文件                                                                                           |

---

### 🖥 三、服务器资源查看

| 操作                   | 命令                        | 说明                                                                |
| ---------------------- | --------------------------- | ------------------------------------------------------------------- |
| 查看显卡信息           | `nvidia-smi`或 `nvitop` | 查看 GPU 状态；强烈推荐使用 `nvitop`，下文将说明 `nvitop`的安装 |
| 查看CPU信息            | `top`或 `htop`          | 实时查看CPU/内存/进程占用                                           |
| 查看硬盘空间           | `df -h .`                 | 查看各挂载点的磁盘使用                                              |
| 查看当前路径下空间占用 | `du -sh *`                | 各文件/文件夹大小（简洁）                                           |

---

### 🧼 四、实用技巧

* `Tab` 自动补全命令或文件名
* `Ctrl + C` 强制终止当前运行的命令

一般来说，服务器上会有两个常见的工作目录：

1. **用户主目录**：即 `~`，对应路径通常为 `/home/{username}` 或 `/User/{username}`，主要用于存放个人配置文件（如 `.bashrc`、`.ssh/` 等）以及轻量级的数据或脚本。
2. **数据目录**：用于存放数据、代码或实验结果等大型文件。该目录结构在不同实验室的服务器上可能有所不同，建议向实验室同学或管理员确认具体路径。

本文以 `HDD_DISK/users/mazipei` 为例，将其作为数据目录，用于存放项目代码和实验数据等文件。

---

## 🐍 Anaconda / Miniconda 安装与配置

在服务器上使用 Anaconda（或更轻量的 Miniconda）可以方便地管理 Python 环境和依赖库。由于不同项目通常依赖不同版本的 Python、PyTorch、CUDA 等组件，使用 Conda 环境进行隔离是一种推荐的做法。

### 📦 安装位置建议

深度学习项目常常依赖较大的包（如 `torch`、`transformers`、`diffusers` 等），安装体积可能达到数 GB。为了避免占用主目录空间（尤其是 `/home/{username}` 容量较小的情况），**建议将 Conda 安装至数据目录**。

### 🛠 安装步骤（以 Miniconda 为例）

1. 下载 Miniconda 安装脚本
   
   可在[官网](https://www.anaconda.com/docs/getting-started/miniconda/main)下载对应版本或使用 `wget` 命令：

   ```bash
   wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
   ```
2. 运行安装脚本

   ```bash
   bash Miniconda3-latest-Linux-x86_64.sh
   ```

   安装过程中会提示输入安装路径，建议填写你自己的数据目录路径：

   ```
   /HDD_DISK/users/{username}/miniconda3
   ```

   其余一律回车。安装完成后，`source ~/.bashrc`
3. 测试 Conda 是否安装成功

   ```bash
   conda --version
   ```

   会输出类似下面的内容，表示 Conda 已正常安装并可用：

   ```bash
   conda 24.3.0
   ```

### 🚀 加速安装：配置国内镜像源

由于服务器访问 PyPI 官方源速度较慢，可能导致安装 Python 包时超时或下载缓慢，建议配置国内镜像源以提升安装速度。

```bash
# 升级 pip
python -m pip install --upgrade pip
# 设置 pip 使用清华源
pip config set global.index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
```

你也可以查看配置是否生效：

```bash
pip config list
```

#### 💡 常见国内源列表（可选）

| 镜像源       | 地址                                                     |
| ------------ | -------------------------------------------------------- |
| 清华大学     | `https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple` |
| 中国科技大学 | `https://pypi.mirrors.ustc.edu.cn/simple`              |
| 阿里云       | `https://mirrors.aliyun.com/pypi/simple/`              |

---
