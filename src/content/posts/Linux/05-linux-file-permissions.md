---
title: "5.Linux 文件权限管理"
slug: 05-linux-file-permissions
published: 2026-05-20
description: "Linux 文件权限管理笔记，记录用户、用户组、读写执行权限和权限位含义。"
tags: ["Linux", "权限", "文件管理"]
category: Linux
draft: false
---
## 基本概念

在 Linux 中，**每一个文件和目录都有访问控制策略**，通过“用户 + 组 + 其他人”三重维度定义谁能读、写或执行。

### 三大核心要素

| 要素                 | 说明                                   |
| -------------------- | -------------------------------------- |
| **所有者（User）**   | 创建该文件的用户，默认拥有完整权限     |
| **所属组（Group）**  | 文件所属的用户组，组内成员可享有组权限 |
| **其他人（Others）** | 不属于所有者也不在所属组中的用户       |

> 示例：
> ```bash
> ls -l myfile.txt
> # 输出：
> -rw-r--r-- 1 alice developers 1024 Apr 5 10:00 myfile.txt
> ↑    ↑    ↑
> │    │    └── others: 只读
> │    └────── group: 只读
> └────────── user: 读写
> ```

---

## 权限表示方式

有两种常见表示法：**符号法（字母）** 和 **数字法（八进制）**

### 1. 符号表示法（rwx）

| 字符 | 含义            | 对文件的影响    | 对目录的影响               |
| ---- | --------------- | --------------- | -------------------------- |
| `r`  | 读（read）      | 可查看内容      | 可列出目录内容（`ls`）     |
| `w`  | 写（write）     | 可修改/删除内容 | 可创建、删除、重命名子文件 |
| `x`  | 执行（execute） | 可运行程序      | 可进入该目录（`cd`）       |

> ⚠️ 注意：
> - 目录要有 `x` 才能 `cd` 进入
> - 删除文件由父目录的 `w` 决定，不是文件本身的权限！

### 2. 数字表示法（八进制）

将每组权限转化为一个数字：

| 权限组合 | 计算方法 | 数值 |
| -------- | -------- | ---- |
| `---`    | 0+0+0    | 0    |
| `--x`    | 0+0+1    | 1    |
| `-w-`    | 0+2+0    | 2    |
| `-wx`    | 0+2+1    | 3    |
| `r--`    | 4+0+0    | 4    |
| `rw-`    | 4+2+0    | 6    |
| `r-x`    | 4+0+1    | 5    |
| `rwx`    | 4+2+1    | 7    |

> 示例：`755` = 所有者 rwx，组 r-x，其他 r-x

```text
7 5 5
│ │ └── others: r-x (5)
│ └───── group: r-x (5)
└─────── user: rwx (7)
```

---

## 权限对应关系（数值法）

| 数字 | 对应权限 | 常见用途               |
| ---- | -------- | ---------------------- |
| `7`  | `rwx`    | 完全控制（目录、脚本） |
| `6`  | `rw-`    | 文件编辑（如配置文件） |
| `5`  | `r-x`    | 可执行但不可改         |
| `4`  | `r--`    | 只读访问               |
| `0`  | `---`    | 无任何权限             |

✅ 最常用权限组合：

| 权限  | 应用场景                             |
| ----- | ------------------------------------ |
| `644` | 普通文件：作者可读写，其他人只读     |
| `755` | 脚本/程序：作者可改，所有人可运行    |
| `600` | 私密文件（如 SSH key）：仅所有者读写 |
| `700` | 私有目录（如 `~/.ssh`）              |

---

## 权限管理命令

| 命令    | 功能       | 示例                                                  |
| ------- | ---------- | ----------------------------------------------------- |
| `chmod` | 修改权限   | `chmod 755 script.sh`, `chmod u+x,g-w file`           |
| `chown` | 修改所有者 | `chown alice file.txt`, `chown alice:developers dir/` |
| `chgrp` | 修改所属组 | `chgrp team dir/`（较少直接使用）                     |

### ✅ chmod 使用方式

#### 数字模式（推荐用于批量设置）
```bash
chmod 644 config.conf
chmod 755 backup.sh
chmod -R 755 myproject/   # 递归赋权
```

#### 符号模式（灵活调整某一项）
```bash
chmod u+x script.sh        # 用户增加执行权
chmod g-w data.txt         # 组减少写权限
chmod o=r file.log         # 其他人设为只读
chmod a+r notes.txt        # all 用户都加读权限
```

> 提示字符：
> - `u`=user, `g`=group, `o`=others, `a`=all
> - `+`=添加, `-`=移除, `=`=精确赋值

---

## 文件所有者与所属组

### 查看当前权限
```bash
ls -l filename
```

### 更改所有者和组

```bash
# 修改所有者
sudo chown bob file.txt

# 修改所有者和组
sudo chown alice:developers project/

# 仅修改组
sudo chgrp admins backup.tar

# 递归更改整个目录树
sudo chown -R www-data:www-data /var/www/html/
```

> ❗ 注意：只有 root 或文件所有者才能使用 `chown`

---

## 默认权限与 umask

新创建的文件/目录会根据 `umask`（掩码）自动设定初始权限。

### ☁️ umask 工作原理

系统从“最大权限”中减去 umask 值得到实际权限：

| 类型 | 最大权限                | 减去 umask 后结果      |
| ---- | ----------------------- | ---------------------- |
| 文件 | `666`（不能默认可执行） | 如 `umask 022` → `644` |
| 目录 | `777`                   | 如 `umask 022` → `755` |

### 查看当前 umask
```bash
umask
# 输出例如：0022
```

### 临时修改 umask
```bash
umask 0077
touch private.txt
# 新建文件权限变为 600
```

> 推荐：
> - 开发服务器可设 `umask 0002`（允许组内共享）
> - 安全要求高时设 `umask 0077`

---

## 特殊权限：SUID、SGID、Sticky Bit

这些是超出普通 rwx 的高级权限，用第四位数字表示（如 `4755`）。

| 名称           | 数字 | 符号                  | 作用                                             |
| -------------- | ---- | --------------------- | ------------------------------------------------ |
| **SUID**       | `4`  | `s` in user execute   | 执行时以**文件所有者的身份运行**                 |
| **SGID**       | `2`  | `s` in group execute  | 执行时以**所属组的身份运行**；目录中新文件继承组 |
| **Sticky Bit** | `1`  | `t` in others execute | 目录中只能由**自己或 root 删除自己的文件**       |

### 使用示例

#### 1. SUID 示例：让普通用户运行 passwd 修改 shadow
```bash
ls -l /usr/bin/passwd
# 输出包含：
# -rwsr-xr-x 1 root root ...
#     ↑               ← s 表示设置了 SUID
```
→ 普通用户可修改 `/etc/shadow` 中自己的密码条目，因为此时命令以 root 身份运行。

#### 设置 SUID
```bash
chmod u+s backup_tool.sh
# 或
chmod 4755 backup_tool.sh
```

#### 2. SGID on Directory：协作目录
```bash
mkdir /shared/project
chgrp developers /shared/project
chmod 2775 /shared/project   # SGID + rwxrwxr-x
```
> 效果：任何人在该目录下创建的文件，所属组自动设为 `developers`

#### 3. Sticky Bit：防误删公共目录
```bash
chmod +t /tmp
# 或
chmod 1777 /tmp
```
> 即使目录所有人都可写，也不能删除别人的文件（除非你是 owner 或 root）

> ⚠️ 安全提醒：滥用 SUID/SGID 会导致安全隐患，应定期审计：
> ```bash
> # 查找系统中所有带特殊权限的文件
> find / -type f \( -perm -4000 -o -perm -2000 \) 2>/dev/null
> ```

---

## 访问控制列表 ACL（扩展权限）

当标准“用户+组+其他”不够用时，可使用 ACL 实现更细粒度控制。

### 是否支持 ACL？

多数现代文件系统（ext4, xfs）默认支持。启用需挂载选项含 `acl`。

### 常用命令

| 命令      | 功能          |
| --------- | ------------- |
| `setfacl` | 设置 ACL 权限 |
| `getfacl` | 查看 ACL 权限 |

### ACL 使用示例

#### 允许用户 `bob` 读取只有 alice 可读的文件
```bash
getfacl secret.txt
setfacl -m u:bob:r secret.txt
getfacl secret.txt
```
输出将显示额外一行：
```text
user:bob:r--
```

#### 允许组 `interns` 对目录有写权限
```bash
setfacl -m g:interns:rwX /projects/doc/
```
> `X` 表示“如果原本有执行权则赋给”，安全又方便

#### 删除特定 ACL 条目
```bash
setfacl -x u:bob secret.txt
```

#### 清空所有扩展 ACL
```bash
setfacl -b secret.txt
```

---

## 常见问题解答

❓ **Q：为什么我不能进入某个目录，明明有 `r` 权限？**
✅ A：缺少 `x` 权限。对目录来说，`x` 是能否 `cd` 或访问其内部的关键。

---

❓ **Q：为什么删除文件不需要对该文件有写权限？**
✅ A：删除是由**父目录的写权限**决定的。只要对目录有 `w+x`，就可以删除里面的文件。

---

❓ **Q：如何让新建文件自动属于某个组？**
✅ A：结合 `SGID` + 组目录：
```bash
sudo chgrp devs shared_dir
sudo chmod 2775 shared_dir   # SGID + 可读写执行
```
之后该目录下的所有新文件都会继承 `devs` 组。

---

❓ **Q：ACL 和普通权限哪个优先级高？**
✅ A：ACL 是对传统权限的**补充**，它提供了更多入口。系统按照如下顺序判断权限：
1. 如果是文件所有者 → 使用 owner 权限
2. 如果匹配某条 ACL 规则 → 使用该规则
3. 否则按 group/others 判断

---

❓ **Q：怎样查看哪些文件设置了 SUID？**
✅ A：
```bash
find /usr/bin -perm -4000 -type f 2>/dev/null
```

---

## 📌 实用场景示例

### ✅ 场景 1：搭建团队共享开发目录

```bash
sudo mkdir /project/shared
sudo chgrp devteam /project/shared
sudo chmod 2775 /project/shared    # SGID + 可读写执行
# 所有人在此创建的文件自动归 devteam 组
```

---

### ✅ 场景 2：保护日志不被篡改，但仍允许写入

```bash
sudo touch /var/log/app.log
sudo chown root:syslog /var/log/app.log
sudo chmod 640 /var/log/app.log
# syslog 组成员可读，其他人无权访问
```

---

### ✅ 场景 3：给特定用户访问私密文件的权限（ACL）

```bash
# 只允许审计员 audited 读取敏感配置
sudo setfacl -m u:auditor:r /etc/secrets.conf
getfacl /etc/secrets.conf
```

---

### ✅ 场景 4：修复 Web 项目权限（典型 LAMP 配置）

```bash
sudo chown -R www-data:www-data /var/www/html/
# 但保留脚本可执行性
find /var/www/html -type f -exec chmod 644 {} \;
find /var/www/html -type d -exec chmod 755 {} \;
```

---

### ✅ 场景 5：禁止任何人删除上传目录中的他人文件

```bash
sudo chmod +t /var/www/uploads/
# 等价于：
sudo chmod 1777 /var/www/uploads/
```

---

## 📘 学习建议

1. ✅ **先理解再动手**
   - 多用 `ls -l` 观察不同权限的表现。
   - 在虚拟机或 Docker 环境中实验 SUID/ACL。

2. 🔐 **最小权限原则**
   - 不要轻易赋予 `777` 或过度使用 SUID。
   - 高风险操作尽量用 ACL 替代放宽整体权限。

3. 🧰 **善用工具辅助检查**
   - 使用 `stat filename` 查看详细权限信息
   - 编写脚本定期扫描异常权限（如世界可写的配置文件）

4. 🤖 **自动化权限设置**
   - 将常用权限配置写成 Shell 脚本或 Ansible Playbook

5. 📚 **延伸阅读**
   - `man chmod`, `man acl`, `man capabilities`
   - SELinux / AppArmor：更高级的安全模块（企业级）

