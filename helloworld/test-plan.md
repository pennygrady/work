# HelloWorld 项目测试计划

- 版本：v1.0
- 日期：2026-10-07
- 输入：requirements.md v1.0、architecture.md v1.0、src/main.c、Makefile
- 状态：待执行
- 说明：本计划仅列出手动验证步骤，不编写测试代码，不引入测试框架（符合 requirements.md §6）。

## 1. 测试范围

| 对应验收标准 | 覆盖内容 |
| --- | --- |
| AC-1 | `make` 编译成功且无警告 |
| AC-2 | 运行输出精确为 `helloworld\n` |
| AC-3 | 程序退出码为 0 |
| AC-4 | `make clean` 完全删除构建产物 |
| AC-5 | GCC 与 Clang 分别以 `-Wall -Wextra` 编译均无警告 |

测试环境：Linux/macOS（gcc + make），或 Windows MinGW/MSYS2。

## 2. 验证步骤

### 步骤 1：编译（AC-1）

```sh
make clean    # 确保从干净状态开始
make
```

**预期结果：**
- 命令执行成功（make 退出码为 0）
- 编译输出中**无任何警告**（`-Wall -Wextra` 下零警告）
- 项目根目录生成可执行文件 `helloworld`（Windows 下为 `helloworld.exe`）

### 步骤 2：运行并比对输出（AC-2）

```sh
./helloworld > out.txt
od -c out.txt        # Linux/macOS；或用文本编辑器查看 out.txt
```

**预期结果：**
- 标准输出**精确**为 `helloworld\n`，即 `od -c` 显示 `h e l l o w o r l d \n`，无其他字符（无额外空格、无回车 `\r`、无多余换行）
- 终端直接运行时仅显示一行 `helloworld`

### 步骤 3：检查退出码（AC-3）

```sh
./helloworld
echo $?              # Linux/macOS
# Windows cmd:  helloworld.exe 然后 echo %ERRORLEVEL%
```

**预期结果：**
- 输出 `0`，即程序以退出码 0 正常结束

### 步骤 4：一键运行（可选补充，对应 F-2 / `make run`）

```sh
make clean
make run
```

**预期结果：**
- 自动完成编译并运行，终端输出一行 `helloworld`

### 步骤 5：清理（AC-4）

```sh
make clean
ls                   # 或 dir（Windows）
```

**预期结果：**
- `helloworld` / `helloworld.exe` 被完全删除
- 目录中仅剩 `requirements.md`、`architecture.md`、`test-plan.md`、`Makefile`、`src/`
- 再次执行 `make clean` 不报错（`rm -f` 幂等）

### 步骤 6：双编译器无警告（AC-5，可选，视环境是否安装 clang）

```sh
make clean
make CC=gcc          # 观察无警告
make clean
make CC=clang        # 观察无警告
```

**预期结果：**
- GCC 与 Clang 分别编译均成功，且 `-Wall -Wextra` 下均无警告

## 3. 通过标准

- **必须全部通过（P0）**：步骤 1、2、3、5 —— 对应 AC-1～AC-4，为本次测试的通过门槛。
- **环境允许时通过**：步骤 6 —— 对应 AC-5；若测试环境未安装 clang，记录为 "跳过（环境缺失）"，不视为失败。
- **可选**：步骤 4 —— 验证 `make run` 便捷目标，失败不阻断，但应记录并修复。

任一必须步骤未达预期结果，则本次测试判定为**不通过**，需修复后从头重新执行全部步骤。

## 4. 记录模板

| 步骤 | 环境（OS/编译器版本） | 结果（通过/失败/跳过） | 备注 |
| --- | --- | --- | --- |
| 1. 编译 |  |  |  |
| 2. 输出比对 |  |  |  |
| 3. 退出码 |  |  |  |
| 4. make run |  |  |  |
| 5. make clean |  |  |  |
| 6. 双编译器 |  |  |  |
