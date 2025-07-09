<h1 align="center">AI服务器极简入门 Easy-AI-Server</h1>

<p align="center"> </p>

> 本文将综合介绍AI科研服务器的一些工具的使用，供大家入门。如果在学习过程中，遇到一些问题，可以问问GPT/Deepseek这类AI工具或Google。若文章有错误，欢迎提issue指正 :D

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

#### （可选）服务器免密登陆

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

### 📁 二、文件与目录操作

这一部分建议大家可以跟着一起在终端敲一遍。

| 操作         | 命令                  | 说明                                                                                                     |
| ------------ | --------------------- | -------------------------------------------------------------------------------------------------------- |
| 查看当前路径 | `pwd`               | 显示当前所在的目录（print working directory）                                                            |
| 列出文件     | `ls`                | 列出当前目录下的文件和文件夹                                                                             |
| 列出详细信息 | `ls -l`             | 显示包含权限、大小、时间等信息的文件列表                                                                 |
| 切换目录     | `cd {dirname/}`       | 进入某个目录，比如 `cd data/` <br />👉 注意：这里的 `dirname` 是你服务器实际文件夹名字，不是固定写法 |
| 返回上级目录 | `cd ..`             | 返回到当前目录的上一级目录（父目录）                                                                     |
| 当前目录     | `.`                 | 表示当前目录，常用于路径组合或执行脚本，例如 `./run.sh`                                                |
| 上级目录     | `..`                | 表示当前目录的上一级，例如 `../data/` 表示父目录下的 `data` 文件夹                                   |
| 创建目录     | `mkdir {new_folder/}` | 创建一个文件夹，名字可以自己起<br />👉 例如：`mkdir results`                                           |
| 删除文件     | `rm {file.txt}`       | 删除文件（不可恢复）                                                                                     |
| 删除目录     | `rm -r {folder/}`     | 递归删除目录及其内容                                                                                     |
| 复制文件     | `cp {a.txt} {b.txt}`    | 将 `a.txt` 复制为 `b.txt`                                                                            |
| 移动/重命名  | `mv {a.txt} {b.txt}`    | 改名或移动文件                                                                                           |

### 🖥 三、服务器资源查看

| 操作                   | 命令                        | 说明                                                                |
| ---------------------- | --------------------------- | ------------------------------------------------------------------- |
| 查看显卡信息           | `nvidia-smi`或 `nvitop` | 查看 GPU 状态，强烈推荐使用 `nvitop`，下文将说明 `nvitop`的安装 |
| 查看CPU信息            | `top`或 `htop`          | 实时查看CPU/内存/进程占用                                           |
| 查看硬盘空间           | `df -h .`                 | 查看各挂载点的磁盘使用                                              |
| 查看当前路径下空间占用 | `du -sh *`                | 各文件/文件夹大小（简洁）                                           |

---

### 🧼 四、实用技巧

* `Tab` 自动补全命令或文件名
* `Ctrl + C` 强制终止当前运行的命令

一般而言，服务器有个人的用户目录和数据目录；用户目录为 `~`，也就是 /User/{username}，主要放置一些配置文件；数据目录各个实验室服务器应该有所不同，具体可以询问实验室的同学。本文以 `HDD_DISK/users/mazipei`为例，设置为数据目录。

# anaconda/miniconda 安装及配置

深度学习一般都需要安装 `torch`，这需要几个GB，因此建议将
