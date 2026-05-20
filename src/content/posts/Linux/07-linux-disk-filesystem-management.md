---
title: "7.Linux 磁盘与文件系统管理"
slug: 07-linux-disk-filesystem-management
published: 2026-05-20
description: "Linux 磁盘与文件系统管理笔记，记录磁盘分区、挂载、文件系统和常用排查命令。"
tags: ["Linux", "磁盘", "文件系统"]
category: Linux
draft: false
---
## 一、硬盘基础知识

### 1. 常见设备命名规则

| 设备类型      | 命名方式                 | 示例                   |
| ------------- | ------------------------ | ---------------------- |
| IDE/SATA 硬盘 | `/dev/sd[a-z]`           | `/dev/sda`, `/dev/sdb` |
| NVMe SSD      | `/dev/nvme0n1`           | `/dev/nvme0n1p1`       |
| 虚拟机/云盘   | `/dev/vd[a-z]`（如 KVM） | `/dev/vda`             |

> 分区表示：
- `/dev/sda1` → 第一块硬盘的第一个主分区
- `/dev/sda2` → 第二个分区（可能是扩展或逻辑分区）

### 2. 查看当前磁盘信息

```bash
lsblk            # 列出所有块设备（推荐）
fdisk -l         # 查看磁盘分区表详情
sudo parted -l   # 更现代的分区工具输出
```

示例输出：
```text
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0   50G  0 disk 
├─sda1        8:1    0  1G   0 part /boot
└─sda2        8:2    0  49G  0 part /
sr0          11:0    1 1024M 0 rom
```

---

## 二、磁盘分区（MBR vs GPT）

| 特性             | MBR（主引导记录）              | GPT（GUID 分区表） |
| ---------------- | ------------------------------ | ------------------ |
| 最大支持磁盘大小 | 2TB                            | 18EB（理论极大）   |
| 最多主分区数     | 4 个主分区（或 3 主 + 1 扩展） | 128+               |
| 是否支持 UEFI    | 否                             | 是                 |
| 安全性           | 无备份                         | 有分区表备份       |
| 推荐用途         | 旧系统、小磁盘                 | 新系统、大容量硬盘 |

### 使用 `parted` 创建 GPT 分区（大于 2TB 必须用 GPT）

```bash
sudo parted /dev/sdb
(parted) mklabel gpt
(parted) mkpart primary ext4 1MiB 100%
(parted) set 1 lvm on      # 若用于 LVM
(parted) print
(parted) quit
```

### 使用 `fdisk` 管理 MBR 分区（适用于小于 2TB 的磁盘）

```bash
sudo fdisk /dev/sdb

# 常用命令：
m        # 显示帮助
n        # 新建分区
p        # 查看当前分区
d        # 删除分区
w        # 保存退出
q        # 不保存退出
```

---

## 三、文件系统类型（ext4, XFS, swap）

格式化时需选择合适的文件系统：

| 文件系统  | 特点                   | 适用场景                |
| --------- | ---------------------- | ----------------------- |
| **ext4**  | 成熟稳定，兼容性好     | 通用服务器、中小型存储  |
| **XFS**   | 高性能，大文件处理强   | 大型数据库、高 I/O 场景 |
| **swap**  | 交换空间，作为内存补充 | 虚拟内存                |
| **btrfs** | 支持快照、压缩         | 实验性使用（生产慎用）  |

> 💡 查看内核支持的文件系统：
> ```bash
> cat /proc/filesystems
> ```

---

## 四、格式化与挂载（mkfs, mount）

### 1. 格式化分区

⚠️ 警告：此操作会清除原有数据！

```bash
# 创建 ext4 文件系统
sudo mkfs.ext4 /dev/sdb1

# 创建 xfs 文件系统
sudo mkfs.xfs /dev/sdb1

# 强制格式化（跳过确认）
sudo mkfs.xfs -f /dev/sdb1
```

### 2. 临时挂载到目录

```bash
# 创建挂载点
sudo mkdir /data

# 挂载
sudo mount /dev/sdb1 /data

# 查看是否成功
df -h /data
```

> 🔍 卸载：
> ```bash
> sudo umount /data
> ```
> ✅ 注意：必须先退出挂载目录再卸载！

---

## 五、查看磁盘空间使用情况（df, du）

### `df`：查看文件系统整体使用情况

```bash
df -h              # 人类可读格式（KB/MB/GB）
df -T              # 显示文件系统类型
df --total          # 统计总用量
```

常用组合：
```bash
df -h /home        # 查看 /home 分区使用率
```

### `du`：查看目录或文件占用空间

```bash
du -sh /var/log    # 总共大小（summary, human）
du -h --max-depth=1 /home  # 查看 home 下各用户占用
du -a /tmp | sort -hr | head -10  # 找出最大的 10 个文件
```

> ✅ 推荐排查磁盘满的方法：
> ```bash
> df -h                # 找出哪个分区满了
> du -sh /* 2>/dev/null | sort -rh   # 查找根下大目录
> ```

---

## 六、永久挂载（/etc/fstab）

为了让系统重启后仍能自动挂载，需要编辑 `/etc/fstab` 文件。

### 编辑 fstab

```bash
sudo vi /etc/fstab
```

添加一行（以 UUID 推荐）：

```text
UUID=abcd1234-defg-5678...  /data  xfs  defaults  0 0
```

或使用设备名（不推荐，可能变化）：

```text
/dev/sdb1  /data  xfs  defaults  0 0
```

#### 字段说明（共6列）

| 列   | 含义                                                      |
| ---- | --------------------------------------------------------- |
| 1    | 设备标识（UUID 或设备路径）                               |
| 2    | 挂载点                                                    |
| 3    | 文件系统类型                                              |
| 4    | 挂载选项（defaults = rw,suid,dev,exec,auto,nouser,async） |
| 5    | 是否备份（dump），通常为 0                                |
| 6    | 开机检查顺序（root为1，其他为2，非文件系统设为0）         |

> 🔍 查看 UUID：
> ```bash
> blkid /dev/sdb1
> # 输出：UUID="abcd..." TYPE="xfs"
> ```

#### 测试 fstab 正确性
```bash
sudo mount -o remount /data     # 重新挂载测试
sudo mount -a                   # 尝试挂载所有未挂载条目（开机时也会这样执行）
```

---

## 七、交换空间管理（swap）

Swap 是物理内存不足时使用的“虚拟内存”。

### 1. 查看当前 Swap 使用情况

```bash
free -h
swapon --show
cat /proc/swaps
```

### 2. 添加 Swap 分区

#### 方法一：使用新分区

```bash
# 格式化为 swap
sudo mkswap /dev/sdb2

# 启用 swap
sudo swapon /dev/sdb2

# 写入 fstab 实现开机启用
echo '/dev/sdb2 none swap defaults 0 0' | sudo tee -a /etc/fstab
```

#### 方法二：使用 Swap 文件（更灵活）

```bash
# 创建一个 2GB 的空文件
sudo fallocate -l 2G /swapfile

# 设置权限（安全起见）
sudo chmod 600 /swapfile

# 格式化为 swap
sudo mkswap /swapfile

# 启用
sudo swapon /swapfile

# 加入 fstab
echo '/swapfile none swap defaults 0 0' | sudo tee -a /etc/fstab
```

> ✅ 建议 swap 大小：
> - 物理内存 ≤ 2GB → swap = 2×RAM
> - 物理内存 > 2GB → swap = RAM size 或固定 4~8GB

---

## 八、逻辑卷管理 LVM（灵活扩容）

LVM（Logical Volume Manager）允许你动态调整存储空间，是企业级环境中常用的高级磁盘管理方式。

### LVM 核心概念

| 概念   | 英文                 | 说明                                     |
| ------ | -------------------- | ---------------------------------------- |
| 物理卷 | PV (Physical Volume) | 底层磁盘或分区                           |
| 卷组   | VG (Volume Group)    | 多个 PV 组合成的存储池                   |
| 逻辑卷 | LV (Logical Volume)  | 从 VG 中划分出的“虚拟磁盘”，可格式化挂载 |

### 操作流程（三步曲）

#### 1️⃣ 初始化 PV

```bash
sudo pvcreate /dev/sdb1 /dev/sdc1
```

#### 2️⃣ 创建 VG

```bash
sudo vgcreate vg_data /dev/sdb1 /dev/sdc1
```

#### 3️⃣ 创建 LV

```bash
# 创建 10GB 的逻辑卷
sudo lvcreate -L 10G -n lv_web vg_data

# 格式化并挂载
sudo mkfs.xfs /dev/vg_data/lv_web
sudo mkdir /web
sudo mount /dev/vg_data/lv_web /web
```

### 动态扩容演示

#### 扩容 LV（无需卸载）

```bash
# 增加 5GB
sudo lvextend -L +5G /dev/vg_data/lv_web

# 扩展文件系统（XFS 特殊语法）
sudo xfs_growfs /web

# 如果是 ext4，则使用：
# resize2fs /dev/vg_data/lv_web
```

#### 扩容 VG（加入新磁盘）

```bash
pvcreate /dev/sdd1
vgextend vg_data /dev/sdd1
```

> ✅ LVM 优势：
> - 支持在线扩容
> - 支持快照备份
> - 磁盘资源整合，提升利用率

---

## 九、软RAID基础（mdadm）

RAID 可提高磁盘性能或冗余性。Linux 提供 `mdadm` 工具实现软 RAID。

### 常见级别对比

| RAID    | 描述                     | 空间利用率 | 安全性                     |
| ------- | ------------------------ | ---------- | -------------------------- |
| RAID 0  | 条带化，速度最快         | 100%       | ❌ 任意一块坏即全损         |
| RAID 1  | 镜像，两块盘完全复制     | 50%        | ✔️ 容忍一块损坏             |
| RAID 5  | 分布奇偶校验，至少三块盘 | (n-1)/n    | ✔️ 容忍一块损坏             |
| RAID 10 | 先镜像再条带             | 50%        | ✔️ 可容忍多个故障（有条件） |

### 创建 RAID 1 示例

```bash
# 安装工具
sudo yum install mdadm

# 使用两个盘创建镜像
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# 格式化
sudo mkfs.xfs /dev/md0

# 挂载
sudo mkdir /raid1
sudo mount /dev/md0 /raid1

# 保存配置（防止重启失效）
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm.conf
```

查看状态：
```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

---

## 十、磁盘配额（Quota）管理

限制用户或组对磁盘空间的使用，常用于多用户环境。

### 启用步骤

#### 1. 修改 `/etc/fstab` 启用 usrquota,grpquota

```text
/dev/sda2  /home  ext4  defaults,usrquota,grpquota  0 0
```

然后重新挂载：
```bash
sudo mount -o remount /home
```

#### 2. 生成 quota 数据库

```bash
sudo quotacheck -cug /home
# -c: 创建, -u: 用户配额, -g: 组配额
```

#### 3. 启动配额

```bash
sudo quotaon /home
```

#### 4. 设置用户配额

```bash
sudo edquota -u alice
```

进入编辑界面设置软硬限制（单位 KB）：

```text
Disk quotas for user alice (uid 1001):
  Filesystem  blocks   soft   hard   inodes   soft   hard
  /dev/sda2   80000    100M   120M   0        0      0
```

> 表示：最多使用 100MB 软限，120MB 硬限（不可超）

#### 5. 查看配额使用情况

```bash
quota -u alice
repquota /home    # 查看所有用户配额汇总
```

---

## 📌 实用场景示例

### ✅ 场景 1：新增一块硬盘并挂载为 /data（完整流程）

```bash
# 1. 查看新硬盘
lsblk

# 2. 分区（假设为 /dev/sdb）
sudo fdisk /dev/sdb   # 创建 sdb1

# 3. 格式化
sudo mkfs.xfs /dev/sdb1

# 4. 创建挂载点
sudo mkdir /data

# 5. 临时挂载
sudo mount /dev/sdb1 /data

# 6. 获取 UUID
blkid /dev/sdb1

# 7. 写入 fstab
echo "UUID=xxx /data xfs defaults 0 0" | sudo tee -a /etc/fstab
```

---

### ✅ 场景 2：给 Web 目录动态扩容（LVM）

```bash
# 当前 LV 不够用了
df -h /var/www

# 扩容 LV（假设有剩余空间）
sudo lvextend -L +5G /dev/vg0/lv_www
sudo xfs_growfs /var/www
```

---

### ✅ 场景 3：防止某个用户占满磁盘（Quota）

```bash
# 启用 /home 配额机制
# （前提是已在 fstab 中添加 usrquota）
sudo quotacheck -cum /home
sudo quotaon /home

# 限制 bob 最多使用 500MB
sudo edquota -u bob
# 设置 hard limit = 512000 KB
```

---

### ✅ 场景 4：应急恢复 swap 空间不足

```bash
# 临时增加 swap 文件
sudo fallocate -l 1G /tmp/swap.tmp
sudo chmod 600 /tmp/swap.tmp
sudo mkswap /tmp/swap.tmp
sudo swapon /tmp/swap.tmp

# 解决燃眉之急，后续应优化程序或加内存
```

---

## 📘 学习建议

1. ✅ **先在 VMware/VirtualBox 中练习**
   - 使用虚拟磁盘模拟新增、删除、挂载等操作。

2. 🔒 **养成查看日志的习惯**
   - `dmesg | tail`
   - `journalctl -xe`

3. 📊 **掌握日常巡检命令**
   - `df -h`, `du -sh *`, `lsblk`, `free -h`

4. 🔄 **重要操作前做快照或备份**
   - 特别是分区、格式化、LVM 操作。

5. 🤖 **自动化脚本处理重复任务**
   - 如磁盘监控、自动报警。

6. 📚 **延伸阅读**
   - `man lvm`, `man mdadm`, `man fstab`
   - 生产环境建议了解 SAN/NAS、iSCSI、Ceph 等网络存储技术

