# 🛡️ Pwn 学习日志：命令链、环境变量与多路复用

**日期：** 2026-02-12
**学习平台：** pwn.college
**完成模块：**
- [x] Linux Luminarium: Chaining Commands (命令链)
- [x] Linux Luminarium: Pondering PATH (环境变量)
- [x] Linux Luminarium: Terminal Multiplexing (tmux)

---

## 1. 🔗 Chaining Commands (命令逻辑链)

在写 Shell 脚本或一行执行多条指令时，我们需要逻辑控制符。

### 1.1 核心逻辑操作符

Linux 根据上一个命令的 **Exit Code (退出码)** 来决定是否执行下一个命令。
* `0` = 成功 (Success)
* `非0` = 失败 (Failure)

| 符号 | 名称 | 逻辑含义 | 示例 | 解释 |
| :--- | :--- | :--- | :--- | :--- |
| **`;`** | **分号** | **不管成败，继续执行** | `cmd1 ; cmd2` | 跑完 cmd1，接着跑 cmd2，不在乎 cmd1 挂没挂。 |
| **`&&`** | **AND** | **成功才继续** | `cmd1 && cmd2` | 只有当 cmd1 **成功** (返回0) 时，才执行 cmd2。 |
| **`||`** | **OR** | **失败才继续** | `cmd1 || cmd2` | 只有当 cmd1 **失败** (返回非0) 时，才执行 cmd2。 |

> **🔥 经典组合拳：**
> `compile_exploit && ./run_exploit || echo "Compilation Failed"`
> (编译成功就运行，编译失败就报错)

### 1.2 Shell 脚本基础 (.sh)
当命令太长时，我们把它写入文件。

1.  **创建文件：** `nano exploit.sh`
2.  **Shebang (必须要写)：** 第一行指定解释器。
    ```bash
    #!/bin/bash
    ```
3.  **写入命令：**
    ```bash
    /challenge/solve
    export FLAG=$(cat /flag)
    echo $FLAG
    ```
4.  **赋予权限 (关键)：** 必须给脚本执行权限才能运行。
    ```bash
    chmod +x exploit.sh
    ```
5.  **运行：** `./exploit.sh` (注意要加 `./`)

---

## 2. 🗺️ Pondering PATH (PATH 环境变量)

**这是 Pwn 提权和劫持的关键知识点。**

### 2.1 什么是 PATH？
当你输入 `ls` 时，Shell 怎么知道 `ls` 程序在哪里？它会去遍历 `$PATH` 变量里列出的所有目录。

* **查看 PATH：** `echo $PATH`
* **格式：** 一堆目录路径，用冒号 `:` 分隔。
    * 例：`/usr/bin:/bin:/usr/local/bin`
* **查找顺序：** **从左到右**。Shell 找到第一个匹配的程序就停止寻找。

### 2.2 修改 PATH (劫持的核心)
你可以把你的恶意目录放到 PATH 的**最前面**，让系统优先运行你的程序。

* **语法：**
    ```bash
    export PATH=/home/hacker/scripts:$PATH
    ```
    * *解释：* 把 `/home/hacker/scripts` 加到旧 PATH 的**头部**。
* **实战场景 (PATH Hijacking)：**
    1.  假设有个程序 `run_challenge` 内部调用了 `ls` (它是通过相对路径调用的)。
    2.  你创建一个恶意脚本叫 `ls`，内容是 `cat /flag`。
    3.  你把当前目录加到 PATH 头部：`export PATH=$(pwd):$PATH`
    4.  你运行 `run_challenge`。
    5.  系统想运行 `ls`，结果先在你当前目录找到了假的 `ls`，于是执行了你的代码，Flag 到手。

### 2.3 绝对路径的坑
* 如果 PATH 被清空了 (`export PATH=""`)，`ls`、`cat` 等命令都会失效。
* **解决方法：** 使用绝对路径。
    * 用 `/bin/ls` 代替 `ls`。
    * 用 `/bin/cat` 代替 `cat`。

---

## 3. 📺 Terminal Multiplexing (tmux)

`tmux` 是黑客的“多屏神器”。它允许你在一个 SSH 连接里开多个窗口，并且**断网后程序依然在后台运行**。

### 3.1 核心概念
* **Session (会话)：** 最大的单位，可以包含多个 Window。
* **Window (窗口)：** 类似于浏览器的标签页，占满整个屏幕。
* **Pane (窗格)：** 把一个 Window 切割成多个小块（上下左右）。

### 3.2 常用快捷键 (Cheat Sheet)
**注意：** 所有快捷键前必须先按 **前缀键 (Prefix)**，默认是 **`Ctrl + B`**。

| 动作 | 命令/快捷键 | 作用 |
| :--- | :--- | :--- |
| **启动 tmux** | `tmux` | 在终端输入开启新会话。 |
| **水平分屏** | `Prefix` + `"` (双引号) | 上下切分屏幕。 |
| **垂直分屏** | `Prefix` + `%` (百分号) | 左右切分屏幕。 |
| **切换光标** | `Prefix` + `方向键` | 在不同窗格间移动。 |
| **新建窗口** | `Prefix` + `c` | 创建一个新的“标签页”。 |
| **切换窗口** | `Prefix` + `n` / `p` | Next / Previous 窗口。 |
| **滚动模式** | `Prefix` + `[` | 进入复制/滚动模式 (按 `q` 退出)。 |
| **分离会话** | `Prefix` + `d` | **Detach**。离开 tmux 回到普通 Shell (程序还在后台跑)。 |
| **重连会话** | `tmux a` | **Attach**。找回刚才分离的会话。 |

### 3.3 为什么 Pwn 需要 tmux？
1.  **一边写代码，一边运行：** 左边开 `vim exploit.py`，右边开终端运行 `./exploit.py`，不用切来切去。
2.  **GDB 调试：** 这里的 `gdb` 经常需要分屏显示源代码、寄存器和堆栈，tmux 是标配。
3.  **防断连：** 跑耗时脚本时，网络断了脚本不会停，重连 `tmux a` 就能看到进度。