---
title: "4.Linux 用户与用户组管理"
slug: 04-linux-user-group-management
published: 2026-05-20
description: "Linux 用户与用户组管理笔记，记录 UID、GID、主组、附加组和用户相关配置文件。"
tags: ["Linux", "用户管理", "用户组"]
category: Linux
draft: false
---
## 基本概念

在 Linux 中，**每个进程都以某个用户身份运行**，系统的资源访问控制依赖于“用户”和“组”的权限机制。

### 核心概念

| 名称                              | 说明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| **UID**                           | 用户 ID（User ID），系统识别用户的唯一数字标识。如 root 的 UID 是 `0` |
| **GID**                           | 组 ID（Group ID），用于标识主组或附加组                      |
| **用户名**                        | 人类可读的名称（如 `alice`），对应一个 UID                   |
| **主组（Primary Group）**         | 用户默认归属的组，创建文件时自动设置为该组                   |
| **附加组（Supplementary Group）** | 可加入多个组以获取额外权限                                   |

> 示例：
> 用户 `devuser` 主组为 `developers`，附加组包括 `docker`, `sudo` —— 可运行 Docker 并提权执行 `sudo`

---

## 用户相关文件

这些是系统中存储用户和组信息的核心配置文件：

| 文件路径       | 功能描述                                                     |
| -------------- | ------------------------------------------------------------ |
| `/etc/passwd`  | 存储所有用户基本信息（不包含密码！）格式：`用户名:passwd占位符:UID:GID:描述:家目录:Shell` |
| `/etc/shadow`  | 加密保存用户密码哈希（仅 root 可读），含密码过期策略         |
| `/etc/group`   | 定义组名、GID 及成员列表                                     |
| `/etc/gshadow` | 组密码与管理员（极少使用）                                   |

> 🔍 查看示例：
> ```bash
> head -3 /etc/passwd
> # 输出类似：
> root:x:0:0:root:/root:/bin/bash
> alice:x:1001:1001:Alice Chen:/home/alice:/bin/zsh
> ```

---

## 用户管理命令

| 命令                  | 功能                                 | 推荐使用场景                            |
| --------------------- | ------------------------------------ | --------------------------------------- |
| `adduser`             | 交互式添加用户（Debian/Ubuntu 推荐） | 初学者友好，自动建家目录、设 Shell      |
| `useradd`             | 底层命令，功能强大但需手动配置       | 需精细控制时使用（如无家目录账户）      |
| `deluser` / `userdel` | 删除用户                             | Ubuntu 用 `deluser`，其他多用 `userdel` |
| `passwd`              | 设置或修改密码                       | 所有用户均可改自身密码；root 可改为他人 |
| `id`                  | 查看用户的 UID、GID 和所属组         | `id alice`                              |
| `whoami`              | 显示当前登录用户名                   | 快速确认身份                            |
| `su`                  | 切换用户身份                         | `su - alice`（推荐加 `-` 加载完整环境） |
| `sudo`                | 以管理员权限运行命令                 | 非 root 用户提权执行敏感操作            |

### ✅ 用户管理常用命令示例

```bash
# 添加新用户（交互式）
sudo adduser john

# 添加用户但不创建家目录
sudo useradd -M monitor

# 添加用户并指定 Shell
sudo useradd -m -s /bin/zsh devuser

# 设置密码
sudo passwd john

# 删除用户及其家目录
sudo deluser --remove-home jane      # Ubuntu
sudo userdel -r bob                   # 其他系统

# 切换到用户 alice
su - alice

# 使用 sudo 运行命令（避免切换）
sudo systemctl restart nginx
```

---

## 组管理命令

| 命令       | 功能                        |
| ---------- | --------------------------- |
| `groupadd` | 创建新组                    |
| `groupdel` | 删除组                      |
| `groups`   | 查看用户所属的所有组        |
| `usermod`  | 修改用户属性，如添加/删除组 |

### 🧩 常见组操作命令

```bash
# 创建开发组
sudo groupadd developers

# 将用户 alice 添加到 developers 组（附加组）
sudo usermod -aG developers alice

# 查看用户当前所在组
groups alice
# 输出：alice : alice developers docker

# 从某组中移除用户？ → 需编辑 /etc/group 或使用 gpasswd
sudo gpasswd -d alice developers

# 删除组
sudo groupdel tempgroup
```

> ❗ 注意：`-aG` 中的 `-a` 表示“追加”，否则会覆盖原有附加组！

---

## 权限与组的关系

### 文件权限回顾

```text
-rw-r--r-- 1 alice developers 4096 Apr 5 10:00 config.ini
  ↑   ↑   ↑
  │   │   └── others: 只读
  │   └────── group: 只读
  └───────── user: 读写
```

- 当用户 `bob` 属于 `developers` 组时，他对这个文件拥有 **只读权限**。
- 因此：**将用户加入特定组是实现协作与权限共享的关键方式之一**

### 典型应用场景

| 组名             | 用途                                     |
| ---------------- | ---------------------------------------- |
| `sudo` / `wheel` | 允许用户使用 `sudo` 提权（取决于发行版） |
| `docker`         | 允许无需 root 运行 Docker 命令           |
| `www-data`       | Web 服务相关的权限控制                   |
| `plugdev`        | USB 设备插拔权限（桌面环境）             |

✅ 示例：让开发人员无需密码运行 Docker

```bash
# 创建 docker 组（通常已存在）
sudo groupadd docker

# 将用户加入 docker 组
sudo usermod -aG docker devuser

# 注销重登后即可运行：
docker ps
```

---

## 实用技巧

### 🔐 使用 `sudo` 替代频繁切换 root

- 编辑 sudoers：`sudo visudo`
- 允许用户免密运行 sudo（谨慎！）：
  ```bash
  devuser ALL=(ALL) NOPASSWD: ALL
  ```
- 更安全做法：只允许某些命令
  ```bash
  deploy ALL=/bin/systemctl restart nginx, /usr/bin/git pull
  ```

### 📁 锁定/解锁账户

```bash
sudo passwd -l username   # 锁定（禁止登录）
sudo passwd -u username   # 解锁
```

### 🧼 清理无效用户

```bash
# 查看所有用户
cut -d: -f1 /etc/passwd

# 查看最近未登录的用户（需安装 lastlog）
lastlog | grep Never

# 删除闲置超一年的用户（脚本化处理）
```

---

## 常见问题解答

❓ **Q：`adduser` 和 `useradd` 有什么区别？**
✅ A：
- `adduser` 是 Perl 脚本，在 Ubuntu 上提供交互体验，自动创建家目录、复制配置文件等。
- `useradd` 是底层二进制命令，行为更灵活但也更复杂，常用于自动化脚本。

---

❓ **Q：为什么新用户加入组后还要重新登录？**
✅ A：
用户的组成员信息是在登录时加载到会话中的。如果不重新登录，`groups` 命令仍显示旧信息。可用以下方法刷新：

```bash
# 方法一：新开终端 / 注销重登
# 方法二：使用 newgrp（临时切换主组）
newgrp developers
```

> ⚠️ 注意：`newgrp` 只影响当前 shell 会话，并不会永久改变主组。

---

❓ **Q：如何批量创建用户？**
✅ A：使用脚本 + `useradd`

```bash
#!/bin/bash
while read name; do
  sudo useradd -m -s /bin/bash "$name"
  echo "默认密码：Password123"
  echo "$name:Password123" | sudo chpasswd
done < users.txt
```

---

❓ **Q：怎样查看谁有 `sudo` 权限？**
✅ A：

```bash
# 查看属于 sudo 组的用户
getent group sudo

# 或查看 wheel 组（CentOS/RHEL）
getent group wheel

# 查看 sudoers 配置
sudo cat /etc/sudoers | grep -v "^#" | grep -v "^$"
```

---

## 📌 场景示例

### ✅ 场景 1：为新员工创建账号并授权

```bash
# 1. 添加用户
sudo adduser amy

# 2. 加入开发组与 sudo 权限
sudo usermod -aG developers,sudo amy

# 3. 设置初始密码
sudo passwd amy

# 4. 提醒首次登录修改密码
echo "请让 Amy 首次登录后执行: passwd"
```

---

### ✅ 场景 2：禁止普通用户登录，仅用于服务运行

```bash
# 创建无家目录、无 Shell 的专用服务账户
sudo useradd -r -s /usr/sbin/nologin appuser

# 设置其拥有日志目录权限
sudo chown -R appuser:appuser /var/log/myapp
```

---

### ✅ 场景 3：快速列出所有具有 sudo 权限的用户

```bash
# 方法一：查属于 sudo 组的用户
getent group sudo | cut -d: -f4 | tr ',' '\n'

# 方法二：检查 sudoers 文件中直接授权的用户
grep -E '^[^#]+' /etc/sudoers | grep -v '^Defaults' | cut -d' ' -f1
```

---

## 📘 学习建议

1. ✅ **不要在生产机上随意删除用户**
   - 若不确定，先禁用账户（`passwd -l`），观察是否影响服务。

2. 🔁 **掌握“最小权限原则”**
   - 不要轻易将用户加入 `sudo` 组，应通过精确授权控制风险。

3. 🔎 **熟悉 `/etc/passwd` 和 `/etc/group` 结构**
   - 手动查看比盲敲命令更能理解底层机制。

4. 🛠️ **动手搭建测试环境**
   - 使用 VirtualBox 或 WSL 创建 Ubuntu 虚拟机进行练习。

5. 🤖 **尝试写用户管理脚本**
   - 自动化创建用户、分配组、生成报告等任务。

