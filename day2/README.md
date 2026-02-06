# 🛡️ Pwn 学习日志：文件操作与文档查询

**日期：** 2026-02-06
**学习平台：** pwn.college
**完成模块：** 
- [x] Linux Luminarium: Comprehending Commands
- [x] Linux Luminarium: Digesting Documentation

---

## 1. 核心命令速查表 (Command Cheat Sheet)

| 命令 | 英文全称 | 功能描述 | 常用示例 |
| :--- | :--- | :--- | :--- |
| **cat** | Concatenate | 查看文件内容（一次性显示所有） | `cat flag` |
| **ls** | List | 列出当前目录下的文件和文件夹 | `ls -a` (显示隐藏文件) |
| **touch** | Touch | 创建一个空文件 / 更新文件时间戳 | `touch newfile` |
| **mkdir** | Make Directory | 创建一个新的文件夹（目录） | `mkdir mydir` |
| **cp** | Copy | 复制文件或目录 | `cp flag flag_bak` |
| **mv** | Move | 移动文件（也可以用来**重命名**文件） | `mv oldname newname` |
| **rm** | Remove | 删除文件（不可恢复，慎用） | `rm junk_file` |
| **grep** | Global Regular Expression Print | 在文件中搜索特定的字符串/文本 | `grep "pwn.college" flag` |
| **diff** | Difference | 对比两个文件的不同之处 | `diff file1 file2` |
| **find** | Find | 在目录树中查找文件（按名字、大小等） | `find / -name "flag"` |
| **man** | Manual | **最重要的命令**：查看命令的使用手册 | `man ls` |

---

## 2. 关键知识点：执行程序的规则

### ⚠️ 为什么必须加 `./` ？
在 Linux 中，当你输入一个命令（如 `ls`）时，Shell 会去系统环境变量 `PATH` 指定的目录（如 `/bin`, `/usr/bin`）里寻找这个程序。

如果你想运行**当前目录下**的一个可执行程序（比如 pwn.college 的 `run` 或你自己写的 `exp` 脚本），你必须**显式地告诉 Shell 文件在这里**。

* **错误写法：** `program_name` 
    * *结果：* Shell 会报错 `command not found`，因为它只去系统目录找了，没看当前目录。
* **正确写法：** `./program_name`
    * *解释：* `.` 代表当前目录，`/` 是分隔符。意思是“运行位于当前目录下的 program_name”。

---

## 3. 关键知识点：学会阅读手册 (Man Page)

`man` 是 Linux 的内置说明书。做 Pwn 题时，很多 Flag 藏在命令的**特殊参数**里。

* **基本用法：** `man <命令名>` (例如 `man ls`)
* **如何阅读：**
    * `↑` / `↓`：上下滚动。
    * `/`：进入搜索模式（输入关键词后回车）。
    * `n`：跳转到下一个搜索结果 (Next)。
    * `N`：跳转到上一个搜索结果。
    * `q`：退出手册 (Quit)。
* **实战技巧：**
    * 有些题目会把 Flag 藏在以 `-` 开头的参数描述里，遇到题目卡住时，一定要用 `man` 看看这个命令有没有什么冷门的参数（比如 `read_flag --give-me-the-flag` 这种隐藏选项）。