# yu-ai-code-mother 项目深度分析报告

> 分析时间：2026-03-24
>
> 项目地址：https://github.com/Lubida123/yu-ai-code-mother

---

## 一、项目概述

**yu-ai-code-mother** 是一个企业级 **AI 零代码应用生成平台**，由程序员鱼皮主导开发，定位为教学级全栈实战项目。用户只需输入自然语言描述，平台即可调用 AI 自动生成完整的前端应用（HTML 单文件、多文件项目、Vue 项目等），并支持可视化编辑、一键云端部署和项目源码下载。

### 核心能力一览

| 能力 | 描述 |
|------|------|
| 🤖 智能代码生成 | 用户输入需求，AI 自动选择生成策略（HTML / 多文件 / Vue 项目），流式输出实时可见 |
| ✏️ 可视化编辑 | 生成的应用实时展示，支持选中元素与 AI 对话快速修改 |
| 🚀 一键部署分享 | 应用部署到云端，自动截图封面，生成可访问链接，支持源码下载 |
| 🏢 企业级管理 | 用户管理、应用管理、对话管理、系统监控、AI 调用指标监控 |

---

## 二、技术栈

### 后端（单体版）

| 技术 | 版本 | 用途 |
|------|------|------|
| Spring Boot | 3.5.4 | 核心框架 |
| LangChain4j | 1.1.0 | AI 应用开发框架（对话记忆、工具调用、流式输出） |
| LangGraph4j | 1.6.0-rc2 | AI 工作流编排框架（节点图、状态管理） |
| MyBatis-Flex | 1.11.1 | ORM 框架，支持 Mapper 代码生成 |
| Spring Session + Redis | — | 分布式 Session 管理 |
| Redisson | 3.50.0 | Redis 客户端，用于分布式限流 |
| Caffeine | — | 本地缓存（多级缓存） |
| Selenium + WebDriverManager | 4.33.0 / 6.1.0 | 网页截图服务 |
| 腾讯云 COS SDK | 5.6.227 | 对象存储（代码文件、封面图） |
| 阿里云 DashScope SDK | 2.21.1 | 文生图 AI 模型（wan2.2-t2i-flash） |
| Knife4j / OpenAPI 3 | 4.4.0 | API 文档 |
| Micrometer + Prometheus | — | 系统与业务指标监控 |
| HikariCP | 4.0.3 | 数据库连接池 |
| Hutool | 5.8.38 | 工具库 |
| Lombok | 1.18.36 | 代码简化 |
| MySQL | — | 主数据库 |

### 后端（微服务版）

在单体基础上额外引入：

| 技术 | 版本 | 用途 |
|------|------|------|
| Spring Cloud Alibaba | 2023.0.1.0 | 微服务生态（服务注册/发现/配置） |
| Nacos | — | 服务注册中心 & 配置中心 |
| Apache Dubbo | 3.3.0 | 高性能 RPC 跨服务调用 |
| Spring Cloud | 2023.0.1 | Spring 微服务基础设施 |

### 前端

| 技术 | 用途 |
|------|------|
| Vue 3 + TypeScript | 核心框架 |
| Vite | 构建工具 |
| Vue Router | 单页路由 |
| Ant Design Vue | UI 组件库 |
| Pinia | 全局状态管理 |
| ESLint + Prettier | 代码规范 |
| openapi2ts | 根据 OpenAPI 规范自动生成 TypeScript API 客户端 |

### AI 模型

| 用途 | 模型 |
|------|------|
| 代码生成（主力） | DeepSeek Chat（deepseek-chat） |
| 推理任务（复杂路由/质检） | DeepSeek Reasoner（deepseek-reasoner） |
| 智能路由分类 | 阿里云 Qwen Turbo（qwen-turbo，轻量快速） |
| 文生图 | 阿里云 wan2.2-t2i-flash |

---

## 三、项目结构总览

```
yu-ai-code-mother/
├── src/                            # 单体项目（Spring Boot）
│   └── main/
│       ├── java/com/yupi/yuaicodemother/
│       │   ├── YuAiCodeMotherApplication.java
│       │   ├── controller/         # HTTP 控制器
│       │   ├── service/            # 业务逻辑层
│       │   ├── config/             # 配置类
│       │   ├── langgraph4j/        # AI 工作流（节点、状态、图）
│       │   ├── ratelimter/         # 分布式限流（AOP + Redisson）
│       │   ├── manager/            # 基础服务封装（COS 等）
│       │   ├── utils/              # 工具类
│       │   ├── common/             # 公共类（BaseResponse、PageRequest 等）
│       │   ├── exception/          # 异常体系
│       │   └── constant/           # 常量定义
│       └── resources/
│           ├── application.yml     # 主配置
│           ├── mapper/             # MyBatis XML Mapper
│           ├── prompt/             # AI 系统提示词文件（txt）
│           └── static/             # 静态测试页面
│
├── yu-ai-code-mother-microservice/ # 微服务版本（Spring Cloud）
│   ├── yu-ai-code-common/          # 公共模块（异常、工具、COS 等）
│   ├── yu-ai-code-model/           # 数据模型（Entity、DTO、VO）
│   ├── yu-ai-code-client/          # Dubbo 内部服务接口定义
│   ├── yu-ai-code-user/            # 用户微服务
│   ├── yu-ai-code-app/             # 应用核心微服务
│   ├── yu-ai-code-ai/              # AI 能力微服务
│   └── yu-ai-code-screenshot/      # 截图微服务
│
├── yu-ai-code-mother-frontend/     # Vue 3 前端
│   └── src/
│       ├── pages/                  # 页面组件
│       ├── components/             # 公共组件
│       ├── api/                    # 自动生成的 API 客户端
│       ├── router/                 # 路由配置
│       ├── utils/                  # 工具函数
│       └── layouts/                # 布局组件
│
├── sql/                            # 数据库初始化 SQL
├── grafana/                        # Grafana 监控面板配置
├── prometheus.yml                  # Prometheus 采集配置
└── pom.xml                         # 单体项目 Maven 配置
```

---

## 四、数据库设计

数据库名：`yu_ai_code_mother`，包含 3 张核心表：

### 4.1 user（用户表）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键（自增） |
| userAccount | varchar(256) | 登录账号（唯一） |
| userPassword | varchar(512) | 加密密码 |
| userName | varchar(256) | 昵称 |
| userAvatar | varchar(1024) | 头像 URL |
| userProfile | varchar(512) | 个人简介 |
| userRole | varchar(256) | 角色：user / admin |
| editTime / createTime / updateTime | datetime | 时间戳 |
| isDelete | tinyint | 逻辑删除标志 |

### 4.2 app（应用表）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| appName | varchar(256) | 应用名称 |
| cover | varchar(512) | 封面图 URL（截图后上传到 COS） |
| initPrompt | text | 初始化 Prompt（AI 生成依据） |
| codeGenType | varchar(64) | 生成类型枚举（HTML / MULTI_FILE / VUE_PROJECT） |
| deployKey | varchar(64) | 部署标识（唯一，用于静态资源路由） |
| deployedTime | datetime | 部署时间 |
| priority | int | 精选排序优先级（管理员设置） |
| userId | bigint | 创建用户 ID（外键） |

### 4.3 chat_history（对话历史表）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | bigint | 主键 |
| message | text | 消息内容 |
| messageType | varchar(32) | 类型：user / ai |
| appId | bigint | 关联应用 ID |
| userId | bigint | 用户 ID |

**索引策略**：在 `(appId, createTime)` 建立联合索引，支持游标分页查询历史消息。

---

## 五、单体项目核心模块

### 5.1 控制器层（controller）

| 控制器 | 职责 |
|--------|------|
| `UserController` | 用户注册、登录、登出、管理员CRUD |
| `AppController` | 应用创建/查询/删除/部署/下载源码 |
| `ChatHistoryController` | 对话历史查询（游标分页） |
| `WorkflowSseController` | 基于 SSE/Flux 的 AI 流式对话接口 |
| `StaticResourceController` | 服务生成的静态应用文件 |
| `HealthController` | 健康检查接口 |

### 5.2 AI 工作流（langgraph4j）

这是本项目的技术亮点之一。使用 LangGraph4j 将代码生成流程拆分为多个节点（Node），通过有向图描述流程：

```
START
  └─→ image_collector     # 图片收集节点（图片搜索 + 文生图）
        └─→ prompt_enhancer   # Prompt 增强节点（优化用户描述）
              └─→ router        # 智能路由节点（决定生成策略）
                    └─→ code_generator  # 代码生成节点（调用 AI 生成代码）
                          └─→ project_builder  # 项目构建节点（代码写入文件系统）
                                └─→ END
```

**并发子图**（`CodeGenConcurrentWorkflow`）：`ImageCollectorNode` 内部包含并发子图，同时并行收集 Logo、插图、内容图等多类图片：

```
START
  └─→ image_plan_node          # 制定图片搜集计划
        ├─→ logo_collector       # 并行
        ├─→ illustration_collector # 并行
        └─→ content_image_collector # 并行
              └─→ (image_aggregator_node) # 汇聚
                    └─→ END
```

### 5.3 分布式限流（ratelimiter）

- **注解驱动**：`@RateLimit` 注解标注在方法上，指定限流类型（USER / IP / GLOBAL）和窗口时间
- **AOP 切面**：`RateLimitAspect` 拦截带注解的方法，从 `RateLimitType` 枚举中获取限流 key
- **执行器**：使用 Redisson 的滑动窗口限流器（`RRateLimiter`）实现精确限流

### 5.4 多级缓存

| 缓存层 | 技术 | 适用场景 |
|--------|------|----------|
| 本地 L1 | Caffeine | 精选应用列表（高频只读） |
| 分布式 L2 | Redis | AI 对话记忆（ChatMemoryStore）、Session |

`RedisCacheManagerConfig` 配置了 Spring 的 `CacheManager`，使用 Redis 作为缓存后端，支持 TTL 配置。

### 5.5 AI 模型路由

`RoutingAiModelConfig` 和 `StreamingChatModelConfig` 实现了**多 AI 模型按场景路由**：

| 模型别名 | 场景 |
|---------|------|
| `routingChatModel` | 简单分类任务（轻量 Qwen Turbo） |
| `streamingChatModel` | 常规代码生成（DeepSeek Chat） |
| `reasoningStreamingChatModel` | 复杂推理（DeepSeek Reasoner） |

---

## 六、微服务架构

### 6.1 服务划分

| 微服务模块 | 端口 | 职责 |
|-----------|------|------|
| `yu-ai-code-user` | — | 用户注册/登录/鉴权 |
| `yu-ai-code-app` | — | 应用管理、对话历史、代码解析、文件保存 |
| `yu-ai-code-ai` | — | AI 代码生成能力（LangChain4j 封装） |
| `yu-ai-code-screenshot` | — | Selenium 网页截图、封面图上传 |
| `yu-ai-code-common` | — | 公共类库（异常、工具、配置） |
| `yu-ai-code-model` | — | 共享数据模型（Entity、DTO、VO） |
| `yu-ai-code-client` | — | Dubbo 内部接口定义（`InnerUserService` 等） |

### 6.2 服务间通信

使用 **Apache Dubbo 3.3.0** 通过 Nacos 注册中心实现 RPC 跨服务调用：

- `yu-ai-code-user` 实现 `InnerUserService`（Dubbo Provider）
- `yu-ai-code-screenshot` 实现 `InnerScreenshotService`（Dubbo Provider）
- `yu-ai-code-app` 通过 Dubbo Consumer 调用以上两个内部服务

### 6.3 依赖关系图

```
yu-ai-code-app
  ├── →(Dubbo RPC)→ yu-ai-code-user
  ├── →(Dubbo RPC)→ yu-ai-code-screenshot
  ├── →(依赖)→ yu-ai-code-ai
  ├── →(依赖)→ yu-ai-code-common
  ├── →(依赖)→ yu-ai-code-model
  └── →(依赖)→ yu-ai-code-client
```

---

## 七、前端架构

### 7.1 页面结构

| 路由 | 页面 | 功能 |
|------|------|------|
| `/` | `HomePage` | 首页，精选应用展示 |
| `/user/login` | `UserLoginPage` | 用户登录 |
| `/user/register` | `UserRegisterPage` | 用户注册 |
| `/admin/userManage` | `UserManagePage` | 管理员用户管理 |
| `/admin/appManage` | `AppManagePage` | 管理员应用管理 |
| `/admin/chatManage` | `ChatManagePage` | 管理员对话管理 |
| `/app/chat/:id` | `AppChatPage` | 应用 AI 对话（核心页面） |
| `/app/edit/:id` | `AppEditPage` | 可视化编辑应用 |

### 7.2 关键组件

| 组件 | 功能 |
|------|------|
| `AppCard` | 应用卡片展示（封面 + 名称 + 操作） |
| `AppDetailModal` | 应用详情弹窗 |
| `DeploySuccessModal` | 部署成功弹窗（展示访问链接） |
| `GlobalHeader` | 顶部导航（含登录状态） |
| `MarkdownRenderer` | AI 回复 Markdown 渲染 |
| `UserInfo` | 用户信息展示 |

### 7.3 API 自动生成

通过 `openapi2ts` 工具，根据后端 OpenAPI 3 规范自动生成 `src/api/` 下的 TypeScript 类型定义和请求函数，保证前后端类型一致性。

---

## 八、AI 提示词（Prompt）工程

`src/main/resources/prompt/` 目录下维护了 6 个系统提示词文件：

| 文件 | 用途 |
|------|------|
| `codegen-html-system-prompt.txt` | 生成单文件 HTML 应用 |
| `codegen-multi-file-system-prompt.txt` | 生成多文件代码项目 |
| `codegen-vue-project-system-prompt.txt` | 生成完整 Vue 3 项目（含工具调用） |
| `codegen-routing-system-prompt.txt` | 路由分类（判断生成哪种类型） |
| `code-quality-check-system-prompt.txt` | 代码质量检查节点 |
| `image-collection-system-prompt.txt` | 图片搜集指导 |
| `image-collection-plan-system-prompt.txt` | 图片搜集计划制定 |

**设计亮点**：提示词与代码解耦，存储在独立 `.txt` 文件中，便于迭代优化，无需修改 Java 代码。

---

## 九、关键设计模式

| 设计模式 | 应用位置 | 说明 |
|----------|----------|------|
| **工厂模式** | `AiCodeGeneratorServiceFactory`、`AiCodeGenTypeRoutingServiceFactory` | 根据代码生成类型动态选择 AI 服务实例 |
| **模板方法模式** | `CodeFileSaverTemplate` → `HtmlCodeFileSaverTemplate` / `MultiFileCodeFileSaverTemplate` | 代码文件保存流程骨架，子类实现差异步骤 |
| **策略模式** | `CodeParser` 接口 + `HtmlCodeParser` / `MultiFileCodeParser` | 不同类型代码的解析策略可互换 |
| **门面模式** | `AiCodeGeneratorFacade` | 统一封装 AI 代码生成入口，对调用方屏蔽复杂度 |
| **责任链/执行器** | `CodeParserExecutor`、`CodeFileSaverExecutor`、`StreamHandlerExecutor` | 将解析、保存、流处理步骤链式执行 |
| **AOP 切面** | `RateLimitAspect`、`AuthInterceptor` | 横切关注点分离（限流、鉴权） |
| **观察者/响应式** | `Flux<String>` 流式输出 | 使用 Project Reactor 实现非阻塞流式 AI 响应 |

---

## 十、监控体系

项目集成了完整的 **Prometheus + Grafana** 监控体系：

- **Spring Boot Actuator**：暴露 `/api/actuator/health`、`/api/actuator/prometheus` 等端点
- **Micrometer**：采集 JVM、HTTP、数据库连接池等指标
- **prometheus.yml**：配置了 Prometheus 抓取规则
- **grafana/**：内置 Grafana Dashboard JSON 配置，可导入即用

---

## 十一、安全设计

| 安全机制 | 实现方式 |
|----------|----------|
| 用户鉴权 | `@AuthCheck` 注解 + AOP 拦截，验证 Session 中的登录态和角色 |
| 分布式限流 | `@RateLimit` 注解 + Redisson 滑动窗口，防止 AI 接口被刷 |
| AI 输入防护 | `PromptSafetyInputGuardrail`：LangChain4j Guardrail 机制，过滤不安全的用户输入 |
| AI 输出防护 | `RetryOutputGuardrail`：对 AI 生成结果进行后置校验，不合格时自动重试 |
| CORS 配置 | `CorsConfig` 统一配置跨域策略 |
| 逻辑删除 | 所有核心表均使用 `isDelete` 字段软删除，防止数据意外丢失 |

---

## 十二、项目亮点总结

1. **AI 工作流架构**：使用 LangGraph4j 将代码生成流程拆解为可观测、可扩展的有向图节点，支持并发子图（图片并行搜集），是业界前沿的 AI 智能体开发模式。

2. **多模型路由**：通过配置不同场景下的 AI 模型（轻量路由 / 代码生成 / 深度推理），在性能与成本之间取得平衡。

3. **双架构并存**：同时提供单体版（教学友好）和微服务版（工程完整），具有很高的学习价值，适合从入门到进阶的完整学习路径。

4. **流式响应**：使用 `Flux<String>` + SSE 实现 AI 流式输出，提升用户体验，同时减少首字节延迟。

5. **代码生成工具调用**：Vue 项目生成时，AI 通过 LangChain4j 工具调用（`FileWriteTool`、`FileModifyTool`、`FileReadTool`、`FileDeleteTool`）直接操作文件系统，模拟真实的 AI Agent 编程能力。

6. **全链路可观测**：从 Actuator 端点到 Prometheus 采集再到 Grafana 可视化，完整覆盖系统健康、业务指标、AI 调用成本监控。

7. **工程化规范**：前端使用 openapi2ts 自动同步接口类型，后端使用 MyBatis-Flex 代码生成器，显著降低重复劳动，体现大厂工程化实践。
