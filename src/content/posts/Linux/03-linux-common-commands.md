---
title: "3.Linux 常用命令笔记"
slug: 03-linux-common-commands
published: 2026-05-20
description: "Linux 常用命令笔记，记录 Shell 基础、路径、帮助参数和常用命令实践。"
tags: ["Linux", "命令行", "Shell"]
category: Linux
draft: false
---
## 基本概念

- **Shell**：命令行解释器，常见为 `bash`（Bourne Again SHell）。
- **工作目录（Working Directory）**：当前所在路径，可用 `pwd` 查看。
- **根目录 `/`**：文件系统的起点，所有路径均由此衍生。
- **家目录 `~`**：用户主目录，通常位于 `/home/用户名`（Linux）或 `/Users/用户名`（macOS）。
- **`.` 和 `..`**：
  - `.` 表示当前目录
  - `..` 表示上级目录

### 🔍 通用帮助参数

| 参数           | 功能               |
| -------------- | ------------------ |
| `-h`, `--help` | 显示使用帮助       |
| `--version`    | 查看版本信息       |
| `man <命令>`   | 调出手册页（推荐） |

> 示例：  
> ```bash
> ls --help
> man grep
> ```

---

## 文件与目录操作

| 命令    | 功能                   | 示例                                            |
| ------- | ---------------------- | ----------------------------------------------- |
| `ls`    | 列出目录内容           | `ls`, `ls -l`, `ls -la`（包含隐藏文件）         |
| `cd`    | 切换工作目录           | `cd /`, `cd ~`, `cd ..`, `cd -`（返回上一目录） |
| `pwd`   | 显示当前绝对路径       | `pwd`                                           |
| `mkdir` | 创建目录               | `mkdir dir`, `mkdir -p a/b/c`（递归创建）       |
| `rmdir` | 删除空目录             | `rmdir empty_dir`                               |
| `rm`    | 删除文件或目录         | `rm file.txt`, `rm -r dir/`（删除非空目录）     |
| `cp`    | 复制文件/目录          | `cp a.txt b.txt`, `cp -r dir1/ dir2/`           |
| `mv`    | 移动或重命名           | `mv old.txt new.txt`, `mv file /tmp/`           |
| `touch` | 创建空文件或更新时间戳 | `touch config.log`                              |

> ⚠️ 危险提示：  
> ❌ 绝对禁止随意运行 `rm -rf /*` 或 `rm -rf /` —— 可能导致系统损毁！

---

## 查看与搜索文件

| 命令      | 功能                           | 示例                                                         |
| --------- | ------------------------------ | ------------------------------------------------------------ |
| `cat`     | 输出整个文件内容               | `cat notes.txt`                                              |
| `less`    | 分页浏览大文件（支持搜索）     | `less large.log` → 按 `/error` 搜索，`q` 退出                |
| `head`    | 显示前几行                     | `head -20 log.txt`                                           |
| `tail`    | 显示末尾行，常用于日志跟踪     | `tail -f /var/log/syslog`（实时输出新增内容）                |
| `grep`    | 全局正则匹配搜索               | `grep "Error" app.log`, `grep -ir "TODO" .`（忽略大小写+递归） |
| `find`    | 根据名称/大小/类型等搜索文件   | `find /home -name "*.conf"`, `find . -size +100M`            |
| `which`   | 查找可执行程序路径             | `which python3` → `/usr/bin/python3`                         |
| `whereis` | 查找二进制、源码、帮助文件位置 | `whereis nginx`                                              |

> 🔍 技巧组合：  
> ```bash
> # 找到最近修改的日志中含“failed”的行
> find /var/log -mtime -1 -name "*.log" | xargs grep "failed"
> ```

---

## 权限管理

Linux 文件权限格式示例：`-rwxr-xr-x`

- 第1位：表示文件类型  
  - `-`: 普通文件
  - `d`: 目录
  - `l`: 符号链接
- 后9位分为三组（每组 rwx）：
  - 用户（user）
  - 组（group）
  - 其他（others）

### 数字权限对照表

| 权限  | 数值 | 含义            |
| ----- | ---- | --------------- |
| `r`   | 4    | 读（read）      |
| `w`   | 2    | 写（write）     |
| `x`   | 1    | 执行（execute） |
| `rwx` | 7    | = 4+2+1         |

> 例如：`chmod 755 script.sh` → u=rwx, g=rx, o=rx

| 命令    | 功能              | 示例                                                       |
| ------- | ----------------- | ---------------------------------------------------------- |
| `chmod` | 修改权限          | `chmod 644 file.conf`, `chmod u+x,g-w script.sh`           |
| `chown` | 修改所有者/所属组 | `chown alice:developers file`, `sudo chown -R bob: /data/` |
| `chgrp` | 修改所属组        | `chgrp admin backup.tar` （较少直接使用）                  |

---

## 用户与组管理

| 命令                    | 功能                  | 示例                                                |
| ----------------------- | --------------------- | --------------------------------------------------- |
| `whoami`                | 显示当前用户名        | `whoami`                                            |
| `id`                    | 查看 UID/GID 和所属组 | `id`, `id alice`                                    |
| `su`                    | 切换用户              | `su -`（切换至 root 并加载环境）, `su alice`        |
| `sudo`                  | 以管理员权限执行命令  | `sudo apt install vim`                              |
| `adduser` / `useradd`   | 添加用户              | `sudo adduser newuser`（交互式），`useradd` 更底层  |
| `deluser` / `userdel`   | 删除用户              | `sudo deluser --remove-home jane`（连同家目录删除） |
| `passwd`                | 修改密码              | `passwd`（自己）, `sudo passwd tom`（他人）         |
| `groups`                | 查看用户所在组        | `groups`, `groups tom`                              |
| `groupadd` / `groupdel` | 创建/删除组           | `sudo groupadd docker`, `sudo groupdel tempgroup`   |

> 💡 建议：
> - Ubuntu 推荐使用 `adduser`（友好交互）
> - CentOS/RHEL 中 `useradd` 更常用
> - 新增用户后记得添加到适当组（如 `sudo`, `docker`）

---

## 进程管理

| 命令                | 功能                   | 示例                                       |
| ------------------- | ---------------------- | ------------------------------------------ |
| `ps`                | 查看进程快照           | `ps aux`, `ps -ef`, `ps aux \| grep nginx` |
| `top`               | 动态监控资源占用       | `top`（按 `q` 退出）                       |
| `htop`              | 更美观的 top（需安装） | `htop`（支持鼠标、颜色、树状视图）         |
| `kill`              | 终止指定 PID 的进程    | `kill 1234`, `kill -9 1234`（强制终止）    |
| `pkill` / `killall` | 按名字杀进程           | `pkill firefox`, `killall chrome`          |
| `bg` / `fg`         | 控制前台/后台任务切换  | `Ctrl+Z` → `bg` 放入后台 → `fg` 恢复       |

> 🕵️‍♂️ `ps aux` 关键字段说明：
> - `USER`: 运行用户
> - `PID`: 进程 ID
> - `%CPU` / `%MEM`: 资源占用率
> - `STAT`: 状态（S=睡眠，R=运行，Z=僵尸）
> - `COMMAND`: 启动命令

---

## 网络命令

| 命令                  | 功能                     | 示例                                                  |
| --------------------- | ------------------------ | ----------------------------------------------------- |
| `ip addr` / `ip link` | 查看 IP 地址与网卡状态   | `ip addr show`, `ip route`（替代 `ifconfig`）         |
| `ping`                | 测试网络连通性           | `ping google.com`                                     |
| `ss` / `netstat`      | 查看端口监听和连接       | `ss -tuln`（列出所有监听 TCP/UDP），`netstat -anp`    |
| `curl`                | 发送 HTTP 请求           | `curl http://example.com`, `curl -O https://file.zip` |
| `wget`                | 下载文件（断点续传）     | `wget -c http://largefile.iso`                        |
| `ssh`                 | 安全远程登录             | `ssh user@host`, `ssh -p 2222 user@server`            |
| `scp`                 | 安全复制文件（基于 SSH） | `scp file.txt user@remote:/tmp/`                      |
| `dig` / `nslookup`    | DNS 查询工具             | `dig example.com A`, `nslookup 8.8.8.8`               |

> 💡 工具选择建议：
> - 使用 `ip` 替代老旧的 `ifconfig`
> - `curl` 更适合 API 调试（灵活），`wget` 更适合批量下载
> - 排查端口问题优先用 `ss -tulnp`

---

## 磁盘与文件系统

| 命令               | 功能                       | 示例                                           |
| ------------------ | -------------------------- | ---------------------------------------------- |
| `df`               | 查看磁盘整体使用情况       | `df -h`（人类可读格式）                        |
| `du`               | 查看目录/文件占用空间      | `du -sh /var/log`, `du -h --max-depth=1 /home` |
| `mount` / `umount` | 挂载/卸载存储设备          | `sudo mount /dev/sdb1 /mnt`, `umount /mnt`     |
| `lsblk`            | 列出块设备结构（推荐）     | `lsblk` → 可视化硬盘分区                       |
| `fdisk -l`         | 查看磁盘分区详情（需sudo） | `sudo fdisk -l`                                |
| `sync`             | 强制将缓存写入磁盘         | `sync`（确保数据持久化）                       |

> ✅ 实践建议：
> - 插入 U 盘后运行 `lsblk` 快速识别设备名
> - 拷贝大量数据后执行 `sync` 再拔线，防止丢失

---

## 压缩与打包

| 命令                   | 功能                          | 示例                                                   |
| ---------------------- | ----------------------------- | ------------------------------------------------------ |
| `tar`                  | 打包/解包                     | `tar -cvf archive.tar folder/`, `tar -xvf archive.tar` |
| `gzip` / `gunzip`      | `.gz` 格式压缩/解压           | `gzip file.txt` → `file.txt.gz`, `gunzip file.gz`      |
| `.tar.gz`（常用组合）  | tar + gzip                    |                                                        |
| &nbsp;&nbsp;打包并压缩 | `tar -czvf data.tar.gz data/` |                                                        |
| &nbsp;&nbsp;解压       | `tar -xzvf data.tar.gz`       |                                                        |
| `zip` / `unzip`        | ZIP 格式（跨平台兼容性好）    | `zip -r project.zip src/`, `unzip project.zip`         |
| `bzip2` / `.tar.bz2`   | 高压缩比（较慢）              | `tar -cjvf big.tar.bz2 huge_dir/`                      |

> 📌 参数助记符：
> - `-c`: create（创建）
> - `-x`: extract（提取）
> - `-v`: verbose（显示过程）
> - `-f`: file（指定文件名，必须紧跟文件）
> - `-z`: gzip 压缩
> - `-j`: bzip2 压缩

---

## 文本处理命令

这些命令是 Shell 自动化的核心，配合管道 `|` 使用威力巨大。

| 命令     | 功能                     | 示例                                     |
| -------- | ------------------------ | ---------------------------------------- |
| `echo`   | 输出字符串               | `echo "Hello World"`                     |
| `printf` | 格式化输出               | `printf "%-10s %d\n" "Age:" 25`          |
| `cut`    | 提取特定列               | `echo "a:b:c" \| cut -d: -f2`（取第2段） |
| `sort`   | 排序                     | `sort names.txt`, `sort -nr`（倒序数字） |
| `uniq`   | 去除相邻重复行           | `sort file.txt \| uniq`                  |
| `wc`     | 统计行数/词数/字节数     | `wc -l file.txt`（统计行）               |
| `tr`     | 字符替换或删除           | `echo "hello" \| tr 'a-z' 'A-Z'` → HELLO |
| `awk`    | 强大的列处理工具         | `awk '{print $1}' access.log`（首字段）  |
| `sed`    | 流编辑器（搜索替换为主） | `sed 's/foo/bar/g' file.txt`（全局替换） |

> 🔥 经典组合案例：
> ```bash
> # 分析 Web 日志：统计访问最多的 IP
> cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -10
> ```

---

## 系统信息与监控

| 命令         | 功能                    | 示例                                               |
| ------------ | ----------------------- | -------------------------------------------------- |
| `uname -a`   | 查看内核和系统架构      | `Linux host 5.4.0... x86_64 GNU/Linux`             |
| `hostname`   | 显示或设置主机名        | `hostname myserver`                                |
| `uptime`     | 查看系统运行时间和负载  | `up 2 days, load average: 0.15, 0.10, 0.05`        |
| `free -h`    | 查看内存使用（易读）    | 包括 total, used, free, buff/cache                 |
| `dmesg`      | 查看内核启动消息        | `dmesg \| tail -20`（最近事件）                    |
| `journalctl` | 查看 systemd 日志       | `journalctl -u nginx.service --since "1 hour ago"` |
| `lscpu`      | CPU 架构详细信息        | 核心数、架构、频率等                               |
| `lsblk`      | 块设备列表（磁盘/分区） | `lsblk -f` 带文件系统类型                          |

---

## 实用技巧与快捷键

### 🔧 快捷键一览

| 快捷键     | 功能                         |
| ---------- | ---------------------------- |
| `Tab`      | 自动补全命令/路径            |
| `↑` / `↓`  | 上下浏览历史命令             |
| `Ctrl + C` | 终止当前正在运行的命令       |
| `Ctrl + Z` | 暂停任务（放入后台）         |
| `Ctrl + D` | 退出当前 shell（EOF）        |
| `Ctrl + L` | 清屏（等效于 `clear`）       |
| `Ctrl + R` | 反向搜索历史命令（模糊匹配） |

### 🔗 高频技巧

#### 历史命令操作
- `history`：查看命令历史
- `!n`：重新执行第 n 条命令
- `!!`：重复上一条命令（`sudo !!` 很有用）

#### 管道与重定向
- `command1 | command2`：将前者的输出作为后者的输入
- `>`：覆盖写入文件
- `>>`：追加写入文件
- `<`：从文件读取输入
- `2>`：重定向错误输出
- `&>` 或 `2>&1`：合并标准输出和错误输出

> 示例：
> ```bash
> # 将成功和错误都记录到日志
> ./run.sh > output.log 2>&1 &
> ```

#### 后台运行
- `nohup command &`：即使终端关闭仍继续运行
- `jobs`：查看后台任务
- `fg %1` / `bg %1`：恢复或放回后台任务（编号来自 `jobs`）

---

## 📘 学习建议

1. ✅ **多动手实践**  
   在云服务器（如腾讯云轻量应用）、WSL 或 VirtualBox 中搭建实验环境。

2. 🧠 **掌握 `man` 和 `--help`**  
   所有命令的帮助文档是最权威的学习资料。养成第一反应查手册的习惯。

3. 🔗 **学会组合指令**  
   使用管道 `|`、重定向 `>` 和变量 `$()` 构建复杂操作。

4. 🔒 **备份 + 小心敏感操作**  
   对 `rm`, `chmod`, `chown`, 编辑 `/etc` 下配置前请先备份。

5. 🤖 **逐步编写 Shell 脚本**  
   将重复动作自动化，例如部署脚本、日志清理脚本等。

6. 🔬 **善用 Google / ChatGPT / Stack Overflow**  
   输入错误信息直接搜索，99% 的问题已被解决过。

---

## 📌 常见场景命令示例

### 1. 搜索日志中的错误信息
```bash
grep -i "timeout\|error" /var/log/app.log
