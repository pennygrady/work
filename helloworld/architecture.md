# helloworld 项目架构设计

## 1. 架构概述

本项目为极简单文件 C 程序，无分层、无模块拆分。整体架构：

```
+---------------------------------+
|  helloworld.c                   |
|  +---------------------------+  |
|  |  int main(void)           |  |
|  |    -> printf("helloworld")|  |
|  |    -> return 0            |  |
|  +---------------------------+  |
+---------------------------------+
        |
        v
  C 标准库 (stdio.h)
        |
        v
  操作系统 stdout
```

## 2. 技术选型

| 项目 | 选择 | 理由 |
|------|------|------|
| 语言 | C（C99+） | 需求指定纯 C |
| 编译器 | gcc | 通用、免费、跨平台 |
| 构建方式 | 直接 gcc 命令行编译 | 单文件，无需 Makefile |
| 依赖 | 仅 `stdio.h` | 最小依赖 |

## 3. 编译与运行

```bash
# 编译（带全部警告）
gcc -Wall -o helloworld helloworld.c

# 运行
./helloworld        # Linux/macOS
.\helloworld.exe    # Windows
```

预期输出：

```
helloworld
```

退出码：`0`

## 4. 目录结构

```
helloworld/
├── requirements.md    # 需求文档
├── architecture.md    # 本文件
└── helloworld.c       # 唯一源文件
```

## 5. 设计决策

- **单文件**：需求只要求打印一行字符串，任何多文件/多模块设计都是过度工程。
- **`int main(void)`**：不接收命令行参数，使用标准签名。
- **`puts` vs `printf`**：选择 `printf("helloworld\n")`，语义直白；`puts` 亦可，无实质差别。

## 6. 架构确认

> 项目极简，按任务指示跳过架构确认门，直接进入开发。
