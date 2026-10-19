# Oiishi — 多 Agent 饮食推荐助手

Oiishi 是一个基于 Spring Boot、AgentScope 和 MySQL 的多 Agent 饮食推荐项目。它通过意图识别、槽位提取、候选检索与重排，将用户的餐次、口味、场景和健康目标转化为可解释的餐食建议，并支持多轮调整与多餐规划。

## 功能亮点

- 多 Agent 编排：意图识别、澄清、推荐回复、计划回复和评估 Judge 分工协作
- 多轮会话：保存会话阶段、历史槽位和上次推荐结果
- 双数据源：支持个人餐食库与公共餐食库
- 推荐流水线：数据库召回、规则重排、LLM 生成推荐理由
- 安全守卫：拦截医疗承诺、极端节食和特殊人群风险表达
- 可观测与评估：记录 Agent Trace、延迟、Token 成本、回退、安全性及人工标注
- Web 界面：提供聊天推荐、餐食管理、Trace 调试和评估页面

## 技术栈

- Java 21
- Spring Boot 3.3
- AgentScope Java
- MyBatis
- MySQL 8
- 原生 HTML、CSS、JavaScript

## 核心流程

```text
用户输入
  → 会话加载与 Trace 创建
  → 意图识别与规则校正
  → 槽位合并 / 必要时澄清
  → 餐食召回与重排
  → 推荐或多餐计划生成
  → 健康风险守卫
  → 状态、消息与 Trace 落库
```

## 本地运行

### 1. 环境要求

- JDK 21
- Maven 3.9+
- MySQL 8.x
- 可用的 DashScope API Key

### 2. 初始化数据库

创建或连接本地 MySQL 后，执行：

```bash
mysql -u <用户名> -p < src/main/resources/db/diet_db.sql
```

默认连接 `localhost:3306/diet_db`，也可以通过 `DB_URL` 覆盖。

### 3. 配置环境变量

```bash
export DB_USERNAME='<数据库用户名>'
export DB_PASSWORD='<数据库密码>'
export DASHSCOPE_API_KEY='<DashScope API Key>'
```

可选配置：

| 环境变量 | 默认值 | 说明 |
| --- | --- | --- |
| `DB_URL` | 本地 `diet_db` | JDBC 连接地址 |
| `SERVER_PORT` | `8080` | 服务端口 |
| `DIET_MAIN_MODEL` | `qwen-max` | 推荐与规划模型 |
| `DIET_LIGHT_MODEL` | `qwen-turbo` | 意图与澄清模型 |
| `DIET_MAX_HISTORY_TURNS` | `10` | 最大历史轮数 |
| `DIET_LOG_LEVEL` | `INFO` | 项目日志级别 |

### 4. 启动项目

```bash
mvn spring-boot:run
```

打开 <http://localhost:8080> 即可使用 Web 界面。

## 主要接口

| 方法 | 路径 | 用途 |
| --- | --- | --- |
| `POST` | `/api/v1/diet/chat` | 多轮饮食推荐对话 |
| `POST` | `/api/v1/diet/sessions` | 创建会话 |
| `GET/POST/PUT/DELETE` | `/api/v1/diet/meals/personal` | 管理个人餐食 |
| `GET` | `/api/v1/diet/meals/public` | 查询公共餐食 |
| `POST` | `/api/v1/diet/feedback` | 提交推荐反馈 |
| `POST` | `/api/v1/diet/evaluations` | 执行 Trace 评估 |

## 项目结构

```text
src/main/java/com/diet/
├── agent/          # Agent 构建、工厂与 Prompt 加载
├── controller/     # HTTP API
├── service/        # 编排、推荐、会话、评估与安全逻辑
├── mapper/         # MyBatis Mapper 接口
└── model/          # 请求、响应及领域模型

src/main/resources/
├── db/             # 数据库结构与演示种子数据
├── diet/prompts/   # Agent Prompt
├── mapper/         # MyBatis XML
└── static/         # Web 前端
```

## 安全说明

项目不在代码中保存数据库密码或模型服务密钥。运行所需凭据必须通过环境变量注入；请勿提交 `.env`、本地配置文件、日志或私钥。

> 本项目提供一般性的餐食推荐，不构成医疗诊断、治疗或个体化营养处方。
