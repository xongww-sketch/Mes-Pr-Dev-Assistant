# MES Agent 部署指南

> 版本：1.0 | 2026-04-14
> 目标平台：Hermes Agent / OpenClaw Agent

---

## 快速部署（5分钟）

### 1. 复制Skill目录

```bash
# 将整个skills目录复制到目标Agent的工作区
cp -r skills/ /path/to/agent/workspace/
```

### 2. 加载顺序

Agent启动时按以下顺序加载：

```
1. skills/_core/hermes-memory.md    ← 恢复记忆
2. skills/_core/mes-domain-knowledge.md  ← 领域知识
3. skills/_core/n8n-process-map.md      ← 流程地图
4. skills/_core/team-context.md         ← 团队上下文
5. skills/_core/decision-framework.md   ← 决策框架
6. skills/_core/mes-glossary.md         ← 术语表
7. skills/_router/problem-router.md     ← 问题路由器
```

### 3. 按需加载执行层Skill

根据问题类型加载对应Skill：

| 问题类型 | 加载文件 |
|----------|----------|
| Bug处理 | `skills/bug-triage/SKILL.md` |
| n8n运维 | `skills/n8n-ops/SKILL.md` |
| SN管理 | `skills/sn-management/SKILL.md` |
| 需求分析 | `skills/requirement-analysis/SKILL.md` |
| 样机管理 | `skills/sample-machine/SKILL.md` |

---

## 跨Agent移植

### 兼容性

- ✅ OpenClaw Agent
- ✅ Hermes Agent
- ✅ 任何支持Markdown文件读取的Agent

### 适配要点

1. **记忆格式**：`hermes-memory.md` 使用标准Markdown，兼容所有Agent
2. **文件路径**：所有路径相对于workspace根目录
3. **工具调用**：执行层Skill中的工具调用需根据目标Agent能力调整

### 最小化部署

如果只需要核心能力，复制以下文件即可：

```
skills/_core/mes-domain-knowledge.md
skills/_core/n8n-process-map.md
skills/_core/decision-framework.md
skills/_router/problem-router.md
```

---

## 更新机制

### 自改进循环

每次处理问题后，Agent应：

1. 检查是否需要更新参考文件
2. 如果是新Bug模式 → 更新 `common-bug-patterns.md`
3. 如果是新风险 → 更新 `risk-register.md`
4. 如果是新术语 → 更新 `mes-glossary.md`
5. 如果是新教训 → 更新 `decision-framework.md`

### 版本管理

- 每次重大更新在文件头部更新版本号
- 重要变更在memory/日期.md中记录

---

## 验证清单

部署后验证：

- [ ] 能正确识别Bug类型
- [ ] 能查询n8n流程信息
- [ ] 能理解MES术语
- [ ] 能按决策框架做判断
- [ ] 能正确路由问题到对应Skill

---

*部署完成后删除本文件中的检查清单。*
