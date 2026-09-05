# JavaScript 与 TypeScript

仅当受影响仓库路径使用 JavaScript 或 TypeScript 时，加载本参考。

- 根据锁文件/workspace 确定包管理器，并使用仓库脚本；不得混用包管理器或手改依赖解析区块。
- 保持实际运行时和模块目标：Node 与浏览器/Worker、ESM 与 CJS、Server 与 Client Component，以及受支持的浏览器/运行时版本。检查 package `exports`、Bundler、测试运行器和生成类型。
- 保持 TypeScript 严格度。用类型表达合法状态；在运行时收窄未知外部 JSON、环境和输入；区分可选/null/空值；避免无依据的 `any`、断言、`@ts-ignore` 或悬空 Promise。
- 每个 Promise 都必须被 await、return 或显式处理。限制并发，传递取消/Abort，防止过期响应和卸载后更新，并释放 Timer/Listener/Subscription。
- 保持框架路由、渲染和数据缓存语义。没有证据与测试时，不得让代码跨越 Server/Client 或 Edge/Node 边界。
- 安全方面避免动态构造代码/命令、不安全 HTML、原型污染、路径穿越，以及在客户端 Bundle/日志中暴露 secret。
- 使用已有 Format/Lint/Typecheck/Test/Build 命令。测试应按需控制时钟、随机数和网络，并断言行为而不是模块调用编排。
