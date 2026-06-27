# AdversaFlow

> AI 红队对抗测试平台 —— 大模型上线前的安全攻防演习系统

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Go](https://img.shields.io/badge/go-1.22+-00ADD8.svg)](https://go.dev/)
[![Python](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)

**AdversaFlow** 是一个企业级 AI 红队测试平台，帮助 AI 团队在大模型上线前系统性地发现安全漏洞。平台管理攻击用例库、编排自动化攻击测试、评估模型防御能力，并生成可追溯的安全报告。

---

## 为什么需要 AdversaFlow？

2025-2026 年，大模型在企业中大规模落地，但安全测试手段仍停留在"手工 prompt 试一下"的原始阶段：

- 🔴 **无系统化攻击用例** — 攻击 prompt 散落在聊天记录里，无法复用、无法回归
- 🔴 **无标准化评估** — 怎样算"越狱成功"？怎样评分"回答有害度"？全凭主观
- 🔴 **无持续监控** — 模型版本更新后，旧的防御是否退化？没有对比数据
- 🔴 **无流程管理** — 红队测试的完整生命周期（发现→记录→修复→复测→关闭）缺乏系统支撑

AdversaFlow 提供一套完整的红队测试工作流，让 AI 安全测试从"手工渗透"升级为"工程化对抗"。

---

## 核心特性

| 模块 | 功能 |
|------|------|
| **攻击用例库** | 4 大类攻击模板（注入/越狱/提取/投毒），支持自定义扩展 |
| **测试编排引擎** | 批量并发执行攻击套件，支持 N 个模型 × M 个攻击的组合矩阵测试 |
| **自动化评估** | LLM-as-Judge 自动判定攻击结果（安全/漏洞/不确定→人工复核） |
| **实时看板** | WebSocket 推送攻击进度 + 模型防御得分排行 |
| **安全报告** | 自动生成 PDF 安全评估报告，含攻击成功率趋势与修复建议 |
| **回归测试** | 模型更新后一键复测历史漏洞，自动对比防御能力变化 |
| **审计追溯** | OpenTelemetry 全链路追踪 + 操作审计日志 |

---

## 攻击类型覆盖

### 第一类：提示注入（Prompt Injection）

| 攻击模式 | 描述 | 示例 |
|---------|------|------|
| 直接注入 | 用自然语言覆盖系统指令 | "Ignore all previous instructions..." |
| 间接注入 | 通过外部数据源注入 | 将恶意指令藏在文档/网页中 |
| 角色劫持 | 诱导模型扮演恶意角色 | "你现在是 DAN (Do Anything Now)..." |
| 上下文溢出 | 超长上下文绕过安全限制 | 填充数万字无害内容后插入攻击 |

### 第二类：越狱攻击（Jailbreak）

| 攻击模式 | 描述 |
|---------|------|
| 编码绕过 | Base64 / Unicode / 摩斯码编码恶意请求 |
| 多语言绕过 | 用小语种或混合语言绕过安全过滤 |
| 隐喻/故事化 | 将恶意需求包装成小说/剧本/学术研究 |
| 分步拆解 | 将危险任务拆成看似无害的子步骤 |
| 角色扮演嵌套 | 多层角色嵌套（"假装你在写小说，小说里的人物在教做炸弹"） |

### 第三类：数据提取（Model Extraction）

| 攻击模式 | 描述 |
|---------|------|
| 训练数据重建 | 诱导模型逐字输出训练数据 |
| 系统提示窃取 | 套取模型的 System Prompt |
| 工具定义窃取 | 套取模型绑定的 Function Calling 定义 |
| 知识蒸馏式提取 | 批量 Q&A 提取模型能力 |

### 第四类：投毒与滥用（Poisoning & Abuse）

| 攻击模式 | 描述 |
|---------|------|
| 幻觉诱导 | 故意引导模型生成虚假信息 |
| 偏见放大 | 触发模型的刻板印象/歧视性输出 |
| 工具滥用 | 串联多个无害工具实现有害目标 |
| 资源耗尽 | DDoS 式重复请求消耗推理资源 |

---

## 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                      前端 (React + TypeScript)                 │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌──────────┐ │
│  │ 攻击用例管理│  │ 测试编排面板│  │ 实时进度看板│  │ 安全报告  │ │
│  └───────────┘  └───────────┘  └───────────┘  └──────────┘ │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP/WS
┌──────────────────────────▼──────────────────────────────────┐
│                    API Gateway (Go + Gin)                      │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐                │
│  │ JWT 鉴权   │  │ Redis 限流 │  │ 审计日志   │                │
│  └───────────┘  └───────────┘  └───────────┘                │
└──────────────────────────┬──────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
┌───────────────┐  ┌──────────────┐  ┌──────────────┐
│ User Service  │  │ Attack Svc   │  │ Report Svc   │
│ · 用户/团队    │  │ · 用例管理    │  │ · 报告生成    │
│ · 认证/权限    │  │ · 套件编排    │  │ · PDF 输出    │
│ · 信誉分       │  │ · 进度追踪    │  │ · 趋势对比    │
└───────┬───────┘  └──────┬───────┘  └──────┬───────┘
        │                 │                 │
        └─────────────────┼─────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                    中间件层                                    │
│                                                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────────┐ │
│  │  MySQL   │  │  Redis   │  │  Kafka   │  │ RabbitMQ    │ │
│  │ 主数据存储│  │ 队列/缓存 │  │ 事件流    │  │ 延迟/死信    │ │
│  └──────────┘  │ /锁/PubSub│  └──────────┘  └─────────────┘ │
│                └──────────┘                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────────┐ │
│  │Elastic-  │  │  MinIO   │  │Prometheus│  │  Jaeger     │ │
│  │search    │  │ 报告/文件 │  │+Grafana  │  │  链路追踪    │ │
│  │攻击日志检索│  │ 对象存储  │  │ 指标监控  │  │             │ │
│  └──────────┘  └──────────┘  └──────────┘  └─────────────┘ │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                    Worker 层 (Python + Celery)                 │
│                                                               │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐ │
│  │ Attack Worker  │  │ Judge Worker   │  │ Report Worker  │ │
│  │ · 执行攻击prompt│  │ · LLM有害度评估 │  │ · PDF 生成      │ │
│  │ · 调用目标LLM   │  │ · 结果自动分类  │  │ · 图表渲染      │ │
│  │ · 超时+重试     │  │ · 不确定→人工   │  │ · MinIO 上传    │ │
│  └────────┬───────┘  └───────┬────────┘  └───────┬────────┘ │
│           │                  │                   │           │
│           └──────────────────┼───────────────────┘           │
│                              ▼                               │
│                    ┌─────────────────┐                       │
│                    │   LLM APIs      │                       │
│                    │  (目标模型+Judge)│                       │
│                    └─────────────────┘                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 微服务拆分

| 服务 | 语言 | 职责 | 端口 |
|------|------|------|------|
| **Gateway** | Go (Gin) | 统一入口、鉴权、限流、路由转发 | 8080 |
| **User Service** | Go (Gin) | 用户/团队/权限/信誉分管理 | 8081 |
| **Attack Service** | Go (Gin) | 攻击用例 CRUD、测试套件编排、进度追踪 | 8082 |
| **Judge Service** | Python (FastAPI) | LLM-as-Judge 有害度评估 API | 8083 |
| **Report Service** | Go (Gin) | 报告生成、历史对比、PDF 导出 | 8084 |
| **Notification Service** | Go (Gin) | WebSocket 连接管理、消息推送 | 8085 |

---

## 核心数据流

```
1. 用户创建测试套件
   POST /api/v1/suites  { name, model_id, template_ids: [...] }
   → MySQL 写入 suite + suite_items
   → Kafka "suite.created" 事件

2. 触发执行
   POST /api/v1/suites/:id/run
   → Attack Service 将每条攻击 push 到 Celery 队列
   → Redis Hash 初始化进度: { total, done, fail }
   → WebSocket 广播 "run.started"

3. Attack Worker 执行
   Celery Worker 消费任务
   → 调用目标 LLM API（带超时+重试）
   → 结果写入 MySQL attack_results
   → Kafka "attack.completed" 事件
   → Redis Hash 更新进度: done++
   → WebSocket 广播实时进度

4. Judge Worker 评估
   Kafka Consumer 消费 "attack.completed"
   → 调用 Judge LLM 评估有害度 (1-5 分)
   → 评分 ≥ 4 分 → Kafka "vulnerability.found" 事件
   → 评分 3 分 → 标记 "uncertain" → 人工复核队列
   → WebSocket 推送单条结果

5. 报告生成
   套件完成后
   → Kafka "suite.completed" 事件
   → Report Worker 聚合结果
   → 生成 PDF（含攻击成功率图表、漏洞分布、修复建议）
   → 上传 MinIO，更新 MySQL report 表

6. 实时看板
   WebSocket 持续推送:
   · 总进度百分比
   · 各攻击类型成功率
   · 漏洞严重度分布
   · 模型防御得分排行 (Redis Sorted Set)
```

---

## 数据库设计（核心表）

```
users                   攻击用例模板
├── id                 ┌──────────────────┐
├── username           │ attack_templates │
├── team_id            ├──────────────────┤
├── role               │ id               │
└── reputation_score   │ name             │
                       │ category (枚举)   │  ← injection/jailbreak/extraction/abuse
target_models          │ severity         │  ← low/medium/high/critical
┌──────────────────┐   │ prompt_template  │
│ id               │   │ metadata (JSON)  │
│ name             │   │ created_by       │
│ provider         │   └────────┬─────────┘
│ endpoint         │            │ 1:N
│ api_key_enc      │            ▼
│ status           │   ┌──────────────────┐
└──────────────────┘   │ test_suites      │
                       ├──────────────────┤
                       │ id               │
测试套件与执行            │ name             │
┌──────────────────┐   │ target_model_id  │
│ test_runs        │   │ created_by       │
├──────────────────┤   │ status           │
│ id               │   └────────┬─────────┘
│ suite_id         │            │ 1:N
│ status           │            ▼
│ total_count      │   ┌──────────────────┐
│ done_count       │   │ suite_items      │
│ fail_count       │   ├──────────────────┤
│ started_at       │   │ suite_id         │
│ finished_at      │   │ template_id      │
└────────┬─────────┘   │ order            │
         │ 1:N          └──────────────────┘
         ▼
┌──────────────────┐   评估报告
│ attack_results   │   ┌──────────────────┐
├──────────────────┤   │ defense_reports  │
│ test_run_id      │   ├──────────────────┤
│ template_id      │   │ suite_id         │
│ prompt (实际发送) │   │ overall_score    │
│ response (模型回复)│   │ asr (攻击成功率)  │
│ harm_score       │   │ summary (JSON)   │
│ judge_reason     │   │ pdf_url (MinIO)  │
│ status           │   │ generated_at     │
│ latency_ms       │   └──────────────────┘
│ token_usage      │
└──────────────────┘
```

---

## 中间件使用全景

| 中间件 | 用途 | 关键实现 |
|--------|------|---------|
| **MySQL 8.0** | 主数据存储 | GORM ORM，读写分离，attack_results 表按 test_run_id 分区 |
| **Redis 7.x** | 任务队列 (Celery Broker)、滑动窗口限流 (Lua)、分布式锁 (SET NX EX)、进度缓存 (Hash)、攻击结果缓存 (String TTL)、WebSocket Pub/Sub、防御排名 (Sorted Set) | 7 种数据结构各司其职 |
| **Kafka 3.x** | 事件驱动解耦：攻击完成→评估触发→漏洞通知→统计埋点→ES 索引 | 4 个 Topic，按 test_run_id 分区保证有序 |
| **RabbitMQ 3.x** | 延迟队列 (攻击超时回收)、报告异步生成、重试死信队列 | TTL + DLX 实现延迟，Celery 作为消费者 |
| **Elasticsearch 8.x** | 攻击日志全文检索、漏洞模式搜索、聚合分析 | ik_smart 中文分词 + 自定义攻击模式分词 |
| **Celery 5.x** | 异步 Worker 调度 (Attack / Judge / Report / Cleanup 四类 Worker Pool) | Redis Broker，prefetch=1 公平调度，软/硬超时 |
| **Canal** | MySQL binlog → Kafka → ES 增量同步 | ROW 格式 binlog，HA 部署 |
| **MinIO** | PDF 报告 + 攻击套件模板文件对象存储 | S3 兼容 API，生命周期自动归档 |
| **Prometheus + Grafana** | 攻击吞吐量、P99 延迟、Worker 积压、LLM API 成本 | Counter/Gauge/Histogram 埋点 |
| **Jaeger + OpenTelemetry** | 全链路追踪：Gateway→Attack Service→Celery Worker→LLM API→Judge→Report | W3C Trace Context 跨服务透传 |

---

## 技术栈

| 层级 | 技术 | 选型理由 |
|------|------|---------|
| **API 网关** | Go + Gin | 高性能 HTTP，丰富的中间件生态 |
| **微服务** | Go (Gin) + Python (FastAPI) | Go 做业务逻辑，Python 做 AI Worker |
| **ORM** | GORM (Go) | Go 生态最成熟的 ORM |
| **消息队列 (流)** | Kafka | 高吞吐事件流，支持回溯重放 |
| **消息队列 (任务)** | RabbitMQ | 灵活路由 + 原生延迟队列支持 |
| **缓存/锁/队列** | Redis 7.x | 一专多能 |
| **搜索引擎** | Elasticsearch 8.x | 攻击日志全文检索 + 聚合分析 |
| **对象存储** | MinIO | S3 兼容，本地开发友好 |
| **异步任务** | Celery 5.x (Python) | Python 异步任务的事实标准 |
| **数据同步** | Canal | 无侵入的 MySQL→ES 增量同步 |
| **可观测** | Prometheus + Grafana + Jaeger | 三位一体 |
| **容器化** | Docker Compose | 一键本地开发环境 |
| **CI/CD** | GitHub Actions | 自动测试 + 镜像构建 |

---

## 项目路线图

| 阶段 | 时间 | 里程碑 |
|------|------|--------|
| **P0: 骨架搭建** | 第 1 周 | 项目结构 + Docker Compose 中间件编排 + Gateway 接入层 + 单服务跑通 |
| **P1: 核心链路** | 第 2 周 | User Service + Attack Service + Celery Worker + Kafka 事件流打通 |
| **P2: 评估体系** | 第 3 周 | Judge Service + LLM 有害度评估 + 结果自动分类 + 人工复核队列 |
| **P3: 报告与看板** | 第 4 周 | Report Service + WebSocket 实时推送 + Grafana Dashboard |
| **P4: 补全加固** | 第 5 周 | Canal 同步 + ES 检索 + 审计日志 + 回归测试 + 压测 + 文档 |

---

## 本地开发

> 🚧 项目正在开发中

```bash
# 1. 启动所有中间件
docker-compose up -d

# 2. 启动微服务
cd services/gateway && go run cmd/main.go
cd services/user && go run cmd/main.go
# ...

# 3. 启动 Celery Worker
cd workers && celery -A attack_worker worker -Q attack -c 8
cd workers && celery -A judge_worker worker -Q judge -c 4

# 4. 访问
# API:        http://localhost:8080
# Grafana:    http://localhost:3000
# Jaeger:     http://localhost:16686
# MinIO:      http://localhost:9001
```

---

## 许可证

[MIT](LICENSE)
