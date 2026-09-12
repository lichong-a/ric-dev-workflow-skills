# C# 与 .NET

仅当受影响仓库路径使用 C# 或其他 .NET 语言时，加载本参考。

- 遵循 `global.json`、SDK 风格项目/解决方案布局、Central Package Management、锁定策略、Analyzer、格式规则、目标框架和 Nullable 设置。使用与 CI 一致的仓库命令或 `dotnet` 命令。
- 保持全链路 async；避免 `.Result`、`.Wait()`、隐式 fire-and-forget Task 和丢失异常。接受并传递 `CancellationToken`；限制并行工作，并响应宿主关闭。
- 可靠释放 `IDisposable`/`IAsyncDisposable`。检查 DI 生命周期，确保 Singleton 不捕获 Scoped/Transient 状态，后台服务正确创建 Scope。
- 保持 ASP.NET Middleware 顺序、Model Binding、授权策略、ProblemDetails/错误契约、序列化选项和配置优先级。认证不等于对象级授权。
- 审核 EF Core 查询投影/跟踪、N+1、并发 Token、事务边界、迁移、Decimal/时间语义和取消。
- 不得为通过检查而抑制 Nullable/Analyzer 警告。测试应控制时钟、网络和数据，并使用仓库既有 Fixture/Test Host 模式。
