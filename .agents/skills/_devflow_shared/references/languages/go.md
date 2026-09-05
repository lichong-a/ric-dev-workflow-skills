# Go

仅当受影响仓库路径使用 Go 时，加载本参考。

- 遵循 `go`/toolchain 版本、module/workspace 布局、vendor 策略、Build Tag、生成文件和仓库命令。使用 `gofmt` 或项目 Wrapper 格式化改动的 Go 代码。
- 将 `context.Context` 作为第一个参数沿请求调用链传递；不得存入长期存活的 struct。在 I/O 和循环中响应取消与 Deadline。
- 调用方需要错误链时使用 `%w` 添加上下文；通过 `errors.Is` 或 `errors.As` 判断，不比较错误消息。避免每层同时记录并返回同一个错误。
- 每个 goroutine 都必须有所有者、有界创建、取消/退出路径和错误收集。明确保护共享可变状态；并发变化时使用 Race Test 验证。
- 接口保持小而专一，通常由消费方定义；没有真实测试或变化边界时，不要为单一实现抽象接口。
- 关闭 Body、文件和 Rows，并检查迭代/Flush 错误。明确事务、精度/时间、JSON 零值/省略和未知枚举行为。
- 只有在提升清晰度时才使用表驱动测试或 Subtest。避免真实时钟、网络、易波动 sleep 和过度 Mock 内部调用顺序。对受影响模块运行 `go test`，并按适用性运行 Vet/Lint/Race/Build。
