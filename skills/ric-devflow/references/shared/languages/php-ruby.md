# PHP 与 Ruby

仅当受影响仓库路径使用 PHP 或 Ruby 时，加载本参考。

## PHP

- 从仓库证据推导 PHP 版本、Composer 工作流/锁定策略、Autoload、Extension、框架容器/生命周期、编码规范、静态分析和测试。
- 在兼容前提下使用严格、精确类型；在边界校验动态请求、配置和解码数据。保持框架校验、授权、转义、CSRF、ORM、队列和事务约定。
- 避免动态 include/eval/命令/SQL 构造、不安全反序列化、路径穿越、宽松比较导致的安全缺陷，以及隐式全局/Session 状态。

## Ruby

- 推导 Ruby 版本、Bundler/Gemfile 锁定策略、框架/Autoload 模式、风格、存在时的类型工具、数据库/队列约定和测试命令。
- 保持 Active Record Callback/校验/事务语义，并避免 N+1、Mass Assignment、不安全 Constantize/反序列化、命令/SQL 注入和无界后台重试。
- 元编程应保持局部且有充分理由；安全、数据和公开契约边界优先使用显式行为。

两种语言都应沿用已有 Fixture/Factory 和确定性测试隔离。不得在功能变更中进行生态迁移或替换项目工具。
