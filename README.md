# 小米商城智能客服

以小米商城电商场景为背景的 Agentic RAG 智能客服系统：商品咨询、售前推荐、订单查询、物流跟踪、售后退换等客服问题，通过「知识库检索 + Agent 工具调用」双链路解决。

- **RAG 问答链路**：问题重写、树形意图识别、多知识库路由、向量召回、RRF 融合、Rerank 精排，回答携带引用来源与原文预览
- **Agent 链路**：ReAct 执行引擎，将知识检索、MCP 工具、技能手册作为工具按需调用；订单类写操作支持人工确认
- **知识库管理**：文档上传至对象存储，解析、分块、向量化经 RocketMQ 异步驱动，入库 Pipeline 可编排、可观测
- **会话记忆**：最近 N 轮消息 + 持久化摘要控制上下文成本，Agent 侧具备跨会话长期记忆
- **模型工程**：模型档位、多候选路由、首包探测、熔断降级，单点模型故障不阻断服务
- **流量保护**：Redis 公平排队与分布式并发控制，突发流量不压垮模型服务
- **管理后台**：仪表盘、知识库 / Chunk 管理、意图树、示例问题、Trace 追踪与配置变更审计

## 技术栈

| 层 | 选型 |
|---|---|
| 后端 | Java 17、Spring Boot 4、MyBatis-Plus、Sa-Token、Redisson |
| 消息 | RocketMQ（事务消息保障分块可靠执行） |
| 存储 | PostgreSQL（业务数据 + pgvector 向量）、Redis、rustfs（S3 兼容对象存储） |
| AI | OpenAI 兼容协议多供应商接入、AgentScope、MCP 协议 |
| 前端 | React 18、TypeScript、Vite、Tailwind CSS |

## 模块结构

前后端分离的模块化单体架构，后端按职责分为七个 Maven 模块：

| 模块 | 职责 |
|---|---|
| `infra-ai` | Chat / Embedding / Rerank / VLM 模型客户端、模型档位、路由与降级 |
| `system` | 用户认证与审计日志 |
| `rag` | RAG 问答、知识库、入库 Pipeline、意图树、检索、会话及管理端 API |
| `agent` | AgentScope ReAct 执行引擎，接入知识检索与 MCP 工具，支持技能与记忆 |
| `bootstrap` | 启动装配层，启动类与主配置 |
| `framework` | 统一响应与异常、幂等、分布式 ID、MQ 适配、SSE 与跨节点流式取消 |
| `mcp-server` | 基于 MCP 协议的客服工具服务：订单、购物车、优惠券、物流、售后、工单 |

## 快速启动

### 1. 环境准备

- JDK 17、Maven 3.9+、Node.js 18+
- PostgreSQL 15+（含 pgvector 扩展）
- Redis 7+
- RocketMQ 5.x（建议 5.3.2+，5.2.0 存在 JVM 崩溃问题）
- rustfs / MinIO 等 S3 兼容对象存储

### 2. 初始化数据库

按顺序执行：

```bash
psql -U postgres -d <db> -f resources/database/schema_pg.sql
psql -U postgres -d <db> -f resources/database/init_data_pg.sql
```

### 3. 配置模型 API Key

通过环境变量注入，不写入配置文件：

```bash
export DEEPSEEK_API_KEY=sk-xxx        # 主对话模型
export BAILIAN_API_KEY=sk-xxx         # Embedding / Rerank / VLM
```

### 4. 启动后端

先启动 mcp-server（9099），再启动主服务（9090），两者存在依赖关系：

```bash
mvn clean package -DskipTests
java -jar mcp-server/target/mcp-server-*.jar
java -jar bootstrap/target/bootstrap-*.jar
```

### 5. 启动前端

```bash
cd frontend
npm install
npm run dev
```

默认管理员账号：`admin / admin`（首次登录后请修改）。

## License

[Apache-2.0](./LICENSE)
