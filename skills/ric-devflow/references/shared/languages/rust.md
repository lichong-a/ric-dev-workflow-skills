# Rust

仅当受影响仓库路径使用 Rust 时，加载本参考。

- 遵循 `rust-toolchain`、Cargo workspace/Feature/Target、锁文件策略、生成代码和 MSRV。使用仓库命令，并对受影响 Feature/Target 运行 `cargo fmt`、适用的 Clippy、测试和构建。
- 用 `Option`/`Result` 表达可达的缺失/失败，并提供有意义错误上下文。运行时路径避免无说明的 `unwrap`、`expect`、panic 或有损错误转换。
- 保持 Ownership/Lifetime 简洁，避免不必要的 Clone/分配。必须使用 `unsafe` 时，将其最小化并封装，记录安全不变量并测试边界；不得盲目复制不安全本地模式。
- 匹配项目选择的 async runtime。不得阻塞 Executor 线程；没有证明时不要跨 `await` 持锁；不得分离无所有者 Task 或忽略取消。限制 Channel/Task 并处理关闭。
- 按适用性审核 `Send`/`Sync`、锁顺序、整数溢出、解析边界、路径/命令构造、序列化兼容性、Feature 组合和 FFI 所有权。
- 测试应确定性覆盖错误和并发；运行相关 Feature Matrix，但不得改变 workspace 预期的默认 Feature 语义。
