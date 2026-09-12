# Java 与 Kotlin

仅当受影响仓库路径使用 Java 或 Kotlin 时，加载本参考。

- 使用仓库 Maven/Gradle Wrapper 和固定的 JDK/toolchain。遵循模块结构、生成源、Annotation Processor、编译参数，以及 BOM 或 Version Catalog 依赖管理。
- 保持框架 DI、事务、Proxy、序列化和生命周期约定。根据真实运行行为验证 Annotation；self-invocation 和 async/reactive 边界可能绕过 Proxy。
- 显式表达 Nullability 和可选状态。Kotlin 对可达输入避免不安全 `!!`；Java 遵循仓库 Annotation，并使用配置语言级别支持的不可变/record 约定。
- 只在当前层能增加稳定语义时转换异常，保留 Cause，不捕获无法处理的宽泛故障。对外错误保持安全、稳定。
- 定义事务传播/隔离与并发所有权。不得阻塞 Reactive/Coroutine 线程；传递取消，并限制 Executor/Task。
- 审核 JSON/数据库枚举演进、时区、精确金额、ORM Fetch Plan、N+1、Cascade/Orphan 语义和迁移兼容性。
- 通过 Wrapper 运行聚焦测试、静态分析、Format/Lint 和模块构建。使用已有 Test Container/Mock；避免定时 sleep 和共享可变 Fixture。
