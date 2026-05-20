---
title: "6.Linux 软件包管理"
slug: 06-linux-package-management
published: 2026-05-20
description: "Linux 软件包管理笔记，记录二进制包、源码包、RPM、YUM 等安装与维护方式。"
tags: ["Linux", "软件包", "YUM", "RPM"]
category: Linux
draft: false
---
## 一、软件包的分类

Linux 中的软件包主要分为两大类：**二进制包** 和 **源码包**。不同类型的包安装方式和管理工具有所不同。

| 类型                                 | 特点                               | 安装方式                              | 典型格式             |
| ------------------------------------ | ---------------------------------- | ------------------------------------- | -------------------- |
| **二进制包（Binary Package）**       | 预先编译好的程序，直接可运行       | 使用 RPM/YUM 管理                     | `.rpm`               |
| **源码包（Source Code Package）**    | 源代码文件（如 C/C++），需自行编译 | `./configure && make && make install` | `.tar.gz`, `.tar.xz` |
| **脚本包（Script-based Installer）** | 封装了安装逻辑的 Shell/Python 脚本 | 直接运行脚本完成安装                  | `.sh`, 自定义 bin 包 |

### 🔍 对比总结

| 方面     | RPM/YUM 包         | 源码包                 | 脚本包         |
| -------- | ------------------ | ---------------------- | -------------- |
| 安装速度 | 快（无需编译）     | 慢（需编译）           | 一般           |
| 系统集成 | 强（记录到数据库） | 弱（不被包管理器追踪） | 弱或中等       |
| 升级卸载 | 支持良好           | 手动处理困难           | 取决于脚本设计 |
| 定制性   | 低（固定配置）     | 高（可选功能）         | 中等           |
| 安全性   | 高（签名验证）     | 依赖用户判断           | 依赖来源可信度 |

> ✅ 推荐原则：
> - 日常使用优先选择 **YUM/RPM**
> - 特殊需求（最新版、定制模块）考虑 **源码包**
> - 第三方工具（如 Docker CE、Node.js）常用 **脚本包 + YUM 混合**

---

## 二、RPM 包管理（底层命令）

`rpm` 是 Red Hat 系列系统中的底层包管理工具，用于直接安装、查询、卸载 `.rpm` 格式的二进制包。

> ⚠️ 不推荐跳过依赖直接使用 RPM 安装大型软件！

### 🛠️ 常用 RPM 命令语法

```bash
rpm [选项] [包文件或包名]
```

### ✅ 主要功能与示例

| 功能                   | 命令示例                    | 说明                                     |
| ---------------------- | --------------------------- | ---------------------------------------- |
| 安装软件包             | `sudo rpm -ivh package.rpm` | `-i`: 安装, `-v`: 显示过程, `-h`: 进度条 |
| 升级软件包             | `sudo rpm -Uvh package.rpm` | `-U`: 升级，若未安装则自动安装           |
| 删除软件包             | `sudo rpm -e package_name`  | 注意不是文件名，而是包名（如 httpd）     |
| 查询是否已安装         | `rpm -q package_name`       | `query`                                  |
| 查询所有已安装包       | `rpm -qa \| grep keyword`   | `-qa` = query all                        |
| 查询某个文件属于哪个包 | `rpm -qf /path/to/file`     | useful for troubleshooting               |
| 查询包的详细信息       | `rpm -qi package_name`      | 查看版本、描述、安装时间等               |
| 查询包包含哪些文件     | `rpm -ql package_name`      | list files                               |
| 查询未安装包的信息     | `rpm -qpi package.rpm`      | 查看 `.rpm` 文件元数据                   |
| 查询未安装包含文件     | `rpm -qpl package.rpm`      |                                          |

> 示例：
```bash
# 安装一个本地 RPM 包
sudo rpm -ivh nginx-1.20.1-1.el7.ngx.x86_64.rpm

# 查询是否安装了 openssl
rpm -q openssl

# 查看 apache 安装了哪些文件
rpm -ql httpd

# 查看 /bin/ls 属于哪个包
rpm -qf /bin/ls
```

### ⚠️ RPM 的局限性

- ❌ **不自动解决依赖关系**
  - 若缺少依赖库，会报错中断。
- ❌ **容易造成“依赖地狱”**
  - 手动下载多个 RPM 文件逐个安装非常繁琐。

> 🔧 解决方案：使用更高层的包管理器 —— **YUM 或 DNF**

---

## 三、YUM/DNF 在线包管理（推荐方式）

`YUM`（Yellowdog Updater Modified）是基于 RPM 的**高级包管理器**，能自动从网络仓库下载软件及其依赖并安装。

> 💡 CentOS 7/8 使用 `yum`，RHEL 8+ / Rocky Linux 9+ 推荐使用 `dnf`（更现代，性能更好）

### ✅ 常用 YUM 命令

| 功能                          | 命令示例                            |
| ----------------------------- | ----------------------------------- |
| 搜索软件包                    | `yum search nginx`                  |
| 查看软件包信息                | `yum info httpd`                    |
| 安装软件包                    | `sudo yum install httpd`            |
| 安装本地 RPM 包（自动补依赖） | `sudo yum localinstall package.rpm` |
| 升级单个软件                  | `sudo yum update httpd`             |
| 升级所有可更新软件            | `sudo yum update`                   |
| 删除软件包                    | `sudo yum remove httpd`             |
| 列出已安装包                  | `yum list installed`                |
| 列出可用包                    | `yum list available`                |
| 查看软件包来自哪个仓库        | `yum info vsftpd`                   |
| 清理缓存                      | `sudo yum clean all`                |
| 重建元数据缓存                | `sudo yum makecache`                |

> 示例：
```bash
# 安装 Web 服务器
sudo yum install httpd

# 安装开发工具组
sudo yum groupinstall "Development Tools"

# 查找提供特定命令的包
yum provides /usr/sbin/semanage
```

### 🌐 1. 网络 YUM 源（常用在线仓库）

网络 YUM 源是从互联网上的官方或镜像站点下载元数据和 RPM 包的软件仓库。系统默认已经配置了基础仓库。

#### 默认启用仓库（以 CentOS 7/8 为例）

| 仓库名称     | 说明                              |
| :----------- | :-------------------------------- |
| `base`       | 主要操作系统组件                  |
| `updates`    | 安全更新和补丁                    |
| `extras`     | 额外实用工具（如 `epel-release`） |
| `centosplus` | 增强功能包（谨慎启用）            |

查看当前启用的仓库：

```bash
yum repolist enabled
```

查看所有仓库（含未启用）：

```bash
yum repolist all
```

#### 更换为国内高速镜像源（提升速度）

以更换为阿里云镜像为例：

```bash
# 备份原有 repo 文件
sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# 下载阿里云 repo 配置
sudo curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# 清理缓存并重建
sudo yum clean all
sudo yum makecache
```

> ✅ 国内推荐镜像站：
>
> - 阿里云：https://mirrors.aliyun.com
> - 华为云：https://mirrors.huaweicloud.com
> - 清华大学开源镜像站：https://mirrors.tuna.tsinghua.edu.cn

------

### 💿 2. 光盘 YUM 源（本地源·适用于断网环境）

当服务器无法联网时，可使用系统安装光盘（ISO）作为本地 YUM 源进行软件安装。

#### 步骤一：挂载光盘镜像

将 CentOS 安装 ISO 文件挂载到目录（物理机插入光盘同理）：

```bash
sudo mkdir /mnt/cdrom
sudo mount /dev/cdrom /mnt/cdrom
# 或挂载 ISO 文件
sudo mount -o loop CentOS-7-x86_64-DVD-2009.iso /mnt/cdrom
```

验证是否成功：

```bash
ls /mnt/cdrom/Packages | head -5
```

#### 步骤二：创建本地 repo 配置文件

```bash
sudo vi /etc/yum.repos.d/local-cd.repo
```

输入以下内容：

```ini
[local-cd]

name=Local CDROM Repository

baseurl=file:///mnt/cdrom

enabled=1

gpgcheck=1

gpgkey=file:///mnt/cdrom/RPM-GPG-KEY-CentOS-7
```

> ⚠️ 注意路径是 `file:///`（三个斜杠），且目录需有执行权限

#### 步骤三：加载本地源

```bash
sudo yum clean all

sudo yum makecache
```

现在就可以在无网络环境下安装光盘中的软件包了：

```bash
sudo yum install httpd
```

#### 🔁 卸载与重新挂载（可选）

使用完后取消挂载：

```bash
sudo umount /mnt/cdrom
```

下次需要时再重新挂载即可。

> 💡 提示：可用于灾备恢复、内网批量部署等场景

------

### 📦 3. YUM 包离线下载（用于隔离网络之间的迁移安装）

有些情况下，目标服务器不能联网，但你可以从一台能上网的机器上 **提前下载 RPM 包及其依赖**，然后通过 U 盘、内网传输等方式拷贝过去安装。

#### 方法一：仅下载不安装（`--downloadonly`）

在能联网的同版本系统中执行：

```bash
# 创建下载目录
mkdir ~/offline-packages && cd ~/offline-packages
# 下载 nginx 及其全部依赖，但不安装
sudo yum install nginx --downloadonly --downloaddir=./
```

完成后你会得到多个 `.rpm` 文件，例如：

```
nginx-1.20.1-1.el7.ngx.x86_64.rpm

openssl-libs-1.0.2k-21.el7.x86_64.rpm

pcre-8.32-17.el7.x86_64.rpm

zlib-1.2.7-18.el7.x86_64.rpm
```

> ⚠️ 注意：必须确保目标系统架构（x86_64/arm64）、系统版本一致！

#### 方法二：使用 `yumdownloader`（更灵活）

安装工具（若未自带）：

```bash
sudo yum install yum-utils
```

下载指定包及依赖：

```bash
# 下载主包
yumdownloader nginx
# 下载所有依赖（不含已安装的）
yumdownloader --resolve nginx
# 同时下载并保存到指定目录
yumdownloader --resolve --destdir=/root/offline-pkgs httpd
```

#### 在目标主机上安装离线包

将整个目录拷贝至目标服务器后，使用 `yum localinstall` 自动解决依赖顺序：

```bash
# 推荐方式（YUM 处理依赖）
sudo yum localinstall *.rpm

# 或使用 rpm（需手动排序，不推荐）
sudo rpm -ivh *.rpm
```

> ✅ 优势：`yum localinstall` 能自动识别依赖关系，比直接 `rpm -ivh` 更安全可靠！

#### 高级技巧：打包成自定义本地仓库（适用于多台服务器）

如果你有多个离线服务器，可以将这些 RPM 包构建成一个私有小型 YUM 仓库。

##### 步骤如下：

```bash
# 1. 把所有 RPM 放入目录
mkdir /repo/offline-apps
cp *.rpm /repo/offline-apps/

# 2. 安装 createrepo 工具
sudo yum install createrepo

# 3. 生成元数据
createrepo /repo/offline-apps/


# 4. 创建 repo 配置
cat > /etc/yum.repos.d/offline.repo <<EOF
[offline]
name=Offline Internal Repository
baseurl=file:///repo/offline-apps
enabled=1
gpgcheck=0
EOF

# 5. 构建缓存
yum clean all
yum makecache
```

之后就可以像使用普通 YUM 源一样安装：

```bash
yum install nginx
```

> 🏗️ 应用场景：企业内网统一软件源、安全区批量部署、CI/CD 离线构建节点

------

### 🧩 补充：YUM 常用高级选项

| 命令                              | 作用                            |
| :-------------------------------- | :------------------------------ |
| `yum history`                     | 查看安装/卸载历史记录           |
| `yum history undo N`              | 回滚第 N 次操作（非常有用！）   |
| `yum --assumeyes install package` | 自动确认安装（脚本中常用 `-y`） |
| `yum --skip-broken check-update`  | 检查更新时跳过损坏仓库          |
| `yum shell`                       | 进入交互式 YUM 命令行，批量操作 |

示例：撤销一次误删操作

```bash
yum history list    # 找到操作 ID

yum history undo 15 # 撤销第 15 条操作
```

------

### 🔐 GPG 密钥验证机制说明

YUM 默认开启 `gpgcheck=1`，用于验证软件包完整性与来源合法。

导入官方 GPG 公钥（通常自动完成）：

```bash
sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

查看已导入密钥：

```bash
rpm -q gpg-pubkey
```

> ⚠️ 生产环境中不要随意关闭 `gpgcheck`！

------

✅ 总结对比表：三种 YUM 源适用场景

| 类型                | 是否需要网络         | 适合场景           | 维护难度 |
| :------------------ | :------------------- | :----------------- | :------- |
| 网络 YUM 源         | ✔️ 需要               | 日常维护、频繁更新 | 低       |
| 光盘 YUM 源         | ❌ 不需要（本地介质） | 应急安装、无网环境 | 中       |
| 离线包 + Local Repo | ❌ 目标机无需联网     | 内网部署、安全区   | 中高     |









## 四、源码包管理（手动编译安装）

当你需要：
- 使用软件的最新版本
- 自定义编译选项（如启用 SSL、关闭调试）
- 某些软件没有提供 RPM 包

就需要使用 **源码包安装**。

### 📦 获取方式

通常为压缩包形式：
```text
httpd-2.4.57.tar.gz
nginx-1.25.3.tar.gz
openssl-3.0.10.tar.gz
```

下载地址：[https://httpd.apache.org](https://httpd.apache.org), [https://nginx.org](https://nginx.org) 等官网

### 🧰 编译安装三步曲

```bash
./configure    # 检查系统环境、配置编译参数
make           # 编译源码 → 生成二进制文件
sudo make install  # 安装到系统目录（默认 /usr/local/）
```

### 参数详解

| 步骤                                 | 详细说明                     |
| ------------------------------------ | ---------------------------- |
| `./configure`                        |                              |
| &nbsp;&nbsp;–prefix=/usr/local/nginx | 设置安装路径                 |
| &nbsp;&nbsp;–enable-ssl              | 启用 HTTPS 支持              |
| &nbsp;&nbsp;–with-http_v2_module     | 启用 HTTP/2                  |
| `make`                               | 根据 Makefile 编译所有源文件 |
| `make install`                       | 将编译好的文件复制到目标目录 |

> 示例：编译安装 Nginx
```bash
tar -zxvf nginx-1.25.3.tar.gz
cd nginx-1.25.3
./configure --prefix=/usr/local/nginx --with-http_ssl_module
make
sudo make install
```

### ⚠️ 源码包注意事项

- ❗ 不被 RPM/YUM 记录，无法通过 `rpm -q` 查询
- ❗ 升级和卸载需手动操作（删除整个目录）
- ❗ 依赖库需提前安装（如 gcc, openssl-devel）

> ✅ 建议做法：
- 写安装日志文档
- 将源码包和脚本集中存放
- 使用 `checkinstall` 工具生成 RPM 包（可选）

---

## 五、脚本包管理（自动化安装包）

有些软件厂商为了简化安装流程，提供一个“一键安装”脚本（常见为 `.sh`），内部封装了：
- 仓库添加
- GPG 密钥导入
- 包安装命令
- 服务启动等操作

### 🎯 常见脚本包示例

| 软件               | 安装命令                                                     |
| ------------------ | ------------------------------------------------------------ |
| Docker CE          | `curl -fsSL https://get.docker.com \| sh`                    |
| NodeSource Node.js | `curl -fsSL https://deb.nodesource.com/setup_18.x \| sudo bash -` |
| GitLab Runner      | `wget -O - https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh \| sudo bash` |

### 🔍 脚本包的工作原理（以 Docker 为例）

1. 下载官方安装脚本
2. 脚本自动检测系统版本（CentOS/Ubuntu）
3. 添加对应的 YUM/Apt 仓库
4. 导入 GPG 公钥保证包完整性
5. 安装 `docker-ce` 软件包
6. 启动 `docker` 服务并设置开机自启

### ⚠️ 安全提醒：谨慎运行未知脚本！

```bash
# ❌ 危险行为：盲目执行远程脚本
curl http://unknown-site.com/install.sh | sh

# ✅ 安全做法：
# 1. 先查看脚本内容
curl -sSL https://get.docker.com -o install-docker.sh
cat install-docker.sh   # 审核后再运行
sh install-docker.sh
```

---

## 📌 实用场景示例

### ✅ 场景 1：从零搭建 LAMP 环境（YUM 方式）

```bash
# 1. 安装 Apache
sudo yum install httpd

# 2. 安装 MariaDB（MySQL 替代）
sudo yum install mariadb-server

# 3. 安装 PHP
sudo yum install php php-mysql

# 4. 启动服务
sudo systemctl start httpd mariadb
sudo systemctl enable httpd mariadb
```

---

### ✅ 场景 2：手动编译安装高版本 GCC（源码包）

```bash
# 安装依赖
sudo yum groupinstall "Development Tools"
sudo yum install gmp-devel mpfr-devel libmpc-devel

# 解压并编译
tar -jxf gcc-12.2.0.tar.bz2
cd gcc-12.2.0
./contrib/download_prerequisites
mkdir build && cd build
../configure --enable-languages=c,c++ --disable-multilib
make -j$(nproc)
sudo make install
```

---

### ✅ 场景 3：安全安装 Docker CE（脚本包 + 审核）

```bash
# 下载脚本
curl -fsSL https://get.docker.com -o get-docker.sh

# 审查内容
less get-docker.sh

# 确认无恶意后执行
sh get-docker.sh

# 将当前用户加入 docker 组（免 sudo）
sudo usermod -aG docker $USER
```

---

### ✅ 场景 4：只查看某个包会安装哪些文件（YUM + RPM）

```bash
# 方法一：yum deplist（查看依赖）
yum deplist httpd

# 方法二：下载但不安装（查看内容）
yum install httpd --downloadonly --downloaddir=./pkgs
rpm -qlp ./pkgs/httpd-*.rpm
```

---

## 📘 学习建议

1. ✅ **日常优先使用 YUM/DNF**
   - 安全、便捷、自动处理依赖。

2. 🔍 **理解 RPM 数据库的作用**
   - 所有通过 RPM/YUM 安装的包都记录在 `/var/lib/rpm` 中。

3. 🧪 **实验环境练习源码编译**
   - 在虚拟机中尝试编译 Nginx、SQLite 等简单项目。

4. 🔐 **绝不随意执行远程脚本**
   - 一定要先审查 `install.sh` 内容再运行。

5. 📦 **了解主流第三方仓库**
   - 如 EPEL、Remi、Docker 官方源，提升软件获取效率。

6. 🤖 **尝试编写自己的安装脚本**
   - 自动化部署常用工具链。

