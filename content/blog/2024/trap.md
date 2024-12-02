---
title: shell 脚本中的 `trap` 命令
tags:
  - shell
  - trap
date: 2024-12-02
---

shell 脚本中有类似编程里的 defer, finally, register_shutdown_function 处理效果吗？有的，就是 `trap` 命令。

## 1. 什么是 `trap`？

`trap` 是 Shell 提供的一个内置命令，用于捕获 UNIX 信号（Signals）或者特殊的运行时事件，并在触发时执行特定命令。它的基本用法如下：

```bash
trap 'commands' signals
```

- **`commands`**：在捕获指定信号或事件时执行的一段命令。
- **`signals`**：触发 `commands` 执行的信号列表，可以是信号名称（如 `EXIT` 或 `INT`）或信号编号（如 `15` 或 `2`）。

通过 `trap` 命令，我们可以在脚本运行的期间甚至退出时，优雅地完成清理、日志记录以及错误处理等任务。

---

## 2. 常见信号与作用

在 UNIX 系统中，信号是用于与进程通信的机制。下表列出了常见信号以及它们的作用：

| 信号名称    | 信号编号 | 描述                                                   | 触发方式                       |
|-------------|-----------|-------------------------------------------------------|--------------------------------|
| `SIGHUP`    | `1`       | 挂起信号，通常在终端关闭时发送。                           | 关闭 SSH 会话或者挂起终端       |
| `SIGINT`    | `2`       | 中断信号，一般由用户按下 `Ctrl+C` 产生。                  | 用户主动中断                  |
| `SIGQUIT`   | `3`       | 退出信号，通常由用户按下 `Ctrl+\` 产生，用于生成调试 core dump。 | 强制退出                      |
| `SIGTERM`   | `15`      | 终止信号，通常通过 `kill` 命令发送，表示希望优雅终止。      | 执行 `kill PID`            |
| `SIGKILL`   | `9`       | 强制终止信号，无法被捕获，是一个直接杀死进程的信号。          | 执行 `kill -9 PID`        |
| `SIGSTOP`   | `19`      | 暂停信号，无法被捕获或忽略，用于暂停进程。                   | 暂停的快捷方式               |
| `EXIT`      | -         | 在脚本退出时触发（无论正常还是错误）。                      | 脚本执行完成或者使用 `exit` 语句|

> 你可以通过 `kill -l` 查看当前系统支持的完整信号列表。

---

## 3. 基本用法

### 示例 1：清理临时文件

当脚本中需要执行一些中间操作时，可能会产生临时文件。如果用户中途中断脚本，这些临时文件可能会遗留在文件系统中。例如：

```bash
#!/bin/bash

temp_file="/tmp/my_temp_file"
trap "rm -f $temp_file; echo '临时文件已清理'" EXIT

# 模拟创建临时文件并操作
touch $temp_file
echo "处理中..."
sleep 5

# 脚本结束后，自动触发清理
echo "执行完成。"
```

运行后，无论脚本是成功执行还是中途退出（如被 `Ctrl+C` 中断），`trap` 都会捕获 `EXIT` 信号，执行清理命令 `rm -f $temp_file`。

---

### 示例 2：捕获用户中断信号 (`SIGINT`)

有时候我们希望让用户按下 `Ctrl+C` 时，脚本能优雅退出，而非直接中止。例如：

```bash
#!/bin/bash

trap "echo '优雅退出中...'; cleanup; exit" SIGINT

cleanup() {
    echo "正在清理资源..."
}

echo "运行脚本，按 Ctrl+C 中断。"
while true; do
    sleep 1
done
```

在脚本中，用户按下 `Ctrl+C` 会发送 `SIGINT` 信号。通过 `trap`，可以拦截该信号并优雅退出，同时完成清理操作。

---

### 示例 3：错误处理 (`ERR` 信号)

在 Shell 脚本中，如果某个命令失败，往往需要及时处理或终止脚本以避免影响后续操作。通过 `trap` 捕获 `ERR` 信号，可以实现全局错误处理：

```bash
#!/bin/bash
set -e  # 任何命令失败时立即退出
trap "echo '发生错误，终止脚本'; cleanup; exit 1" ERR

cleanup() {
    echo "清理完成。"
}

# 模拟正常命令
echo "开始执行脚本..."
ls /nonexistent_folder  # 强制触发错误
echo "这条命令不会被执行。"
```

当某条命令（如 `ls /nonexistent_folder`）出错时，会触发 `ERR` 信号，调用 `cleanup` 函数清理资源并退出脚本。

---

## 4. 高级用法

### 捕获多个信号

你可以通过 `trap` 同时捕获多个信号，并为它们定义相同的行为：

```bash
trap "echo '收到信号，正在优雅退出...'; exit" SIGINT SIGTERM
```

无论是按下 `Ctrl+C`（触发 `SIGINT`）还是通过 `kill` 命令（触发 `SIGTERM`），都会执行同样的命令。

---

### 忽略某些信号

如果希望某些信号不影响脚本运行，可以让 `trap` 执行空操作：

```bash
trap "" SIGINT SIGTERM
echo "信号被忽略，按 Ctrl+C 无法中断脚本。"
while true; do
    sleep 1
done
```

需要注意的是，部分信号（例如 `SIGKILL` 和 `SIGSTOP`）无法被忽略或捕获。

---

### 调试脚本流（`DEBUG` 信号）

`trap` 的 `DEBUG` 信号使你可以在脚本执行每条命令之前插入调试信息，非常适用于调试复杂脚本。

```bash
trap 'echo "正在执行命令：$BASH_COMMAND"' DEBUG

echo "执行第一步"
ls /nonexistent_file # 触发错误
echo "执行下一步"
```

当脚本运行时，每条命令（包括未定义的或出错的命令）都会在执行前打印到屏幕上，便于跟踪脚本行为。

---

### 捕获函数的返回状态（`RETURN` 信号）

当 Shell 函数返回时，可以通过捕获 `RETURN` 信号执行额外逻辑，例如记录函数的返回值或完成特定清理操作。

```bash
trap 'echo "函数返回了。"' RETURN

test_function() {
    echo "正在执行函数任务..."
}
test_function
```

---

## 5. 潜在问题与注意事项

1. **部分信号无法捕获**：`SIGKILL` 和 `SIGSTOP` 是特殊信号，无法被 `trap` 捕获或阻止。这是 UNIX 系统的设计策略，无法绕过。

2. **并发信号处理**：当多个信号几乎同时触发时，Shell 可能会按顺序依次处理它们，但这可能引发意外行为，建议测试关键脚本。

3. **复杂命令需注意转义**：`trap` 中的命令如果包含 `$`、`"` 等特殊字符，可能需要正确转义。例如：
   ```bash
   trap 'echo "命令失败: $BASH_COMMAND"' ERR
   ```

---

## 6. 总结

`trap` 是 Shell 脚本中功能强大且不可或缺的工具。无论是资源清理、安全退出、错误处理还是调试跟踪，它都能为我们的脚本带来极大的灵活性与可靠性。

通过结合信号特性与实际需求，合理设计脚本中的信号处理逻辑，可以避免意外问题的发生，提升脚本的健壮性和安全性。

