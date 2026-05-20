---
title: "12.Bash & Shell 脚本编程"
slug: 12-bash-shell-scripting
published: 2026-05-20
description: "Bash 与 Shell 脚本编程笔记，记录脚本结构、变量、流程控制和自动化实践。"
tags: ["Linux", "Bash", "Shell", "脚本"]
category: Linux
draft: false
---
## 概述：什么是 Shell？Bash 又是什么？

- **Shell** 是用户与操作系统内核之间的命令解释器。
- 常见 shell：
  - `sh`（`Bourne Shell`，最原始）
  - `bash`（`Bourne-Again Shell`，Linux 默认）
  - `zsh`（功能更强，macOS 新版本默认）
  - `dash`（轻量级，Debian 启动用）

> 本文以 **Bash** 为主，语法兼容 `sh` 的大部分内容。

---

## 脚本基本结构

### ✅ 标准格式
```bash
#!/bin/bash
# 注意：第一行称为 "Shebang"，告诉系统用哪个解释器运行

# 第二行开始写注释或代码
echo "Hello, World!"
```

💡 小贴士：
- 文件扩展名可为 `.sh`（推荐但不是必须）
- 赋予执行权限：`chmod +x hello.sh`
- 运行脚本：`./hello.sh`

---

## 执行 Shell 脚本的 3 种方式

| 方式               | 命令                                | 特点                          |
| ------------------ | ----------------------------------- | ----------------------------- |
| 直接执行           | `./script.sh`                       | 需要 `+x` 权限，启动子 shell  |
| 使用 bash 命令     | `bash script.sh`                    | 不需要权限，总是新建进程      |
| 源码加载（source） | `source script.sh` 或 `. script.sh` | 在当前 shell 中执行，变量保留 |

> 推荐使用 `bash script.sh` 进行开发测试。

---

## 变量与参数

### 定义变量
```bash
name="Alice"
age=25
PI=3.14159
```

📌 规则：
- 无空格在 `=` 两边
- 变量名只能包含字母、数字、下划线 `_`，不能以数字开头
- 默认所有变量是字符串类型

### 引用变量
```bash
echo $name
echo ${name}    # 更标准，推荐用于复杂场景
```

### 只读变量
```bash
readonly site="https://example.com"
site="new"  # ❌ 报错：cannot assign
```

### 删除变量
```bash
unset name
unset age
# 注意：不能 unset $0、$$ 等特殊变量
```

---

## 输入输出处理（read / echo / printf）

### `echo` —— 输出文本
```bash
echo "Hello"
echo -n "No newline"   # 不换行输出
echo -e "Line\nBreak"  # 启用转义字符（\n \t 等）
```

### `printf` —— 格式化输出（更强大）
```bash
printf "Name: %s, Age: %d\n" "Bob" 30
# 支持 %s %d %f %c 等格式符
```

### `read` —— 从键盘读取输入
```bash
echo -n "Enter your name: "
read username
echo "Hello, $username"
```

#### read 高级用法
```bash
read -p "Password: " -s password  # -s 隐藏输入（密码）
read -t 5 response                # -t 设置超时（5秒）
read -a arr                       # 读入数组
```

---

## 条件判断（if、test、[ ]、[[ ]]）

### 基本 if 结构
```bash
if [ 条件 ]; then
    命令
elif [ 其他条件 ]; then
    命令
else
    命令
fi
```

> 注意：中括号 `[ ]` 是一个命令（即 `test` 命令），左右必须有空格！

### 示例：比较数值
```bash
if [ $age -gt 18 ]; then
    echo "Adult"
fi
```

| 操作符（整数） | 含义     |
| -------------- | -------- |
| `-eq`          | 等于     |
| `-ne`          | 不等于   |
| `-lt`          | 小于     |
| `-le`          | 小于等于 |
| `-gt`          | 大于     |
| `-ge`          | 大于等于 |

### 字符串比较
```bash
if [ "$name" = "Alice" ]; then
    echo "Hi Alice"
fi
```

| 操作符（字符串） | 含义         |
| ---------------- | ------------ |
| `=` 或 `==`      | 是否相等     |
| `!=`             | 是否不等     |
| `-z str`         | 字符串为空？ |
| `-n str`         | 字符串非空？ |

> 💡 推荐使用双引号包裹变量防止出错：`[ "$var" = "value" ]`

### 使用 `[[ ]]`（Bash 扩展，功能更强）

```bash
if [[ $name == A* ]]; then
    echo "Starts with A"
fi
```
支持通配符匹配 `*`, `<`, `>` 比较字符串大小写等。

---

## 字符串操作

### 获取长度
```bash
str="hello"
echo ${#str}           # 输出：5
```

### 截取子串
```bash
str="abcdefg"
echo ${str:2:3}        # 从第2位开始取3个字符 → "cde"
echo ${str:2}          # 从第2位到结尾 → "cdefg"
```

### 替换字符串
```bash
filename="report.txt.bak"
echo ${filename/.txt/.log}    # 替换第一个匹配 → report.log.bak
echo ${filename//./_}        # 全局替换 → report_txt_bak
```

### 提取路径部分（常用于脚本）
```bash
path="/home/user/docs/file.txt"

dirname $path     # → /home/user/docs
basename $path    # → file.txt
basename $path .txt  # → file （去掉后缀）
```

---

## 数学运算

### 方法一：`$(( ))` 整数运算
```bash
a=5
b=3
sum=$((a + b))        # 加法
diff=$((a - b))       # 减法
prod=$((a * b))       # 乘法
div=$((a / b))        # 除法（向下取整）
mod=$((a % b))        # 求余
power=$((a ** 2))     # 幂运算
```

### 方法二：`expr` 命令（旧方式，少用）
```bash
result=$(expr $a + $b)
```
⚠️ 注意：`*` 必须转义：`\*`

### 浮点数计算？用 `bc`
```bash
echo "scale=2; 10/3" | bc    # 输出：3.33
result=$(echo "sqrt(16)" | bc -l)
```
> 安装：`sudo apt install bc` 或 `yum install bc`

---

## 循环结构（for / while / until）

### for 循环

#### 标准 for
```bash
for i in 1 2 3 4 5; do
    echo $i
done
```

#### 类 C 风格（需双括号）
```bash
for (( i=1; i<=5; i++ )); do
    echo $i
done
```

#### 遍历数组
```bash
files=("a.txt" "b.log" "c.conf")
for file in "${files[@]}"; do
    echo "Processing $file"
done
```

### while 循环

```bash
count=1
while [ $count -le 5 ]; do
    echo $count
    ((count++))
done
```

#### 读取文件每行
```bash
while IFS= read line; do
    echo "Line: $line"
done < /etc/passwd
```

### until 循环（当条件为假时继续）

```bash
i=1
until [ $i -gt 5 ]; do
    echo $i
    ((i++))
done
```

---

##  函数定义与使用

### 定义函数
```bash
greet() {
    local name=$1
    echo "Hello, $name"
}
```

> 提示：
> - `local` 创建局部变量，避免污染全局
> - `$1`, `$2`... 是传入的参数

### 调用函数
```bash
greet "Alice"
greet $USER
```

### 返回值说明
- `return N`：返回状态码（0~255），用于判断成功失败
- 若需返回数据，应使用 `echo` 输出并捕获：

```bash
get_time() {
    date +"%H:%M:%S"
}

now=$(get_time)
echo "Current time: $now"
```

---

## 位置参数与特殊变量

| 变量        | 含义                               |
| ----------- | ---------------------------------- |
| `$0`        | 脚本名称（含路径）                 |
| `$1` ~ `$9` | 第1~第9个参数                      |
| `${10}`     | 第10个及以上参数（必须加 {}）      |
| `$#`        | 参数总个数                         |
| `$@`        | 所有参数列表（每个作为独立字符串） |
| `$*`        | 所有参数作为一个字符串             |
| `$$`        | 当前脚本进程 ID（PID）             |
| `$?`        | 上一条命令的退出状态（0 表示成功） |
| `$!`        | 最近一个后台进程的 PID             |
| `$PPID`     | 父进程 ID                          |

✅ 推荐遍历所有参数的方法：
```bash
for arg in "$@"; do
    echo "Arg: $arg"
done
```

---

## 重定向与管道

### 重定向

| 操作 | 说明                        |
| ---- | --------------------------- |
| `>`  | 覆盖输出到文件              |
| `>>` | 追加输出到文件              |
| `<`  | 从文件读取输入              |
| `2>` | 错误输出重定向              |
| `&>` | 同时重定向 stdout 和 stderr |

示例：
```bash
ls /root > output.txt 2>&1    # 正确+错误都保存
command >> log.txt             # 日志追加
```

### 管道 `|`
将前一个命令输出作为下一个命令输入。

```bash
ps aux | grep ssh
cat /etc/passwd | cut -d: -f1 | sort
history | tail -10 | grep sudo
```

---

## 退出状态与错误处理

### 查看上条命令状态
```bash
ls /fake/path
echo $?   # → 2（非零表示失败）
```

### 主动设置退出码
```bash
if [ ! -f config.cfg ]; then
    echo "Config missing!" >&2
    exit 1
fi
```

### 错误中断：set -e
```bash
#!/bin/bash
set -e  # 一旦某条命令失败，脚本立即终止
command1
command2  # 如果失败，不会继续执行后面命令
```

### 组合开关（推荐开发中使用）
```bash
set -euo pipefail
# -e: 出错即停
# -u: 使用未定义变量时报错
# -o pipefail: 管道中任意环节出错即视为失败
```

---

## 脚本调试技巧

### 方法一：打印模式 `-x`
```bash
bash -x script.sh       # 显示每一行执行过程
```

或在脚本内部启用：
```bash
set -x
your_commands_here
set +x  # 关闭
```

### 方法二：启用严格模式（前面已提）
```bash
set -euo pipefail
```

### 方法三：使用 trap 捕获信号
```bash
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/temp.*
}

trap cleanup EXIT  # 当脚本结束时自动调用 cleanup
```

可用于资源释放，如删除临时文件、关闭连接等。

---

## 最佳实践与安全建议

✅ **推荐做法**：
- 添加 shebang 和注释
- 使用 `set -euo pipefail` 开启严格模式
- 变量尽量用双引号包围：`"$var"`
- 使用 `local` 定义函数内的变量
- 给脚本合理命名并分类存放（如 `/usr/local/bin/`）
- 使用 `readonly` 保护常量
- 对外部输入进行验证
- 记录日志而非仅打印屏幕

🚫 **避免行为**：
- 在 root 下随意运行未经审查的脚本
- 硬编码敏感信息（密码、密钥）
- 使用 `rm -rf $var/` 而不做检查（可能变 `rm -rf /`）
- 忽略错误状态
- 不给变量加引号导致词分裂（Word Splitting）

---

## 附录A：常用内置命令速查

| 命令            | 说明                  |
| --------------- | --------------------- |
| `cd`            | 切换目录              |
| `pwd`           | 显示当前路径          |
| `export`        | 导出环境变量          |
| `source` 或 `.` | 在当前 shell 加载脚本 |
| `readonly`      | 设置只读变量          |
| `unset`         | 删除变量              |
| `alias`         | 创建别名              |
| `jobs`          | 查看后台任务          |
| `exit N`        | 退出脚本并返回状态 N  |
| `sleep N`       | 暂停 N 秒             |

---

## 附录B：文件测试条件表

可在 `[ ]` 判断中使用：

| 表达式                | 含义                            |
| --------------------- | ------------------------------- |
| `[ -f file ]`         | 是否为普通文件                  |
| `[ -d dir ]`          | 是否为目录                      |
| `[ -r file ]`         | 是否可读                        |
| `[ -w file ]`         | 是否可写                        |
| `[ -x file ]`         | 是否可执行                      |
| `[ -s file ]`         | 文件大小非零？                  |
| `[ -e file ]`         | 文件是否存在（任何类型）        |
| `[ file1 -nt file2 ]` | file1 比 file2 更新（修改时间） |
| `[ file1 -ot file2 ]` | file1 比 file2 更旧             |

例：
```bash
if [ -f "/etc/passwd" ]; then
    echo "passwd exists"
fi
```

