# 🛡️ Pwn 学习日志：Linux 基础与路径探索

**日期：** 2026-02-05
**学习平台：** pwn.college
**当前模块：** Linux Luminarium (Linux 启蒙)
**进度：** - [x] Welcome: Interacting with the Dojo
- [x] Linux Luminarium: Hello Hackers
- [x] Linux Luminarium: Pondering Paths

---

## 📝 核心知识点梳理

### 1. 初识 Dojo 环境 (Interacting with the Dojo)
* **交互方式：** 我们通过网页端的终端（Terminal）与远程服务器进行交互。
* **本质：** 这里的终端本质上是一个 **Shell**，它接收我们的文本指令，传给操作系统内核执行，并返回结果。
* **Flag 的获取：** 在 pwn.college 中，我们的目标通常是运行某个特定的程序或读取特定的文件来获取 `pwn.college{...}` 格式的 Flag。

### 2. 执行程序 (Hello Hackers)
* **程序的本质：** 在 Linux 中，程序通常就是存储在文件系统中的一个**文件**。
* **调用方式：** 需要通过**文件路径**来告诉 Shell 去哪里找到并运行这个程序。
* **关键命令：**
    ```bash
    /challenge/run
    ```
    * 这是 pwn.college 专门设置的挑战程序，运行它通常会根据你的操作给出 Flag 或提示。

### 3. 路径详解 (Pondering Paths) —— ⭐ 今日重点
Linux 的文件系统是一棵倒置的树，根节点是 `/` (Root)。

#### A. 绝对路径 (Absolute Path)
* **定义：** 从根目录 `/` 开始，完整描述文件位置的路径。
* **特征：** **一定是以 `/` 开头的。**
* **例子：**
    * `/challenge/run` (根目录下的 challenge 目录下的 run 文件)
    * `/home/hacker` (用户的家目录)
* **何时使用：** 当你迷路了，或者无论你在哪个目录下都想精准调用某个文件时，用绝对路径最保险。

#### B. 相对路径 (Relative Path)
* **定义：** 相对于“当前所在目录”的路径。
* **特征：** **不以 `/` 开头。**
* **例子：**
    * 如果我现在在 `/challenge` 目录下，直接输入 `run` (或者 `./run`) 就能找到文件。
    * 如果我在 `/` 目录下，输入 `challenge/run`。
* **特殊符号：**
    * `.`  代表当前目录
    * `..` 代表上一级目录（父目录）

#### C. 常用路径命令
* `pwd` (Print Working Directory)：告诉我，我现在在哪里？
* `cd` (Change Directory)：切换我的位置。
    * `cd /`：去根目录。
    * `cd /home/hacker`：去家目录。
    * `cd ..`：向上一级。

---

## 💡 学习心得与常见误区

1.  **"No such file or directory" 报错：**
    * 原因通常是路径写错了。
    * **检查方法：** 如果我写的是 `run`，但我当前不在 `/challenge` 目录下，Shell 就找不到它。此时必须用绝对路径 `/challenge/run`。

2.  **根目录 `/` vs 路径分隔符 `/`：**
    * 出现在路径最开头的 `/` 是**根目录**（Root）。
    * 出现在中间的 `/` 只是**目录层级的分隔符**。
    * *例：* `/usr/bin/python` (第一个是根，后面的只是分隔)。

3.  **Linux 不依赖扩展名：**
    * 不像 Windows 需要 `.exe`，Linux 下的文件能不能运行，取决于它有没有“执行权限”（这是后续 Permissions 章节的内容），而不是看名字。

---

## 📅 下一步计划 (Next Steps)

1.  **继续 Linux Luminarium：**
    * 下一章通常是 `Comprehending Commands` 或 `File Globbing`。
    * 重点关注如何查看文件内容（`cat`）。
2.  **实操习惯：**
    * 每次报错时，先用 `pwd` 确认自己在哪里，再用 `ls` 看周围有什么文件。