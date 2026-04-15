---
name: mes-server
version: 1.0
description: >
  MES Server(NestJS)代码知识库。覆盖完整架构、API接口、n8n迁移状态、
  关键业务逻辑（上位机分析、包装解绑、UUID追踪、工序版本、自动审计）。
category: knowledge
tags: [mes, nestjs, prisma, api, architecture, coros]
depends_on:
  - _core/mes-domain-knowledge
  - _core/n8n-process-map
---

# MES Server 知识库

> 来源：coros-mes-prod 代码通读 | 2026-04-15

## 何时使用此Skill

- 需要了解 MES Server 的架构、模块、技术栈
- 需要查找某个 API 接口的路径、参数、数据来源
- 需要了解 n8n→NestJS 的迁移进度和计划
- 需要理解复杂业务逻辑（上位机测试分析、包装解绑、UUID绑定等）
- 需要排查后端代码问题

## 参考文档

| 文档 | 路径 | 用途 |
|------|------|------|
| 架构文档 | references/mes-server-architecture.md | 技术栈/项目结构/数据模型/认证/审计/12个模块详解 |
| API参考 | references/mes-api-reference.md | 全量接口列表+n8n工作流映射 |
| 迁移地图 | references/mes-n8n-migration-map.md | 迁移状态/业务逻辑详解/优先级路线图 |

## 快速查询

### 技术栈
NestJS + Prisma + PostgreSQL + Redis + BullMQ + Vue3 + 飞书SDK + MQTT + n8n + FTP + SSH

### 核心实体关系
```
ProductModel → ProductInfo → Joborder → Retroid → MachineData
                                                     ↓
                                              ProcedureDetail
```

### 关键ID
- procedureDetailId=91: 解绑记录
- procedureDetailId=79: 称重
- procedureDetailId=81: 制箱
- DEFAULT_PROCEDURE_IDS(上位机): [96,95,94,93,97,125,134,135,136,148,149,155,156,140,141]

### 测试状态判定
- 第一次Pass → 直通(DIRECT_PASS)
- 第一次Fail + 最后Pass → 误测(MIS_TEST)
- 第一次Fail + 最后Fail → 真失败(TRUE_FAIL)
