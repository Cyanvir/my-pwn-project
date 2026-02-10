# 🛡️ Pwn 学习日志：进程管理与权限体系

**日期：** 2026-02-10
**学习平台：** pwn.college
**完成模块：**
- [x] Linux Luminarium: Processes (进程)
- [x] Linux Luminarium: Users (用户)
- [x] Linux Luminarium: Permissions (权限)

---

## 1. ⚙️ Processes (进程管理)

进程是正在运行的程序的实例。掌握进程管理是控制 Shell 和后台任务的基础。

### 1.1 查看进程 (ps)

| 命令/参数 | 功能描述 | 示例 |
| :--- | :--- | :--- |
| **`ps`** | Process Status，查看当前终端下的进程。 | `ps` |
| **`-ef`** | **Standard Syntax**。查看系统所有进程（包含其他用户的）。 | `ps -ef` |
| **`aux`** | **BSD Syntax**。同样查看所有进程，但显示 CPU/内存占用率。 | `ps aux` |
| **`-o`** | **Output**。自定义输出列（只看你想看的信息）。 | `ps -o pid,cmd` |

### 1.2 进程状态代码 (STAT)

在 `ps` 输出中，STAT 列代表进程当前的状态：

| 状态符 | 含义 | 解释 |
| :--- | :--- | :--- |
| **`R`** | **Running** | 正在运行或准备运行。 |
| **`R+`** | **Running (Foreground)** | 在**前台**运行的进程组（占用着你的终端）。 |
| **`S`** | **Sleeping** | 睡眠状态（等待某些事件，如输入）。大多数进程处于此状态。 |
| **`T`** | **Stopped / Traced** | 已停止/挂起（被 `Ctrl+Z` 暂停或被调试器挂起）。 |
| **`Z`** | **Zombie** | 僵尸进程（已结束但父进程未回收）。 |

### 1.3 进程控制与后台 (Job Control)

如何让程序在后台跑，或者把挂起的程序救回来？

| 命令/符号 | 功能描述 | 场景 |
| :--- | :--- | :--- |
| **`&`** | **后台运行符**（放在命令末尾）。 | `python exp.py &` (直接在后台启动，不占用终端) |
| **`Ctrl + Z`** | **挂起**当前前台进程（状态变为 `T`）。 | 程序跑死循环了，先暂停它。 |
| **`fg`** | **Foreground**。把后台/挂起的进程调回**前台**。 | `fg` (恢复最近一个) |
| **`bg`** | **Background**。让挂起的进程在**后台**继续运行。 | 先 `Ctrl+Z` 暂停，再 `bg` 让它在后台跑。 |
| **`kill`** | 发送信号给进程（通常用于结束进程）。 | `kill <PID>` (默认发送 TERM 信号) |

### 1.4 特殊变量
* **`$?`** : **退出状态码 (Exit Code)**。
    * 检查上一个命令是否执行成功。
    * `0` = 成功。
    * `非0` (如 1, 127) = 失败/报错。
    * *用法：* `ls /root; echo $?` (如果没权限，会输出非0数字)。

---

## 2. 👤 Users (用户体系)

Linux 是多用户系统，Pwn 的最终目标通常是从普通用户变成 Root 用户。

* **Root 用户**：ID 为 0 的超级管理员，拥有至高无上的权限（God Mode）。
* **su (Switch User)**：切换用户身份。
    * `su root` (输入密码后切换到 root)。
* **sudo (SuperUser Do)**：以 root 身份执行单条命令。
    * `sudo cat /flag` (作为 root 读取 flag)。

---

## 3. 🔐 Permissions (权限管理)

**这是 Pwn 中最关键的概念之一（特别是 SUID）。**

### 3.1 权限解读 (`ls -l`)
当你输入 `ls -l file`，你会看到类似 `-rwxr-xr--` 的字符串：

1.  **第一位**：文件类型 (`-` 文件, `d` 目录, `l` 链接)。
2.  **后九位**：权限位，每三位一组。

| 分组 | 代表用户 | 缩写 |
| :--- | :--- | :--- |
| **前3位** | **User (Owner)** | `u` (拥有者) |
| **中3位** | **Group** | `g` (所属组) |
| **后3位** | **Others** | `o` (其他人) |

### 3.2 修改权限 (chmod)

`chmod` (Change Mode) 用于修改文件权限。

**操作符：**
* `+` (增加权限)
* `-` (剥夺权限)
* `=` (设定为唯一权限)

**权限位表：**

| 符号 | 含义 | 对文件的作用 | 对目录的作用 |
| :--- | :--- | :--- | :--- |
| **`r`** | Read | 可以读取内容 (`cat`) | 可以列出目录内容 (`ls`) |
| **`w`** | Write | 可以修改内容 | 可以删除/新建文件 |
| **`x`** | Execute | 可以作为程序运行 (`./file`) | 可以进入目录 (`cd`) |
| **`s`** | **SUID/SGID** | **⭐ 提权关键**。运行时**临时获得文件拥有者(root)的权限**。 | (SGID在目录下有不同含义) |

**常用命令示例：**
```bash
chmod u+x file      # 给拥有者增加执行权限
chmod g-w file      # 剥夺组用户的写权限
chmod o=r file      # 其他人只能读，不能写也不能执行
chmod u+s file      # ⭐ 设置 SUID 位 (变红/变亮)，Pwn 题常见
```
### 3.3 修改归属 (Ownership)

在 Linux 中，文件不仅有权限（rwx），还有“主人”（Owner）和“所属组”（Group）。

| 命令 | 全称 | 功能描述 | 语法示例 | 注意事项 |
| :--- | :--- | :--- | :--- | :--- |
| **`chown`** | **Ch**ange **Own**er | 修改文件的**拥有者**。 | `chown hacker flag` | **通常需要 Root 权限**。普通用户不能随便把文件“过户”给别人（为了安全）。 |
| **`chgrp`** | **Ch**ange **Gr**oup | 修改文件的**所属组**。 | `chgrp hackers flag` | 你只能把文件改到**你所在的组**。 |

> **💡 Pwn 场景举例：**
> 如果 `flag` 文件的拥有者是 `root`，权限是 `r--------` (只有拥有者能读)，那你作为一个普通用户 `hacker` 是读不了的。
> Pwn 的目标往往就是让自己变成 `root` (提权)，或者利用漏洞把 `flag` 的权限改成 `rwxrwxrwx`。

---

## 📚 拓展资料 (Recommended Reading)

* **Terminals Are Weird (终端是很怪的)**
    * **简介：** 这是一篇深入浅出的经典文章，解释了终端（Terminal）、TTY、PTY 的历史遗留问题和底层逻辑。如果你好奇为什么 `Ctrl+C` 能终止程序，或者为什么有时候终端会乱码，这篇文章会给你答案。
    * **链接：** [https://catern.com/posts/terminal_quirks.html](https://catern.com/posts/terminal_quirks.html)

---

### 🏆 今日复习重点 (Key Takeaways)

1.  **进程状态 (Process State)**
    * **R (Running)**: 正在跑。
    * **S (Sleeping)**: 等待中（大部分时间）。
    * **T (Stopped)**: 被挂起 (`Ctrl+Z`)，需要 `fg` 或 `bg` 唤醒。
    * **Z (Zombie)**: 僵尸进程（父进程不管它了）。

2.  **权限核心 (Permissions)**
    * **`chmod +x`**: 让脚本能运行。
    * **`chmod u+s` (SUID)**: **这是 Pwn 的核心考点**。如果一个文件由 root 拥有且有 `s` 位，普通用户运行它时，会暂时获得 root 权限。