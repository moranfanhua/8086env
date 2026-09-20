自用，顺带可能给有需要学这种老古董的人

~多半是被学校逼的吧~

/BIN 里面是学校给我的比古董更老的古董。

---
# 8086 Assembly Environment for VS Code

一个基于 **VS Code + MASM 5.10 + DOSBox-X** 的 8086 汇编开发环境。

配置完成后，可以直接在 VS Code 中编写 `.asm` 文件，并通过 VS Code Task 自动调用 DOSBox-X 完成：

```text
ASM 源代码
    ↓
MASM 5.10
    ↓
OBJ 目标文件
    ↓
LINK 3.64
    ↓
EXE
    ↓
DOSBox-X 运行
```

## 1. 安装 VS Code

首先安装 [Visual Studio Code](https://code.visualstudio.com/)。

安装完成后，在 VS Code 扩展商店搜索：

```text
MASM
```

安装一个支持 MASM 语法高亮的扩展即可。

> MASM 扩展主要用于提供语法高亮等编辑功能。  
> 实际的汇编和链接仍然由本项目中的 **MASM 5.10** 和 **LINK** 在 DOSBox-X 中完成。

---

## 2. 下载本项目

### 方法一：Git Clone

```shell
git clone https://github.com/moranfanhua/8086env.git
```

### 方法二：直接下载

也可以直接下载整个仓库的 ZIP 文件，然后解压到任意位置。

例如：

```text
C:\Users\YourName\Documents\8086
```

假设项目目录结构如下：

```text
8086
├── .vscode
│   └── tasks.json
├── BIN
│   └── DEBUG.COM
├── MASM51
│   ├── MASM.EXE
│   ├── LINK.EXE
│   └── INCLUDE
└── CODE
    └── test.asm
```

其中：

- `MASM51`：MASM 5.10 及相关工具
- `BIN`：DEBUG 等 DOS 工具
- `CODE`：存放自己编写的 `.asm` 源代码
- `.vscode`：VS Code Task 配置

---

## 3. 安装 DOSBox-X

下载并安装：

[https://dosbox-x.com/](https://dosbox-x.com/)

建议使用 Windows 64 位版本。

安装完成后，记住 `dosbox-x.exe` 的实际位置。

例如：

```text
C:\DOSBox-X\dosbox-x.exe
```

后面配置 VS Code Task 时需要使用这个路径。

---

## 4. 配置 DOSBox-X 启动脚本

启动 DOSBox-X。

依次进入：

```text
主菜单
→ 配置文件
→ AUTOEXEC.BAT
```

在 AUTOEXEC.BAT 中加入：

```bat
mount c <你的项目文件夹路径>
c:
set PATH=C:\MASM51;C:\BIN
set INCLUDE=C:\MASM51\INCLUDE
cd \CODE
```

例如项目位于：

```text
C:\Users\Moxia\Documents\8086
```

则填写：

```bat
mount c C:\Users\Moxia\Documents\8086
c:
set PATH=C:\MASM51;C:\BIN
set INCLUDE=C:\MASM51\INCLUDE
cd \CODE
```

其中：

```text
mount c ...                 将项目目录挂载为 DOS 的 C:
set PATH=C:\MASM51;C:\BIN   配置 MASM、LINK、DEBUG 等工具
set INCLUDE=...             配置 MASM Include 文件路径
cd \CODE                    启动后自动进入源代码目录
```

配置完成后点击：

```text
保存
→ 保存并重新启动
```

重新启动 DOSBox-X 后，正常情况下应该自动进入：

```text
C:\CODE>
```

可以输入：

```dos
masm
```

检查 MASM 是否可用。

正确情况下会显示：

```text
Microsoft (R) Macro Assembler Version 5.10
```

---

## 5. 配置 VS Code Task

使用 VS Code 打开整个 `8086` 项目目录。

打开：

```text
.vscode\tasks.json
```

> 注意文件名是 `tasks.json`，不是 `task.json`。

找到：

```json
"command": "C:\\DOSBox-X\\dosbox-x.exe"
```

将其修改为你电脑上 `dosbox-x.exe` 的实际路径。

例如 DOSBox-X 安装在：

```text
D:\Program Files\DOSBox-X\dosbox-x.exe
```

则应该写成：

```json
"command": "D:\\Program Files\\DOSBox-X\\dosbox-x.exe"
```

注意 JSON 中 Windows 路径的 `\` 需要写成：

```text
\\
```

---

## 6. 测试环境

项目提供了测试程序：

```text
CODE\test.asm
```

示例程序：

```asm
DATA SEGMENT
    MSG DB 'Hello, 8086!', 0DH, 0AH, '$'
DATA ENDS

STACK SEGMENT STACK
    DW 128 DUP(?)
STACK ENDS

CODE SEGMENT
    ASSUME CS:CODE, DS:DATA, SS:STACK

START:
    MOV AX, DATA
    MOV DS, AX

    MOV DX, OFFSET MSG
    MOV AH, 09H
    INT 21H

    MOV AH, 4CH
    INT 21H

CODE ENDS
END START
```

在 VS Code 中打开：

```text
CODE\test.asm
```

然后按：

```text
Ctrl + Shift + P
```

输入：

```text
Tasks: Run Task
```

选择对应的 8086 Build & Run Task。

如果配置正确，VS Code 会自动启动 DOSBox-X，并依次完成：

```text
MASM test.asm
      ↓
TEST.OBJ
      ↓
LINK test.obj
      ↓
TEST.EXE
      ↓
运行 TEST.EXE
```

最终 DOSBox-X 中应该显示：

```text
Hello, 8086!
C:\CODE>
```

这说明环境配置成功。

---

## 7. 手动编译

除了使用 VS Code Task，也可以直接在 DOSBox-X 中手动编译。

进入：

```text
C:\CODE>
```

执行：

```dos
masm test.asm;
link test.obj;
test
```

其中：

```text
masm test.asm;   汇编 ASM → OBJ
link test.obj;   链接 OBJ → EXE
test             运行 TEST.EXE
```

注意命令末尾的 `;`。

它可以让 MASM/LINK 直接采用默认输出文件名，而不再逐项询问输出文件名称。

---

## 8. 使用 DEBUG 调试程序

项目中的 `BIN` 目录包含 DOS DEBUG 工具。

编译完成后可以执行：

```dos
debug test.exe
```

进入 DEBUG 后，常用命令包括：

```text
r       查看寄存器
u       反汇编
t       单步执行一条指令
g       连续运行程序
d       查看内存
q       退出 DEBUG
```

例如：

```dos
C:\CODE>debug test.exe
-r
```

可以查看：

```text
AX
BX
CX
DX
SP
BP
SI
DI
DS
ES
SS
CS
IP
FLAGS
```

使用：

```text
-t
```

可以逐条执行 8086 指令并观察寄存器变化。

---

## 9. 编写自己的程序

建议将自己的 `.asm` 文件统一放在：

```text
CODE
```

例如：

```text
CODE
├── test.asm
├── hello.asm
├── lab1.asm
└── lab2.asm
```

如果编写：

```text
hello.asm
```

则手动编译：

```dos
masm hello.asm;
link hello.obj;
hello
```

或者直接使用 VS Code Task 编译运行当前打开的 ASM 文件。

---

## 常见问题

### `mount` 提示“错误的命令或文件名”

如果 DOSBox-X 启动后已经显示：

```text
C:\CODE>
```

说明 AUTOEXEC.BAT 已经完成挂载，不需要再次手动执行 `mount`。

---

### LINK 提示 `warning L4021: no stack segment`

说明程序没有定义 STACK 段。

可以加入：

```asm
STACK SEGMENT STACK
    DW 128 DUP(?)
STACK ENDS
```

并在代码段中声明：

```asm
ASSUME CS:CODE, DS:DATA, SS:STACK
```

---

### 输出后命令提示符没有换行

例如出现：

```text
Hello, 8086!C:\CODE>
```

是因为 DOS 的字符串输出不会自动换行。

将：

```asm
MSG DB 'Hello, 8086!$'
```

改成：

```asm
MSG DB 'Hello, 8086!', 0DH, 0AH, '$'
```

其中：

```text
0DH = CR（Carriage Return）
0AH = LF（Line Feed）
```

即可正常换行。

---

### 如何确认使用的是 MASM 5.10？

在 DOSBox-X 中执行：

```dos
masm
```

应该显示：

```text
Microsoft (R) Macro Assembler Version 5.10
```

如果显示的是其他版本，请检查：

```bat
set PATH=C:\MASM51;C:\BIN
```

`C:\MASM51` 必须放在 `C:\BIN` 前面。

---

## 最终开发流程

配置完成后，日常使用只需要：

```text
1. 使用 VS Code 打开项目
        ↓
2. 在 CODE 中编写 .asm
        ↓
3. 保存文件
        ↓
4. Ctrl + Shift + P
        ↓
5. Tasks: Run Task
        ↓
6. 选择 8086 Build & Run
        ↓
7. DOSBox-X 自动完成汇编、链接和运行
```

也可以在 DOSBox-X 中手动执行：

```dos
masm xxx.asm;
link xxx.obj;
xxx
```

需要调试时：

```dos
debug xxx.exe
```

至此，**VS Code + MASM 5.10 + LINK + DEBUG + DOSBox-X** 的 8086 汇编开发环境配置完成。