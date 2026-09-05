# Python

仅当受影响仓库路径使用 Python 时，加载本参考。

- 从 `pyproject.toml`、锁文件、工具配置、CI 和仓库 Wrapper 推导 Python 版本及环境/包工作流。不得混用 uv、Poetry、Pipenv、Conda 或裸 pip 约定。
- 遵循既有 package/import 布局和 Format/Lint/类型检查器。避免路径 hack 和意外命名空间/package 变化。
- 为公开边界和复杂逻辑提供准确类型；验证不可信动态数据，不能用宽泛 `Any` 或 Cast 掩盖不确定性。
- 使用 Context Manager 和确定性清理。避免可变默认参数、裸捕获/宽泛静默异常、依赖异常字符串的控制流和丢失 Cause；转换错误时用 Chaining 保留上下文。
- async 只用于合适的 I/O；不得在 Event Loop 上运行阻塞的 CPU/文件/网络操作。限制 Task，传递取消/超时，并收集 Task 失败。
- 处理带时区 datetime、Decimal/金额语义、编码，以及共享状态的进程/线程安全。
- 使用仓库既有 pytest/unittest Fixture 和隔离模型。控制时间、随机数、环境和网络；不得依赖执行顺序或真实外部服务。
