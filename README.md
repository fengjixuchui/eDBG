<div align="center">
  <img src="logo.png"/>

  [![GitHub Release](https://img.shields.io/github/v/release/ShinoLeah/eDBG?style=flat-square)](https://github.com/ShinoLeah/eDBG/releases)
  [![License](https://img.shields.io/github/license/ShinoLeah/eDBG?style=flat-square)](LICENSE)
  [![Platform](https://img.shields.io/badge/platform-Android%20ARM64-red.svg?style=flat-square)](https://www.android.com/)
  ![GitHub Repo stars](https://img.shields.io/github/stars/ShinoLeah/eDBG)

  简体中文 | [English](README_EN.md)
</div>

> eDBG 是一款基于 eBPF 的轻量级 CLI 调试器。<br />
>
> 相比于传统的基于 ptrace 的调试器方案，eDBG 不直接侵入或附加程序，具有较强的抗干扰和反检测能力。

## ✨ 特性

- 基于 eBPF 实现，基本无视反调试。
- 支持常规调试功能（详见“命令详情”）
- 使用类似 [pwndbg](https://github.com/pwndbg/pwndbg) 的 CLI 界面和类似 GDB 的交互方式，简单易上手
- 基于文件+偏移的断点注册机制，可以快速启动，支持多线程或多进程调试。

## 💕 演示

![](demo.png)

## 🚀 运行环境

- 目前仅支持 ARM64 架构的 Android 系统，需要 ROOT 权限，推荐搭配 [KernelSU](https://github.com/tiann/KernelSU) 使用
- 系统内核版本5.10+ （可执行`uname -r`查看）

## ⚙️ 使用

1. 下载最新 [Release](https://github.com/ShinoLeah/eDBG/releases) 版本

2. 推送到手机的`/data/local/tmp`目录下，添加可执行权限

   ```shell
   adb push eDBG /data/local/tmp
   adb shell
   su
   chmod +x /data/local/tmp/eDBG
   ```

3. 运行调试器

   ```shell
   ./eDBG -p com.pakcage.name -l libname.so -b 0x123456
   ```

   | 选项名称          | 含义                               |
   | ----------------- | ---------------------------------- |
   | -p                | 目标应用包名                       |
   | -l                | 目标动态库名称                     |
   | -b                | 初始断点偏移列表（逗号分隔）       |
   | -t                | 线程名称过滤器（逗号分隔）         |
   | -i filename       | 使用配置文件                       |
   | -s                | 保存进度到使用的配置文件           |
   | -o filename       | 保存进度到指定文件名（与 -s 冲突） |
   | -hide-register    | 禁用寄存器信息输出                 |
   | -hide-disassemble | 禁用反汇编代码输出                 |

3. 运行被调试 APP

   > eDBG 也可以直接附加正在运行的 APP，但 eDBG 不会主动拉起被调试 APP。

## ⚠️ 注意

- 由于本项目使用基于文件+偏移的断点注册机制，在调试系统库（`libc.so`、`libart.so`）时可能会比较卡顿。
- 本项目不能随时暂停被调试程序，因此如果没有可用的断点，该项目可能无法调试目标程序。
- 该项目可以在目标程序运行之前被运行，因此不支持在启动时指定线程 id。
- 最多支持 20 个启用的断点。

## 💡命令说明

- **断点** `break / b`

  - 偏移：`b 0x1234`（相对初始动态库的偏移）
  - 内存地址：`b 0x6e9bfe214c`（需要当前程序正在运行）
  - 库名+偏移：`b libraryname.so+0x1234`
  - 当前偏移：`b $+1`，（当前位置+**指令条数**）
  - 启用断点：`enable id`，启用指定断点（你可以在 `info` 中查看断点信息）
  - 禁用断点：`disable id`，禁用指定断点
  - 删除断点：`delete id`，删除第 id 号断点

- **继续运行** `continue / c`：继续执行至下一断点

- **单步调试**

  - `step / s` 单步步入（进入函数调用）
  - `next / n` 单步步过（跳过函数调用）

- **退出函数** `finish / fi`：执行直到当前函数退出

- **运行直到** `until / u`：运行直到指定地址。地址的指定方法与断点相同

- **查看内存** `examine / x`

  - 地址：`x 0x12345678`（默认长度 16）
  - 地址+长度：`x 0x12345678 128`
  - 寄存器：`x X0`，查看对应寄存器地址对应的内存(`[X0]`)
  - 寄存器+长度：`x X0 128` 

- **展示内存** `display / disp`

  - 地址：`disp 0x123456`，(每次触发断点或单步时打印)

  - 地址+长度：`disp 0x123456 128`

  - 地址+长度+变量名：`disp 0x123456 128 name`，展示同时打印该变量名

    > ⚠️ 若内存地址变化（e.g. 应用重启），此功能将无法输出正确信息。

- **取消展示内存**`undisplay / undisp <id>`：取消展示第 id 号变量

- **写内存** `write 0x1235 62626262`：向指定地址写入 Hex String，地址指定方法与 `examine` 相同。

- **退出** `quit / q`：退出**调试器**（不会影响程序运行）

- **查看代码** `list / l / disassemble / dis`

  - 直接查看：`l`，打印当前 PC 位置开始 10 条指令
  - 查看指定地址：`l 0x1234`，打印对应内存地址 10 条指令
  - 查看指定地址指定长度指令：`l 0x1234 20`，打印对应内存地址对应**指令条数**的指令

- **查看信息** `info / i`

  - `info b/break`：列出当前所有断点（`[+]`=已启用，`[-]`=未启用）
  - `info register/reg/r`：查看所有寄存器信息。
  - `info thread/t`：列出当前所有线程和已设定的线程过滤器。

- **线程相关** `thread / t`

  - `t`：列出所有可用线程。
  - `t + 0`：增加线程过滤器在第 0 个线程（使用`info t`查看所有线程 id），注意不是指定 `tid`
  - `t - 0`：取消第 0 个线程过滤器
  - `t all`：删除所有线程过滤器。
  - `t +n threadname`：增加线程名称过滤器。

- **设置符号** `set address name`：设置指定地址符号。

- **重复上一条指令**：直接回车


## 🛫 编译

1. 环境准备

   本项目在 x86 Linux 下交叉编译

   ```shell
   sudo apt-get update
   sudo apt-get install golang==1.18
   sudo apt-get install clang==14
   export GOPROXY=https://goproxy.cn,direct
   export GO111MODULE=on
   ```

2. 编译

   ```shell
   git clone --recursive https://github.com/ShinoLeah/eDBG.git
   ./build_env.sh
   make
   ```

## 💭 实现原理

- 基本是简单的基于 uprobe 和 SIGSTOP / SIGCONT 的简易调试...填坑中

## 🧑‍💻 To Do

- frame 功能
- backtrace 功能
- watch 功能

## 🤝 参考

- [SeeFlowerX/stackplz](https://github.com/SeeFlowerX/stackplz/tree/dev)
- [pwndbg](https://github.com/pwndbg/pwndbg)

## ❤️‍🩹 其他

- 喜欢的话可以点点右上角 Star 🌟
- 欢迎提出 Issue 或 PR！
