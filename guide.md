# MaoRpc 学习与实践指南

## 1. 项目定位与难点综述
- **定位**：基于 Netty + Kryo + Zookeeper 的教学型 RPC 框架，强调“像调用本地方法一样调用远程服务”的端到端链路。项目提供了完整示例（接口、服务端、客户端）并配套文档，适合作为进阶练手和毕设项目。【F:README.md†L25-L67】
- **核心难点**
  1. **高性能通信**：用 Netty 替换 BIO 套接字，需理解 Channel 复用、粘包拆包和心跳保活等细节。【F:README.md†L29-L36】【F:README.md†L88-L105】
  2. **协议与序列化**：自定义消息头（魔数、序列化器编号、长度等）+ Kryo/Protostuff/Hessian 序列化，既保证安全性又兼顾兼容性与扩展性。【F:README.md†L31-L36】【F:README.md†L101-L105】
  3. **注册发现与负载均衡**：依赖 Zookeeper 做服务注册/订阅，再通过随机或一致性哈希等策略挑选服务实例，确保水平扩展能力。【F:README.md†L31-L33】【F:README.md†L94-L95】
  4. **Spring 集成与 SPI 扩展**：`@RpcService`、`@RpcReference`、`@RpcScan` 注解让服务自动装配，同时 SPI 机制使序列化、负载均衡等组件可插拔，这是二次开发的基础。【F:README.md†L33-L36】【F:README.md†L253-L265】【F:README.md†L99-L100】

## 2. 核心链路速览
1. **服务发布**：服务实现类用 `@RpcService` 标记并指定 `group`/`version`，Spring 启动时借助 `@RpcScan` 扫描并注册到 Zookeeper，对应地址挂在注册中心节点下。【F:README.md†L166-L205】【F:README.md†L253-L255】
2. **服务发现**：客户端应用在启动时同样使用 `@RpcScan`，框架读取 `@RpcReference` 注解，基于接口名 + 分组 + 版本向注册中心拉取可用服务列表。【F:README.md†L207-L243】【F:README.md†L253-L255】
3. **请求发起**：动态代理拦截调用，封装 `RpcRequest`，通过 Netty Channel 发出，协议头会带魔数、序列化器编号、消息长度；负载均衡器从候选节点中挑选目标地址。【F:README.md†L73-L105】
4. **服务端处理**：Netty 解析请求后执行目标实现类，结果经序列化写回；心跳和 Channel 复用保证连接存活，`CompletableFuture` 异步解耦发送与响应。【F:README.md†L91-L105】
5. **结果回传**：客户端的 `CompletableFuture` 在收到响应后完成，代理将结果返回给业务调用方，整条链路对用户透明，体验类似本地方法调用。【F:README.md†L91-L105】

> **排错建议**：若调用失败，按“注册中心 → 客户端代理 → 序列化/协议 → 服务实现”逆序定位；充分利用心跳日志和 Zookeeper 节点信息判断连通性。

## 3. 快速使用步骤
1. **环境准备**：JDK 8+、Maven 3.6+、Zookeeper 3.5.8+。【F:README.md†L112-L117】
2. **启动 Zookeeper**：`docker run -d --name zookeeper -p 2181:2181 zookeeper:3.5.8`。【F:README.md†L120-L129】
3. **构建项目**：`git clone` 仓库后执行 `mvn clean install`，会打包公共模块、框架核心及示例。【F:README.md†L131-L137】
4. **定义接口**：在 `hello-service-api` 中创建接口与 DTO（示例 `HelloService`、`Hello`）。【F:README.md†L139-L160】
5. **实现与暴露服务**：示例服务实现位于 `example-server`，用 `@RpcService(group="test1", version="version1")` 标注，并在 `NettyServerMain` 中 `autoRegistry()` 启动 Netty 服务端。【F:README.md†L166-L205】
6. **消费服务**：客户端在 `example-client` 中通过 `@RpcReference` 注入代理，`NettyClientMain` 启动后即可像本地方法一样调用 `helloService.hello()`。【F:README.md†L207-L243】
7. **运行顺序**：先启动 Zookeeper，再启动 `NettyServerMain`，最后运行 `NettyClientMain`。【F:README.md†L245-L249】

## 4. 建议的学习路径
1. **结构认知**：按照 README 的目录图从 `rpc-framework-common` → `rpc-framework-simple` → 示例模块逐步阅读，理解公共枚举、协议对象、网络层、代理层的分布。【F:README.md†L38-L50】
2. **模块聚焦**：优先掌握 README 列举的优化点：Netty 通信、Kryo 序列化、Zookeeper 注册、心跳、负载均衡、版本/分组、SPI 等；这些模块是面试与实战的重点。【F:README.md†L88-L109】
3. **动手实验**：跟随 README 的“运行项目”章节动手跑通示例，通过断点观察请求发送/接收链路，理解代理、序列化与 Channel 交互细节。【F:README.md†L120-L249】
4. **扩展实践**：选择 README 中的“可优化点”作为练习，例如“可配置化序列化/注册中心”“协议头扩展”“增加监控中心”等，提交 PR 争取被合入。【F:README.md†L100-L108】
5. **知识储备**：根据 README 的建议补齐 Java 动态代理、序列化框架、线程池、`CompletableFuture`，以及 Netty/Zookeeper 的核心概念，打牢理论基础。【F:README.md†L278-L299】

## 5. 写在简历上的表达方式
- **项目一句话亮点**：
  > 手写基于 Netty + Zookeeper 的高性能 RPC 框架，支持多序列化、可插拔负载均衡、服务分组/版本与 Spring 注解自动注册，延迟可控、链路透明。【F:README.md†L29-L37】【F:README.md†L94-L105】【F:README.md†L166-L255】
- **职责/成果示例**：
  1. 负责自定义协议设计与 Netty 编解码实现（魔数校验、序列化器编号、消息体长度），降低异常请求风险并提升诊断效率。【F:README.md†L101-L105】
  2. 基于 Curator 对接 Zookeeper，实现服务注册发现与随机/一致性哈希负载均衡，支撑服务多实例部署。【F:README.md†L31-L33】【F:README.md†L94-L95】
  3. 通过 `@RpcService/@RpcReference/@RpcScan` + SPI 提供组件化扩展能力，简化业务接入成本并方便替换序列化/注册中心实现。【F:README.md†L33-L36】【F:README.md†L99-L100】【F:README.md†L253-L265】
- **成果量化建议**：描述吞吐/延迟对比（如“相比 BIO 版本延迟降低 xx%”）、可用性（心跳 + Channel 复用减少重连次数）、可扩展性（新增序列化插件成本）。

## 6. 如何贡献 PR
1. Fork 仓库到个人账号，本地 clone 并创建特性分支。
2. 在 `rpc-framework-simple` 或其它模块实现功能/修复 bug，确保遵循 README 列出的可优化方向。
3. 补充必要的文档（如 `docs/`、`guide.md`）和示例，验证示例工程可运行。
4. 提交规范 commit，Push 后发起 PR，详细描述修改点与测试方式，方便 Maintainer 快速 Review。【F:README.md†L51-L52】

> **小贴士**：在 PR 描述中关联“可优化点”清单中的条目，更容易体现贡献价值。
