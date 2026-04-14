# MES Agent Framework

> 高驰(COROS) MES制造执行系统 — Agent智能助理框架
> 
> 版本：1.0 | 创建日期：2026-04-14
> 作者：Jackson (熊旺)

---

## 📦 这是什么？

这是一套为 **MES产品经理兼研发** 角色设计的Agent Skill框架。

**核心目标**：无论使用哪个AI模型或Agent平台（OpenClaw、Hermes、或其他），装载这套框架即可立即投入MES系统的工作——处理Bug、对接需求、运维n8n、管理SN等。

**设计理念**：知识即代码，SOP即程序。所有领域知识、处理流程、决策原则都以Markdown文件形式存在，可版本控制、可移植、可扩展。

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

| 文件 | 内容 |
|------|------|
| `mes-domain-knowledge.md` | MES定位、系统边界、核心流程、ISA-95、7大模块、长期规划 |
| `n8n-process-map.md` | 322个workflow分类、关键路径、风险登记、故障排查速查 |
| `team-context.md` | 团队人员、职责分工、系统负责人、升级路径、沟通规范 |
| `mes-glossary.md` | MES领域术语速查表（按字母排序） |
| `decision-framework.md` | 3条决策原则、5类常见问题、决策树、经验教训、模板 |
| `hermes-memory.md` | Hermes持久化记忆初始化文件 |

### 编排层（_router/）— 统一入口

| 文件 | 内容 |
|------|------|
| `problem-router.md` | 关键词匹配路由、边缘情况关键词、上下文判断、紧急程度自动判断 |

### 执行层（SOP）— 具体处理流程

| 文件 | 内容 |
|------|------|
| `bug-triage/SKILL.md` | Bug接收→诊断→分类处理→验证→复盘 |
| `bug-triage/references/common-bug-patterns.md` | 常见Bug模式库 + 边缘情况关键词 + 诊断SQL速查 |
| `bug-triage/references/escalation-rules.md` | 升级矩阵、自动升级条件、联系人 |
| `n8n-ops/SKILL.md` | 日常巡检→故障排查→Workflow管理→性能优化→安全维护 |
| `n8n-ops/references/workflow-catalog.md` | Workflow分类目录（按业务域/紧急程度） |
| `n8n-ops/references/workflow-analysis.md` | n8n工作流分析参考（统计/分类/风险/SQL） |
| `n8n-ops/references/troubleshooting.md` | 故障排查标准流程 |
| `n8n-ops/references/risk-register.md` | 风险登记册（10项风险，持续跟踪） |
| `sn-management/SKILL.md` | SN生成→绑定→解绑→查询→数据修复→外部上传 |
| `requirement-analysis/SKILL.md` | 需求接收→评估→决策→PRD撰写→评审→跟踪 |
| `requirement-analysis/references/prd-template.md` | PRD标准模板 |
| `sample-machine/SKILL.md` | 样机领用→试用→到期提醒→退还→退款 |

### 部署与参考

| 文件 | 内容 |
|------|------|
| `DEPLOYMENT.md` | 部署指南（跨Agent移植/更新机制/验证清单） |
| `QUICK-REFERENCE.md` | 快速参考卡（紧急处理/升级路径/常用SQL/决策树） |
| `INDEX.md` | 主索引与使用指南 |

**总计：22个文件，~100KB**

---

## 🚀 快速开始

### 新Agent加载步骤

1. **读取索引** → `INDEX.md`
2. **读取基础层** → `_core/` 下6个文件（建立知识根基）
3. **读取路由层** → `_router/problem-router.md`（学会如何分发问题）
4. **按需读取执行层** → 根据问题类型读取对应SOP

### 最小化部署

如果只需要核心能力，复制以下文件即可：

```
_core/mes-domain-knowledge.md
_core/n8n-process-map.md
_core/decision-framework.md
_core/mes-glossary.md
_router/problem-router.md
```

---

## 🔄 持续改进

### 自改进机制

每次处理问题后，Agent应：

1. 检查是否需要更新参考文件
2. 新Bug模式 → 更新 `common-bug-patterns.md`
3. 新风险 → 更新 `risk-register.md`
4. 新术语 → 更新 `mes-glossary.md`
5. 新教训 → 更新 `decision-framework.md`
6. 团队变动 → 更新 `team-context.md`
7. n8n workflow变动 → 更新 `n8n-process-map.md`

### 版本管理

- 每次重大更新在文件头部更新版本号
- 重要变更在memory/日期.md中记录
- 定期同步到本仓库

---

## 📊 覆盖场景

| 场景 | 覆盖Skill | 状态 |
|------|----------|------|
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

## 📝 维护日志

| 日期 | 变更 | 维护人 |
|------|------|--------|
| 2026-04-14 | 初始版本，22个文件，~100KB | OpenClaw Agent |

---

## 📄 License

内部使用，高驰(COROS)信息化团队。
