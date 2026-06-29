# 任务：修复 Rust 项目 logstat 的所有 Bug

你面前是一个 Rust CLI 工具 `logstat`（日志文件分析器），由一位实习生编写。
这个工具用于解析日志文件、按级别过滤、统计数量、彩色输出。

代码有 **10 个 Bug**，分三类：

1. **编译错误**（3 个）— `cargo build` 无法通过
2. **逻辑错误**（4 个）— 编译能过但运行结果不对，`cargo test` 会失败
3. **代码质量问题**（3 个）— 能跑但代码很烂，`cargo clippy` 会报警告

## 你的任务

按顺序完成以下三步：

1. **让 `cargo build` 通过** — 修复所有编译错误
2. **让 `cargo test` 全部通过** — 修复所有逻辑 Bug（共 10 个测试）
3. **让 `cargo clippy` 零警告** — 修复代码质量问题

## 约束

- 不要改变程序的功能和公开 API 行为
- 不要删除或修改测试用例
- 不要添加外部依赖（保持 Cargo.toml 只有标准库）
- 保留所有注释

## 验证

完成后依次运行：
```bash
cargo build
cargo test
cargo clippy -- -D warnings
```

三个命令都通过即为完成。
