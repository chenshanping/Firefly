---
title: "1.Linux 学习思维导图"
slug: 01-linux-learning-mindmap
published: 2026-05-20
description: "Linux 学习知识体系思维导图，整理文件目录、用户权限、进程服务和网络等核心主题。"
tags: ["Linux", "学习笔记", "思维导图"]
category: Linux
draft: false
---
# Linux 学习知识体系

## 一、基础概念
### 1.1 文件与目录
- 文件颜色含义
  - 蓝色 → 目录
  - 绿色 → 可执行文件
  - 红色 → 压缩包/归档
  - 白色 → 普通文件
  - 浅蓝色 → 符号链接
- 文件类型标识（ls -l首个字符）
  - `-` 普通文件  `d` 目录  `l` 符号链接
  - `b` 块设备  `c` 字符设备  `p` 管道  `s` 套接字

### 1.2 系统目录结构
- /boot - 内核、引导程序
- /etc - 系统配置文件
- /lib - 动态链接库
- /sys - sysfs虚拟文件系统
- /bin - 基本命令  /sbin - 系统管理命令
- /dev - 设备文件  /media - 可移动设备  /mnt - 临时挂载点
- /tmp - 临时文件  /run - 运行时数据
- /root - root家目录  /home - 普通用户家目录
- /usr - 用户级应用资源(/usr/bin /usr/sbin)
- /var - 变动数据(/var/log日志 /var/cache缓存 /var/spool队列)
- /proc - 虚拟文件系统  /opt - 第三方软件  /srv - 服务数据

### 1.3 Shell与Bash
- 常见Shell: sh bash zsh dash
- 快捷键: Ctrl+C终止 Ctrl+Z挂起 Ctrl+D退出 Ctrl+R搜索 Tab补全

### 1.4 通配符与元字符
- `*` 任意长度 `?` 单个字符 `[abc]` 括号内 `{str1,str2}` 多字符串

## 二、常用命令
### 2.1 文件与目录
- ls列出 cd切换 pwd路径 mkdir创建 rm删除 cp复制 mv移动 touch创建

### 2.2 查看与搜索
- cat输出 less分页 head/tail首尾 grep搜索 find查找 which/whereis定位

### 2.3 权限用户网络
- chmod修改权限 chown所有者 chgrp所属组
- whoami/id/su/sudo useradd/userdel groupadd/groupdel
- ip地址 ping测试 ss端口 ssh/scp远程 curl/wget下载

### 2.4 进程磁盘压缩
- ps快照 top/htop监控 kill终止 pkill/killall杀进程 bg/fg切换
- df磁盘 du目录 mount挂载 fdisk分区 mkfs格式化
- tar打包 gzip压缩 zip压缩

### 2.5 文本处理
- echo printf cut sort uniq wc tr awk sed

## 三、文件权限管理
### 3.1 权限三要素
- 所有者(User) 所属组(Group) 其他人(Others)

### 3.2 权限表示法
- 符号法: rwx  数字法: r=4 w=2 x=1
- 常用: 644普通文件 755脚本 600私密文件 700私有目录

### 3.3 特殊权限
- SUID(4) - 以所有者身份运行
- SGID(2) - 以组身份运行/目录继承组
- Sticky Bit(1) - 目录防误删

### 3.4 ACL扩展
- setfacl设置 getfacl查看

## 四、用户与用户组管理
### 4.1 核心概念
- UID用户ID GID组ID 主组 附加组

### 4.2 相关文件
- /etc/passwd用户信息 /etc/shadow密码 /etc/group组信息

### 4.3 管理命令
- useradd/userdel添加删除用户 passwd修改密码 usermod修改属性
- groupadd/groupdel创建删除组 groups查看组

## 五、软件包管理
### 5.1 包分类
- 二进制包(.rpm) 源码包(.tar.gz需编译) 脚本包(.sh一键安装)

### 5.2 RPM命令
- rpm -ivh安装 -Uvh升级 -e删除 -q查询 -qa列出所有

### 5.3 YUM/DNF命令
- search搜索 install安装 update升级 remove删除 clean清理

### 5.4 YUM源类型
- 网络源(在线仓库) 本地源(光盘/ISO) 离线源(预下载包)

### 5.5 源码编译
- ./configure配置 make编译 make install安装

## 六、磁盘与文件系统管理
### 6.1 设备命名
- SATA: /dev/sd[a-z]  NVMe: /dev/nvme0n1  虚拟机: /dev/vd[a-z]

### 6.2 分区方案
- MBR: ≤2TB,4主分区  GPT: >2TB,128分区

### 6.3 文件系统
- ext4成熟稳定 XFS高性能 swap交换 btrfs快照

### 6.4 挂载管理
- 临时mount 永久/etc/fstab UUID方式

### 6.5 LVM逻辑卷
- PV物理卷 VG卷组 LV逻辑卷

### 6.6 RAID级别
- RAID0条带化 RAID1镜像 RAID5奇偶校验 RAID10镜像+条带

## 七、系统管理
### 7.1 启动流程
- BIOS自检→GRUB引导→内核加载→systemd启动→进入目标

### 7.2 系统监控
- CPU: lscpu top htop  内存: free vmstat  磁盘: df du iostat  进程: ps top

### 7.3 网络管理
- ip网络配置 nmcli管理 firewall防火墙

### 7.4 SSH安全
- 禁用root登录 密钥认证 修改端口 限制用户

### 7.5 定时任务
- crontab周期性 at一次性

### 7.6 时间同步
- chrony timedatectl

### 7.7 日志管理
- rsyslog journalctl

### 7.8 SELinux
- Enforcing强制 Permissive宽松 Disabled禁用

## 八、进程管理
### 8.1 进程概念
- PID进程ID PPID父进程 前台/后台/守护进程

### 8.2 进程状态
- R运行/就绪 S可中断睡眠 D不可中断 T停止 Z僵尸

### 8.3 进程查看
- ps aux静态 top/htop动态 pstree进程树

### 8.4 信号机制
- SIGTERM(15)正常终止 SIGKILL(9)强制杀死 SIGHUP(1)重载

### 8.5 作业控制
- &后台运行 jobs查看 fg/bg切换 nohup忽略挂断

## 九、Shell脚本编程
### 9.1 脚本基础
- #!/bin/bash shebang 定义变量name="value"

### 9.2 变量与输入输出
- 引用:$name ${name} 只读readonly 删除unset
- echo/printf输出 read读取输入

### 9.3 条件循环
- if/elif/else/fi test [ ] [[ ]]
- for while until循环

### 9.4 函数与重定向
- func() { local变量 } 定义函数
- > >>输出 <输入 2>错误重定向

### 9.5 调试
- bash -x跟踪 set -euo pipefail严格模式 trap捕获信号

## 十、systemd服务管理
### 10.1 核心命令
- start启动 stop停止 restart重启 reload重载 status状态
- enable开机自启 disable取消自启

### 10.2 单元文件结构
- [Unit] Description After Wants
- [Service] Type ExecStart User Restart
- [Install] WantedBy

### 10.3 Service Type
- simple主进程即ExecStart forking fork后退出 oneshot一次性 notify启动完成通知

### 10.4 日志查看
- journalctl -u service -f实时

## 十一、服务部署
### 11.1 DHCP服务
- 作用域动态分配IP 配置 租约管理

### 11.2 DHCP中继
- 跨网段中继 路由配置

## 学习建议
- 多动手实践 掌握man手册 学会组合命令
- 注意安全备份 自动化脚本 查阅文档

