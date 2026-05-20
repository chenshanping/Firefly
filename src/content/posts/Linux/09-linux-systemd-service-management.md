---
title: "9.Linux systemd 服务管理"
slug: 09-linux-systemd-service-management
published: 2026-05-20
description: "Linux systemd 服务管理笔记，记录 systemctl、unit 文件和常见服务管理操作。"
tags: ["Linux", "systemd", "服务管理"]
category: Linux
draft: false
---
## 概述：什么是服务？

在 Linux 中，**服务**（也称守护进程 daemon）是后台运行的程序，用于提供长期可用的功能，如：

- Web 服务器：Nginx、Apache
- 数据库：MySQL、PostgreSQL
- 安全登录：SSH（sshd）
- 定时任务：cron
- 系统监控：Prometheus Node Exporter

现代 Linux 使用 **systemd** 作为初始化系统（PID 1）和服务管理器。它通过 `.service` 类型的“单元文件”来描述每个服务的行为。

---

##  systemctl 核心命令速查

| 功能               | 命令                            |
| ------------------ | ------------------------------- |
| 启动服务           | `sudo systemctl start <name>`   |
| 停止服务           | `sudo systemctl stop <name>`    |
| 重启服务           | `sudo systemctl restart <name>` |
| 重载配置（不重启） | `sudo systemctl reload <name>`  |
| 查看状态           | `systemctl status <name>`       |
| 开机自启           | `sudo systemctl enable <name>`  |
| 取消开机启动       | `sudo systemctl disable <name>` |
| 查看是否启用       | `systemctl is-enabled <name>`   |
| 重新加载 unit 配置 | `sudo systemctl daemon-reload`  |

> 💡 提示：
> - `<name>` 可以是 `nginx`、`mysql.service` 或完整路径。
> - 推荐省略 `.service` 扩展名（自动补全）。

---

## 查看和管理系统服务

### 列出所有已加载的服务
```bash
systemctl list-units --type=service
```

### 列出所有服务（包括未运行）
```bash
systemctl list-units --type=service --all
```

### 查看开机启动状态
```bash
systemctl list-unit-files --type=service
```
输出中显示 `enabled` / `disabled` / `static`（被依赖但不能单独启用）

### 过滤特定服务
```bash
systemctl list-unit-files --type=service | grep nginx
```

---

## systemd 单元文件结构

### 单元文件位置

| 目录                       | 用途                                 |
| -------------------------- | ------------------------------------ |
| `/usr/lib/systemd/system/` | 系统默认服务（由软件包安装）         |
| `/etc/systemd/system/`     | 管理员自定义或覆盖服务（优先级更高） |
| `/run/systemd/system/`     | 运行时生成的临时服务                 |

> 修改建议：若要修改系统服务，推荐复制原文件到 `/etc/systemd/system/` 再编辑，避免升级覆盖。

### 基本语法格式

```ini
[Unit]
Description=简要说明
After=network.target

[Service]
Type=simple
ExecStart=/path/to/executable
User=myuser

[Install]
WantedBy=multi-user.target
```

> ✅ 注意：
> - 所有路径必须为绝对路径
> - 不支持 shell 变量扩展（如 `$HOME`）
> - 多行值可用反斜杠 `\` 续行

---

## `[Unit]` 区块详解

该部分定义服务的元信息、依赖关系和条件。

| 字段             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| `Description=`   | 服务描述，出现在 `systemctl status` 中                       |
| `Documentation=` | 文档链接，支持 `man:`、`http://`、`file:`                    |
| `After=`         | 表示启动顺序：“在此目标之后”                                 |
| `Before=`        | 在其他单元前启动                                             |
| `Requires=`      | 强依赖，若失败则本服务也失败                                 |
| `Wants=`         | 弱依赖，推荐使用                                             |
| `Conflicts=`     | 与其他服务互斥                                               |
| `Condition...=`  | 启动前提条件，例如：<br>- `ConditionPathExists=/var/run/docker.sock`<br>- `ConditionHost=prod-server` |

💡 实际用法建议：
```ini
[Unit]
Description=Redis Server
Documentation=man:redis-server(1)
After=network.target
Wants=network.target
ConditionFileIsExecutable=/usr/bin/redis-server
```

---

## `[Service]` 区块详解

这是最关键的部分，控制服务如何运行。

### Type= 启动类型

| Type             | 场景说明                                                     |
| ---------------- | ------------------------------------------------------------ |
| `simple`（默认） | 主进程即 `ExecStart` 命令本身，前台阻塞运行（Python Flask、Node.js） |
| `forking`        | 守护进程模型，主进程 fork 后退出（Nginx、MySQL）             |
| `oneshot`        | 一次性执行，配合 `RemainAfterExit=yes` 使用                  |
| `notify`         | 启动完成后通过 sd_notify() 通知 systemd                      |
| `dbus`           | 注册 D-Bus 名称后才算启动成功                                |

> ⚠️ 若设置 `forking`，务必配合 `PIDFile=` 使用！

---

### 执行命令设置

| 字段             | 说明                           |
| ---------------- | ------------------------------ |
| `ExecStart=`     | 必须，指定启动命令（绝对路径） |
| `ExecStartPre=`  | 启动前检查，如测试配置有效性   |
| `ExecStartPost=` | 启动成功后执行的操作           |
| `ExecReload=`    | 重载命令，如 `kill -HUP`       |
| `ExecStop=`      | 优雅停止命令                   |
| `ExecStopPost=`  | 停止后清理操作                 |

🔧 示例：
```ini
ExecStartPre=/usr/sbin/nginx -t -q
ExecStart=/usr/sbin/nginx -g "daemon off;"
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -TERM $MAINPID
```

📌 特殊变量：
- `$MAINPID`：主进程 PID
- `$ROOT`：根目录
- `$HOME`：用户家目录（需显式传递）
- `$SERVICE`：服务名称

---

### 进程控制参数

| 字段                     | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `Restart=`               | 是否重启？常用值：<br>• `no`（默认）<br>• `always`<br>• `on-failure` ✅ 推荐生产环境使用 |
| `RestartSec=5s`          | 重启前等待时间                                               |
| `TimeoutStartSec=300`    | 启动超时时间（秒）                                           |
| `TimeoutStopSec=60`      | 停止等待时间                                                 |
| `KillMode=control-group` | 杀掉整个进程组（默认）                                       |
| `KillSignal=SIGTERM`     | 发送的第一个信号                                             |
| `SendSIGKILL=yes`        | 是否强制 kill（超时后）                                      |

✅ 推荐配置片段：
```ini
Restart=on-failure
RestartSec=5s
TimeoutStartSec=120
TimeoutStopSec=30
```

---

### 用户权限与安全

| 字段                     | 说明                                 |
| ------------------------ | ------------------------------------ |
| `User=`                  | 以哪个用户身份运行（避免 root）      |
| `Group=`                 | 用户组                               |
| `SupplementaryGroups=`   | 补充组列表                           |
| `UMask=0027`             | 设置文件权限掩码                     |
| `RuntimeDirectory=myapp` | 自动创建 `/run/myapp` 并归属正确用户 |
| `StateDirectory=myapp`   | 创建持久化状态目录 `/var/lib/myapp`  |
| `CacheDirectory=cache`   | 创建缓存目录 `/var/cache/myapp`      |

✅ 安全增强推荐：
```ini
User=www-data
Group=www-data
UMask=0027
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
```

---

### 环境变量与路径

| 字段                                | 说明                   |
| ----------------------------------- | ---------------------- |
| `WorkingDirectory=`                 | 设置工作目录           |
| `Environment=KEY=value`             | 设置单个环境变量       |
| `EnvironmentFile=/path/to/env.conf` | 从文件加载多条         |
| `StandardOutput=journal`            | 输出到日志系统（默认） |
| `StandardError=inherit`             | 错误流继承 stdout      |

💼 典型用法：
```ini
WorkingDirectory=/opt/myproject
Environment="NODE_ENV=production"
EnvironmentFile=-/etc/default/myproject
StandardOutput=append:/var/log/myproject.log
```

---

### 资源限制与沙箱安全（高级）

| 字段                                  | 说明           |
| ------------------------------------- | -------------- |
| `LimitNOFILE=65536`                   | 最大打开文件数 |
| `LimitNPROC=1024`                     | 最大进程数     |
| `PrivateDevices=true`                 | 隔离设备访问   |
| `ReadOnlyDirectories=/etc /usr`       | 挂载为只读目录 |
| `ReadWriteDirectories=/var/lib/app`   | 指定可写目录   |
| `InaccessibleDirectories=/root /home` | 完全禁止访问   |

🔧 示例（高安全性应用）：
```ini
NoNewPrivileges=true
PrivateTmp=true
ProtectHome=read-only
ProtectSystem=strict
RestrictSUIDSGID=true
LockPersonality=true
```

---

## `[Install]` 区块详解

决定服务如何被“启用”。

| 字段                         | 说明                   |
| ---------------------------- | ---------------------- |
| `WantedBy=multi-user.target` | 最常用：普通服务器模式 |
| `WantedBy=graphical.target`  | 图形界面模式           |
| `RequiredBy=`                | 强依赖方式启用（少用） |
| `Alias=`                     | 别名，便于引用         |
| `Also=`                      | 同时启用另一个服务     |

📌 解释 `multi-user.target`：
- 相当于传统 runlevel 3
- 不包含图形界面
- 适合绝大多数服务器

🔁 启用效果：
```bash
sudo systemctl enable myapp.service
```
会在 `/etc/systemd/system/multi-user.target.wants/` 下创建软链接。

---

## 创建自定义服务示例

### 示例：部署一个 Python HTTP 服务

#### Step 1：编写脚本 `/opt/hello/app.py`
```python
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"Hello from systemd service!")

if __name__ == "__main__":
    HTTPServer(("0.0.0.0", 8000), Handler).serve_forever()
```

#### Step 2：创建服务文件 `/etc/systemd/system/hello-web.service`

```ini
[Unit]
Description=Simple Python Web Service
After=network.target
Wants=network.target

[Service]
Type=simple
User=www-data
Group=www-data
WorkingDirectory=/opt/hello
ExecStart=/usr/bin/python3 app.py
Restart=on-failure
RestartSec=5s
Environment=PORT=8000

# 安全加固
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true

[Install]
WantedBy=multi-user.target
```

#### Step 3：启用并启动服务

```bash
sudo systemctl daemon-reload
sudo systemctl enable hello-web.service
sudo systemctl start hello-web.service
sudo systemctl status hello-web.service
```

#### Step 4：查看日志

```bash
journalctl -u hello-web -f
```

---

## 日志查看与排错技巧

### 使用 `journalctl` 查看日志

| 命令                                      | 说明                         |
| ----------------------------------------- | ---------------------------- |
| `journalctl -u name`                      | 查看某服务日志               |
| `journalctl -u name --since "1 hour ago"` | 查最近一小时                 |
| `journalctl -u name -n 50`                | 显示最后 50 行               |
| `journalctl -u name -f`                   | 实时追踪日志（类似 tail -f） |
| `journalctl --no-pager`                   | 强制输出不翻页               |

### 排查服务无法启动

1. 查看状态：
   ```bash
   systemctl status myservice
   ```
2. 查看日志：
   ```bash
   journalctl -u myservice -n 100 --no-pager
   ```
3. 验证语法：
   ```bash
   sudo systemd-analyze verify /etc/systemd/system/myservice.service
   ```
4. 检查路径和权限：
   - 脚本是否可执行？
   - 目录是否存在且权限正确？
   - 用户是否已创建？

---

##  最佳实践与注意事项

✅ **推荐做法**：
- 使用非 root 用户运行服务
- 启用 `Restart=on-failure`
- 设置合理的超时时间
- 使用 `Wants=` 而不是 `Requires=`
- 修改后运行 `systemctl daemon-reload`
- 将自定义服务放在 `/etc/systemd/system/`

🚫 **避免行为**：
- 直接编辑 `/usr/lib/systemd/system/*.service`
- 使用相对路径
- 忽略日志排查
- 让服务永远以 root 运行

---

## 附录A：常见服务场景命令表

| 场景                | 命令                                                       |
| ------------------- | ---------------------------------------------------------- |
| 检查 Nginx 是否运行 | `systemctl status nginx`                                   |
| 重启 MySQL          | `sudo systemctl restart mysql`                             |
| 设置 Redis 开机自启 | `sudo systemctl enable redis`                              |
| 查看 Apache 日志    | `journalctl -u apache2 -f`                                 |
| 临时关闭防火墙      | `sudo systemctl stop firewalld`                            |
| 查看监听端口        | `ss -tulnp \| grep :80`                                    |
| 查看所有启用的服务  | `systemctl list-unit-files --type=service \| grep enabled` |
| 重启 systemd 配置   | `sudo systemctl daemon-reload`                             |


