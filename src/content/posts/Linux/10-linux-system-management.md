---
title: "10.Linux 系统管理"
slug: 10-linux-system-management
published: 2026-05-20
description: "Linux 系统管理笔记，记录启动流程、系统服务、日志与常用管理操作。"
tags: ["Linux", "系统管理", "运维"]
category: Linux
draft: false
---
## 一、系统启动流程详解

### 🔁 启动阶段划分（以 systemd 为主）

| 阶段                     | 描述                     |
| ------------------------ | ------------------------ |
| 1. BIOS/UEFI 自检        | 初始化硬件，选择启动设备 |
| 2. GRUB2 引导菜单        | 加载内核和 initramfs     |
| 3. 内核实例化            | 挂载根文件系统           |
| 4. systemd 启动（PID=1） | 管理所有用户空间服务     |
| 5. 进入默认 target       | 如 `multi-user.target`   |

### 🧩 核心机制说明

- **GRUB2**：位于 `/boot/grub2/grub.cfg`，自动生成
- **initramfs**：临时根文件系统，用于加载驱动以便挂载真实根分区
- **systemd unit 文件**：服务存储在 `/usr/lib/systemd/system/*.service`

### 📌 常用命令

```bash
# 查看上次启动时间
who -b

# 查看启动耗时
systemd-analyze

# 查看各服务启动时间
systemd-analyze blame

# 分析关键阶段耗时
systemd-analyze critical-chain

# 设置默认启动目标（图形 or 命令行）
sudo systemctl set-default multi-user.target
sudo systemctl set-default graphical.target
```

### ⚠️ 救援模式使用场景

**忘记 root 密码？进入单用户模式重置密码**

**步骤：**
1. 开机进入 GRUB 菜单，按 `e` 编辑启动项
2. 在 `linux16` 行末尾添加 `rd.break` 或 `init=/bin/bash`
3. 按 `Ctrl+X` 启动
4. 重新挂载 `/sysroot` 为可写：
   ```bash
   mount -o remount,rw /sysroot
   chroot /sysroot
   passwd root
   touch /.autorelabel    # SELinux 用户需执行此步
   exit; reboot
   ```

---

## 二、系统资源查看

### 📈 1. CPU 使用情况

```bash
lscpu                # 显示 CPU 架构信息
top                  # 动态查看 CPU 占用
htop                 # 更直观（需安装：yum install htop）
mpstat                # 来自 sysstat 包，显示多核统计
```

示例：
```bash
mpstat -P ALL 1       # 每秒刷新一次，查看所有核心
```

### 💾 2. 内存使用情况

```bash
free -h               # 人类可读内存状态
cat /proc/meminfo     # 详细内存信息
vmstat 1              # 监控虚拟内存、进程、CPU（每秒刷新）
```

> 📝 参数解释：
- `free`: 可用内存
- `buffers/cache`: 缓冲区与缓存（Linux 会利用空闲内存做缓存，仍可用）

### 💿 3. 磁盘与 I/O 性能

```bash
df -h                 # 查看挂载点使用率
du -sh /path/to/dir   # 查目录大小
iostat -x 1           # 来自 sysstat，查看磁盘 I/O 延迟、利用率
iotop                # 类似 top 的 I/O 监视工具（需要权限）
```

### 🔀 4. 进程管理

```bash
ps aux | grep sshd     # 查指定进程
ps aux --sort=-%mem    # 按内存降序排列
kill 1234              # 终止 PID=1234 的进程
killall httpd          # 终止所有名为 httpd 的进程
pkill -u alice         # 终止 alice 用户运行的所有进程
```

---

## 三、远程登录管理（SSH）

### 🔐 SSH 安全远程访问协议（Secure Shell）

#### 基本使用

```bash
ssh user@192.168.1.100        # 登录远程主机
scp file.txt user@host:/tmp   # 安全复制文件
sftp user@host               # 安全 FTP 交互式传输
```

#### 提高安全性的最佳实践

| 措施                       | 方法                                                  |
| -------------------------- | ----------------------------------------------------- |
| ❌ 禁用 root 直接登录       | 修改 `/etc/ssh/sshd_config`：<br>`PermitRootLogin no` |
| ✅ 使用密钥认证代替密码     | 免输密码 + 更安全                                     |
| 🔁 修改默认端口（可选）     | `Port 2222` 防止机器人扫描                            |
| ⚠️ 禁止密码登录（仅限密钥） | `PasswordAuthentication no`                           |
| 👤 限制允许登录的用户       | `AllowUsers alice bob@192.168.*`                      |

应用更改后重启服务：
```bash
sudo systemctl restart sshd
```

#### SSH 密钥对生成与部署

```bash
# 本地生成密钥（通常 ~/.ssh/id_rsa）
ssh-keygen -t rsa -b 2048

# 将公钥上传到服务器
ssh-copy-id user@remote_host

# 测试免密登录
ssh user@remote_host
```

---

## 四、网络配置与网络测试

### 🌐 1. 网络接口管理

```bash
ip addr show           # 替代旧 ifconfig
ip route show          # 查看路由表
nmcli device status    # NetworkManager 查看网卡状态
nmtui                  # 文本界面配置网络（交互式）
```

#### 静态 IP 配置示例（CentOS 使用 nmcli）

```bash
sudo nmcli con mod "System eth0" \
ipv4.addresses 192.168.1.100/24 \
ipv4.gateway 192.168.1.1 \
ipv4.dns "8.8.8.8,8.8.4.4" \
ipv4.method manual

sudo nmcli con up "System eth0"
```

### 🧪 2. 网络连通性测试

| 工具                        | 用途                                   |
| --------------------------- | -------------------------------------- |
| `ping`                      | 测试网络是否可达                       |
| `traceroute` 或 `tracepath` | 跟踪数据包路径                         |
| `mtr`                       | 结合 ping 和 traceroute 的实时诊断工具 |
| `ss -tuln`                  | 查看开放端口（替代 netstat）           |
| `telnet ip port`            | 测试某端口是否开放                     |
| `nc -zv ip port`            | 更专业的端口探测工具（netcat）         |

示例：
```bash
ss -tulnp | grep :22      # 查看哪个程序监听 22 端口
nc -zv 192.168.1.100 80   # 测试 Web 是否响应
```

---

## 五、网络安全基础

### 🛡️ 1. 防火墙管理（firewalld）

```bash
systemctl status firewalld    # 查看状态
firewall-cmd --state          # 检查运行中

firewall-cmd --list-all       # 查看当前区域规则

# 开放服务或端口
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload         # 重新加载配置

# 删除规则
firewall-cmd --permanent --remove-service=http
```

常见服务名：
- `ssh`, `http`, `https`, `mysql`, `dhcpv6-client`

> 提示：生产环境建议关闭不必要的端口和服务。

---

### 🔒 2. SELinux 安全增强型 Linux

SELinux 是强制访问控制（MAC）系统，防止越权操作。

#### 状态查看与设置

```bash
sestatus                          # 查看 SELinux 状态
getenforce                        # 输出：Enforcing / Permissive / Disabled

# 临时切换模式
setenforce 0                      # 关闭 enforce（只记录不阻止）
setenforce 1                      # 重新启用

# 永久修改：编辑 /etc/selinux/config
# SELINUX=enforcing     # 正常模式
# SELINUX=permissive    # 宽松模式（调试用）
# SELINUX=disabled      # 彻底禁用（不推荐）!
```

#### 实战技巧

遇到服务无法启动但权限正常 → 很可能因 SELinux 报错！

解决方案：
```bash
# 查看拒绝日志
ausearch -m avc -ts recent
# 或使用更友好的
sealert -a /var/log/audit/audit.log
```

批量修复上下文（如网站目录被拷贝错误）：
```bash
restorecon -Rv /var/www/html
```

---

## 六、定时任务管理（crond & at）

### ⏰ 1. crontab —— 周期性任务调度器

格式：`分 时 日 月 星期 命令`

```text
*   *   *   *   *   command
│   │   │   │   └─── 星期 (0–6, 0=周日)
│   │   │   └────── 月 (1–12)
│   │   └────────── 日 (1–31)
│   └────────────── 时 (0–23)
└────────────────── 分 (0–59)
```

#### 示例

```bash
# 每天凌晨 2:30 备份数据库
30 2 * * * /usr/bin/mysqldump -u root -p'pass' mydb > /backup/db_$(date +\%F).sql

# 每 10 分钟检查 nginx 是否运行
*/10 * * * * pgrep nginx || systemctl start nginx
```

#### 用户级 crontab 操作

```bash
crontab -e          # 编辑自己的计划任务
crontab -l          # 列出当前任务
crontab -r          # 删除所有任务（慎用！）
```

> 注意：脚本尽量使用完整路径，避免因环境变量缺失失败！

---

### 📅 2. at —— 一次性延迟执行

适合执行“一段时间后的某个命令”。

```bash
# 安装（若未自带）
yum install at

# 示例：30分钟后关机
echo "shutdown -h now" | at now + 30 minutes

# 查看队列
atq

# 删除任务
atrm 1
```

---

## 七、时间同步服务（NTP / chrony）

准确的时间对于日志审计、证书验证、集群协调至关重要。

### ✅ 推荐工具：chrony（替代老旧 ntpd）

```bash
yum install chrony
systemctl enable --now chronyd
```

#### 常用命令

```bash
chronyc tracking                    # 查看时间同步状态
chronyc sources -v                 # 查看时间源
timedatectl                        # 查看系统时间和时区
timedatectl set-timezone Asia/Shanghai   # 设置中国时区
timedatectl set-ntp true           # 启用自动时间同步
```

#### 查看是否已同步

```bash
timedatectl
# 输出应包含:
# System clock synchronized: yes
# NTP service: active
```

---

## 八、系统备份与恢复策略

### 📦 备份方式分类

| 类型     | 特点                 | 工具示例              |
| -------- | -------------------- | --------------------- |
| 完全备份 | 所有数据全部备份     | `tar`, `rsync`        |
| 增量备份 | 仅上次以来变化的部分 | `rsync --link-dest`   |
| 差异备份 | 相对于完全备份的变化 | `rsync` + 时间戳      |
| 快照备份 | LVM / Btrfs 快照     | `lvcreate --snapshot` |

### ✅ 推荐组合方案：`tar + cron` 实现每日自动备份

```bash
# 脚本：/usr/local/bin/backup.sh
#!/bin/bash
DATE=$(date +%F)
BACKUP_DIR="/backup"
TARGET_FILE="$BACKUP_DIR/system_$DATE.tar.gz"

mkdir -p $BACKUP_DIR

tar --exclude=/backup --exclude=/proc --exclude=/sys --exclude=/dev \
    -czf $TARGET_FILE /etc /home /var/www

# 删除7天前的旧备份
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete
```

加入定时任务：
```bash
crontab -e
# 添加：
0 2 * * * /usr/local/bin/backup.sh
```

### 🔄 恢复操作

```bash
tar -xzvf system_2025-04-01.tar.gz -C /
```

> 💡 建议把备份文件定期传送到异地服务器或云存储（如 rsync + SSH）

---

## 九、日志服务管理（rsyslog + journalctl）

### 📜 1. syslog 标准日志系统（rsyslog）

负责接收来自应用程序的消息，并分类记录。

主要配置文件：
- `/etc/rsyslog.conf`
- `/etc/rsyslog.d/*.conf`

日志存放位置：
- `/var/log/messages` —— 系统通用日志（CentOS）
- `/var/log/secure` —— 认证相关（SSH 登录等）
- `/var/log/cron` —— 定时任务日志
- `/var/log/maillog` —— 邮件服务日志

查看最新日志：
```bash
tail -f /var/log/messages
journalctl -f                   # 实时查看 systemd 日志
```

### 🔎 2. journalctl 查询 systemd 日志（强力推荐）

```bash
journalctl                            # 查看全部日志
journalctl -u sshd.service          # 查某个服务的日志
journalctl --since "today"           # 今天日志
journalctl -u nginx --since "1 hour ago"
journalctl -p err..emerg             # 错误及以上级别
journalctl _PID=1234                 # 查特定进程 ID
journalctl --disk-usage              # 查日志占用空间
```

#### 清理过大的日志文件

```bash
# 保留最近一周
journalctl --vacuum-time=7d

# 或限制总大小
journalctl --vacuum-size=100M
```

---

## 十、性能指标监控（持续观察）

### 🎯 推荐工具清单

| 工具           | 用途                           |
| -------------- | ------------------------------ |
| `top` / `htop` | CPU & 内存实时监控             |
| `iostat`       | 磁盘 I/O 使用率                |
| `iotop`        | 哪个进程占用了磁盘 IO？        |
| `vmstat`       | 系统整体健康状况（换页、中断） |
| `nmon`         | 专业监控工具（提供图形化界面） |
| `sar`          | 收集历史统计数据（sysstat 包） |

#### 启用 sar 数据收集

```bash
# 安装
yum install sysstat

# 编辑配置启用采集
sed -i 's/SA1=false/SA1=true/' /etc/sysconfig/sysstat
systemctl enable sysstat

# 查看报告
sar -u       # CPU 使用率
sar -r       # 内存使用
sar -b       # I/O 统计
sar -n DEV   # 网络流量
```

输出示例：
```bash
Linux 4.18.0 (your-host)   04/05/2025      _x86_64_    (2 CPU)

10:00:01 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
10:10:01 AM     all      12.3       0.1       4.5       2.1       0.0      81.0
```

表示：平均 CPU 利用率为 19%，I/O 等待较低，系统较流畅。

---

## 附录 A：常用命令速查表

| 功能           | 命令                        |
| -------------- | --------------------------- |
| 查系统版本     | `cat /etc/os-release`       |
| 查 IP 地址     | `ip a`                      |
| 查开放端口     | `ss -tuln`                  |
| 查磁盘使用     | `df -h`                     |
| 查目录大小     | `du -sh /dir`               |
| 查进程         | `ps aux \| grep name`       |
| 查防火墙状态   | `firewall-cmd --list-all`   |
| 查 SELinux     | `sestatus`                  |
| 查时间是否同步 | `timedatectl`               |
| 查日志         | `journalctl -u serviceName` |
| 创建定时任务   | `crontab -e`                |
| 查启动时间     | `who -b`                    |

---

## 附录 B：真实场景实战案例

### ✅ 案例 1：服务器突然变慢？如何排查？

**步骤流程图：**
```
1. 登录查看 load average → uptime
2. top → 发现哪个进程 CPU 高
3. ps aux → 确认 PID 和归属
4. df -h → 是否磁盘满？
5. iostat -x 1 → 是否磁盘瓶颈？
6. journalctl -xe → 是否有报错？
7. 最终定位是爬虫程序无节制请求。
```

### ✅ 案例 2：新部署 Web 服务无法从外网访问？

1. 内部 `curl http://localhost` → 成功
2. 外部 telnet ip 80 → 失败
3. `ss -tuln | grep :80` → 无监听？→ 服务没启动！
4. 修正后仍有问题 → 检查 firewall-cmd 是否放行 http
5. 再不行 → 检查云平台安全组规则（AWS/Aliyun）

