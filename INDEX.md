# MES Agent Skills — 索引与使用指南

> 版本：1.0 | 最近更新：2026-04-14
> 用途：所有Skill的统一入口，新Agent加载此文件即可理解整个知识体系

---

## 📦 这是什么？

这是一套为 **MES产品经理兼研发** 角色设计的Skill体系，覆盖：
- Bug处理、需求分析、n8n运维、SN管理、样机管理
- MES领域知识、n8n流程地图、团队上下文、决策框架
- 问题路由引擎（自动判断问题类型并分发到对应SOP）

**设计目标**：不管换哪个模型或平台，装载这些Skill就能立即工作。

---

## 🏗 架构总览

```
Layer 4: 执行层 (Action)          ← 具体SOP，处理实际问题
  ├── bug-triage/                 ← Bug处理
  ├── n8n-ops/                    ← n8n运维
  ├── sn-management/              ← SN管理
  ├── requirement-analysis/       ← 需求分析
  └── sample-machine/             ← 样机管理
       │
       ▼
Layer 3: 编排层 (Orchestration)   ← 问题路由，自动分发
  └── _router/problem-router.md   ← 统一入口
       │
       ▼
Layer 2: 知识层 (Knowledge)       ← 共享知识，所有Skill引用
  ├── _core/mes-domain-knowledge.md   ← MES领域知识
  ├── _core/n8n-process-map.md        ← n8n流程地图
  ├── _core/team-context.md           ← 团队上下文
  └── _core/mes-glossary.md           ← MES术语表
       │
       ▼
Layer 1: 记忆层 (Memory)          ← 决策原则、经验教训、持久化记忆
  ├── _core/decision-framework.md   ← 决策框架
  └── _core/hermes-memory.md        ← Hermes持久化记忆
```

---

## 📋 文件清单

### 基础层（_core/）— 所有Skill共享

| 文件 | 大小 | 内容 |
|------|------|------|
| `mes-domain-knowledge.md` | 9.4KB | MES定位、系统边界、核心流程、ISA-95、7大模块、长期规划 |
| `n8n-process-map.md` | 9.0KB | 322个workflow分类、关键路径、风险登记、故障排查速查 |
| `team-context.md` | 5.5KB | 团队人员、职责分工、系统负责人、升级路径、沟通规范 |
| `mes-glossary.md` | 2.0KB | MES领域术语速查表（按字母排序） |
| `decision-framework.md` | 8.0KB | 3条决策原则、5类常见问题、决策树、经验教训、模板 |
| `hermes-memory.md` | 1.4KB | Hermes持久化记忆初始化文件 |

### 编排层（_router/）— 统一入口

| 文件 | 大小 | 内容 |
|------|------|------|
| `problem-router.md` | 9.0KB | 关键词匹配路由、边缘情况关键词、上下文判断、紧急程度自动判断、路由示例 |

### 执行层（SOP）— 具体处理流程

| 文件 | 大小 | 内容 |
|------|------|------|
| `bug-triage/SKILL.md` | 5.7KB | Bug接收→诊断→分类处理→验证→复盘 |
| `bug-triage/references/common-bug-patterns.md` | 4.5KB | 常见Bug模式库 + 边缘情况关键词 + 诊断SQL速查 |
| `bug-triage/references/escalation-rules.md` | 1.3KB | 升级矩阵、自动升级条件、联系人 |
| `n8n-ops/SKILL.md` | 5.5KB | 日常巡检→故障排查→Workflow管理→性能优化→安全维护 |
| `n8n-ops/references/workflow-catalog.md` | 2.1KB | Workflow分类目录（按业务域/紧急程度） |
| `n8n-ops/references/workflow-analysis.md` | 2.1KB | n8n工作流分析参考（统计/分类/风险/SQL） |
| `n8n-ops/references/troubleshooting.md` | 1.9KB | 故障排查标准流程 |
| `n8n-ops/references/risk-register.md` | 1.7KB | 风险登记册（10项风险，持续跟踪） |
| `sn-management/SKILL.md` | 6.2KB | SN生成→绑定→解绑→查询→数据修复→外部上传 |
| `requirement-analysis/SKILL.md` | 6.0KB | 需求接收→评估→决策→PRD撰写→评审→跟踪 |
| `requirement-analysis/references/prd-template.md` | 1.6KB | PRD标准模板 |
| `sample-machine/SKILL.md` | 5.0KB | 样机领用→试用→到期提醒→退还→退款 |

### 部署与参考

| 文件 | 大小 | 内容 |
|------|------|------|
| `DEPLOYMENT.md` | 1.7KB | 部署指南（跨Agent移植/更新机制/验证清单） |
| `QUICK-REFERENCE.md` | 1.1KB | 快速参考卡（紧急处理/升级路径/常用SQL/决策树） |

**总计：21个文件，~85KB**

---

## 🚀 快速开始

### 新Agent加载步骤

1. **读取索引** → 本文件（`INDEX.md`）
2. **读取基础层** → `_core/` 下4个文件（建立知识根基）
3. **读取路由层** → `_router/problem-router.md`（学会如何分发问题）
4. **按需读取执行层** → 根据问题类型读取对应SOP

### 处理问题的标准流程

```
收到消息
  ↓
读取 _router/problem-router.md
  ↓
判断问题类型 → 匹配关键词/上下文
  ↓
判断紧急程度 → P0/P1/P2/P3
  ↓
路由到对应SOP → 读取对应Skill
  ↓
SOP中引用 _core/ 文件 → 获取领域知识/流程地图/团队信息/决策原则
  ↓
执行SOP → 按步骤处理
  ↓
输出结果 → 诊断结果 + 处理建议 + 是否需要升级
```

---

## 🔗 Skill依赖关系

```
bug-triage        → _core/* + _router + bug-triage/references/*
n8n-ops           → _core/* + _router + n8n-ops/references/*
sn-management     → _core/* + _router
requirement-analysis → _core/* + _router + requirement-analysis/references/*
sample-machine    → _core/* + _router
```

---

## 📊 覆盖场景

| 场景 | 覆盖Skill | 覆盖率 |
|------|----------|--------|
| 产线Bug（SN/绑定/过站/质检） | bug-triage + sn-management | ✅ |
| n8n故障（worker/队列/Token） | n8n-ops | ✅ |
| 需求对接（评估/PRD/评审） | requirement-analysis | ✅ |
| 样机管理（领用/试用/退款） | sample-machine | ✅ |
| 数据同步（OMS/SAP/渠道） | n8n-ops + bug-triage | ✅ |
| 外部系统（COROS/京东/抖音） | n8n-ops + bug-triage | ✅ |
| 数据库问题（慢查询/连接） | n8n-ops | ✅ |
| 决策判断（接不接需求） | decision-framework | ✅ |
| 找人/升级 | team-context | ✅ |

---

## 🔄 持续改进

### 如何更新Skill

1. 处理新问题后，补充到对应的 `references/` 文件
2. 发现新的Bug模式 → 更新 `common-bug-patterns.md`
3. 发现新的风险 → 更新 `risk-register.md`
4. 发现新的决策模式 → 更新 `decision-framework.md`
5. 团队变动 → 更新 `team-context.md`
6. n8n workflow变动 → 更新 `n8n-process-map.md`

### 自改进机制

如果部署在Hermes Agent上：
- 每次处理问题后，自动记录到记忆
- 定期Review Skill，补充新模式
- 用户反馈 → 自动优化SOP

---

## 📝 维护日志

| 日期 | 变更 | 维护人 |
|------|------|--------|
| 2026-04-14 | 初始版本，15个文件，78KB | OpenClaw Agent |
| 2026-04-14 | 增强：新增mes-glossary/hermes-memory/workflow-analysis/prd-template/DEPLOYMENT/QUICK-REFERENCE，共21个文件~85KB | OpenClaw Agent |

---

*本索引是Skill体系的入口文件。*
*新Agent加载此文件即可理解整个知识体系并开始工作。*
