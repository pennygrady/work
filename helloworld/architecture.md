# HelloWorld 项目架构设计

- 版本：v1.0
- 日期：2026-10-07
- 输入：requirements.md v1.0（已确认）
- 状态：已确认

## 1. 架构目标

依据需求文档，实现一个**单文件 C11 程序**：编译运行后向标准输出精确打印 `helloworld\n` 并以退出码 0 结束。架构上保持极简、零依赖（仅 libc）、跨平台（Windows MinGW/MSYS2 与 Linux/macOS），并以 Makefile 提供一键构建/运行/清理能力。

## 2. 总体设计

### 2.1 程序结构（单文件）

唯一源文件 `src/main.c`，由两个逻辑部分组成：

| 部分 | 职责 |
| --- | --- |
| 文件头注释 | 标明文件名与用途（满足代码风格约定） |
| 入口函数 `int main(void)` | 程序唯一入口；调用输出实现；显式 `return 0;` |

输出实现：函数体内直接调用标准库 `printf("helloworld\n");`。不抽象辅助函数、不引入全局状态——对单行输出而言任何额外分层都是过度设计。

### 2.2 关键设计决策

| 决策点 | 选择 | 理由 |
| --- | --- | --- |
| 输出方式 | `printf`（而非 `puts`/`write`） | 需求 3.3 明确指定 `printf("helloworld\n");` |
| 头文件 | 仅 `<stdio.h>` | 需求要求仅包含必要头文件；`printf` 声明于此 |
| `main` 签名 | `int main(void)` | C11 标准的无参宿主入口签名，配合 `-Wall -Wextra` 无警告 |
| 退出方式 | 显式 `return 0;` | 满足 F-3 / AC-3（退出码 0），且语义清晰 |
| 换行 | 字符串内显式 `\n` | 保证输出**精确**为 `helloworld\n`（AC-2），不依赖 `puts` 的隐式换行行为 |
| 抽象层级 | 无（main 直接调 printf） | 极简原则，避免过度工程 |

### 2.3 数据流

```
┌──────┐     ┌────────┐     ┌────────┐
│ main │ ──> │ printf │ ──> │ stdout │   (输出: "helloworld\n")
└──────┘     └────────┘     └────────┘
     │
     └──> return 0  (退出码 0 → 操作系统)
```

无输入、无中间状态、无文件/网络 I/O。唯一的流是：进程启动 → `main` → `printf` 写入 stdout 缓冲（行缓冲/全缓冲由 libc 管理，进程正常退出时自动冲刷）→ 返回 0 结束进程。

## 3. 构建设计

### 3.1 构建方式

主构建方式为 **Makefile**（满足 US-2 / F-2）；同时保留**直接 gcc 命令**作为无 make 环境下的降级方案。

编译参数（与需求 3.2 一致）：

| 变量 | 值 | 说明 |
| --- | --- | --- |
| `CC` | `?=` 赋值为 `gcc` | 允许环境覆盖（如 `CC=clang make`，满足 AC-5） |
| `CFLAGS` | `-std=c11 -Wall -Wextra -O2` | C11 标准、全警告、常规优化 |

### 3.2 Makefile 目标

| 目标 | 行为 | 依赖 |
| --- | --- | --- |
| `all`（默认） | 编译 `src/main.c` 生成可执行文件 | `src/main.c` |
| `run` | 先确保已构建，再执行可执行文件 | `all` |
| `clean` | 删除构建产物 | 无 |

要点：
- 可执行文件名 `helloworld`；Windows 下 `clean` 需兼容 `helloworld.exe`（可通过变量或通配处理）。
- 规则带源文件依赖，`make` 仅在源码变化时重编。
- `run`/`clean`/`all` 声明为 `.PHONY`，避免与同名文件冲突。
- `clean` 使用 `rm -f`（Unix）兼容写法；Windows MinGW/MSYS2 环境下 `rm` 可用，无需额外分支。

### 3.3 降级方案：直接 gcc 命令

无 make 环境时等价构建命令：

```sh
gcc -std=c11 -Wall -Wextra -O2 -o helloworld src/main.c
```

## 4. 目录布局

```
work/helloworld/
├── requirements.md    # 需求文档（已存在）
├── architecture.md    # 本文档
├── Makefile           # 构建脚本（all / run / clean）
└── src/
    └── main.c         # 唯一源文件（入口 + 输出）
```

构建后（临时产物，`make clean` 可清除）：

```
work/helloworld/
└── helloworld         # 或 helloworld.exe（Windows）
```

设计说明：
- 源码集中于 `src/`，与文档、构建脚本分层，符合需求 3.3 的目录规范。
- 产物直接落在项目根，便于 `./helloworld` 运行；无需 `bin/` 目录（单产物场景下属于过度分层）。

## 5. 部署 / 运行方案

本项目无外部依赖、无安装步骤，"部署"即"本地构建后直接运行"：

```sh
# 标准流程
make            # 构建（等价 make all）
./helloworld    # 运行 → 输出 helloworld

# 一键流程
make run        # 构建并运行

# 清理
make clean      # 删除可执行产物
```

| 平台 | 环境要求 | 说明 |
| --- | --- | --- |
| Linux / macOS | gcc 或 clang + make | 原生支持 |
| Windows | MinGW / MSYS2（提供 gcc + make + rm） | 产物为 `helloworld.exe` |

验证（对应验收标准）：
- AC-2：运行后 stdout 精确为 `helloworld\n`，可用 `./helloworld | od -c` 或输出重定向比对。
- AC-3：`echo $?`（或 Windows 下 `echo %ERRORLEVEL%`）应为 `0`。
- AC-5：`make CC=clang` 与 `make CC=gcc` 均无警告通过。

## 6. 非功能考量

| 维度 | 结论 |
| --- | --- |
| 可移植性 | 仅用 C11 标准库，无平台特定 API；`\n` 由 libc 处理行尾转换 |
| 可维护性 | 单文件 + Makefile，模板项目可直接复制扩展 |
| 性能 | 忽略（启动即退出）；`-O2` 仅为与需求编译参数一致 |
| 安全性 | 无输入面，无攻击面 |

## 7. 后续阶段指引

实现阶段按本架构落地两个新文件：
1. `src/main.c` —— 按 §2.1/§2.2 的结构与风格约定编写。
2. `Makefile` —— 按 §3.2 的三个目标实现。

不新增其他文件、目录或工具链（requirements.md §6 明确排除 CMake、测试框架等）。
