---
name: bug-triage
version: 1.0
description: >
  Bug处理SOP。从接收Bug报告到诊断、处理、验证、复盘的全流程。
  覆盖MES系统各类Bug的处理模式，是日常最高频使用的Skill。
category: execution
tags: [bug, triage, troubleshooting, sop, mes]
depends_on:
  - _core/mes-domain-knowledge
  - _core/n8n-process-map
  - _core/team-context
  - _core/decision-framework
  - _router/problem-router
---

# Bug 处理 SOP

> 版本：1.0 | 最近更新：2026-04-14
> 用途：标准化Bug处理流程，从接收到复盘

---

## 1. 接收Bug报告

### 1.1 信息收集
收到Bug报告时，确认以下信息：

```
【必填】
- 问题描述：发生了什么？
- 影响范围：哪些工单/SN/产线受影响？
- 发现时间：什么时候开始的？

【选填】
- 复现步骤：怎么触发的？
- 截图/日志：有报错信息吗？
- 之前是否正常：是突然出现的还是逐渐出现的？
```

如果信息不全，先回复：
> "收到，请补充：影响范围（哪些工单/SN）、发现时间、是否有报错截图。"

### 1.2 紧急程度判断
按 `_core/decision-framework.md` 中的决策树判断 P0-P3。

---

## 2. 快速诊断（5分钟内）

### 2.1 第一步：判断问题域
根据Bug描述，判断属于哪个业务域：

| 问题域 | 检查点 | 相关Workflow |
|--------|--------|-------------|
| SN相关 | SN生成/绑定/解绑/查询 | SNgenerate_API, uuid_blind_api, unblind_api |
| 工单相关 | 工单创建/查询/关闭 | get_jobmessage, jobnumber_create_API |
| 过站相关 | 扫码过站/前置校验 | upload_machine_data, 前置工序校验通用接口 |
| 物料相关 | 物料绑定/查询 | get_matialCode_by_sn, binding_materials |
| 质检相关 | FQC/OQC/不良品 | insert_FQC_msg, repair_message |
| 包装相关 | 包装/出库/箱号 | 仓库扫码出库, upload_box_info |
| 打印相关 | 打印失败/模板 | updateprintinfo_API, createprintinfo_API |
| 设备相关 | 资源离线/登录 | resourceOnlineStatus, ResouceLogin |
| 同步相关 | 渠道/数据同步 | [Sync]系列, 手动同步 |
| 外部系统 | COROS后台/SAP/OMS | 对应外部对接workflow |

### 2.2 第二步：检查n8n Workflow状态
```
1. 查对应workflow是否活跃（active=true）
2. 查最近执行记录是否有失败
3. 查worker状态（docker ps | grep n8n）
4. 查数据库连接（PostgreSQL是否正常）
```

### 2.3 第三步：检查数据库
```sql
-- 查SN状态
SELECT * FROM sn_info WHERE sn = 'xxx' LIMIT 10;

-- 查绑定记录
SELECT * FROM binding_record WHERE sn = 'xxx' LIMIT 10;

-- 查过站记录
SELECT * FROM station_record WHERE sn = 'xxx' ORDER BY timestamp DESC LIMIT 10;

-- 查工单状态
SELECT * FROM work_order WHERE work_order_id = 'xxx' LIMIT 1;
```

---

## 3. 分类处理

### 3.1 n8n Workflow异常

**症状**：workflow不执行、执行失败、队列堆积

**处理步骤**：
1. 检查workflow是否活跃 → 如非活跃，确认是否应该激活
2. 检查worker状态 → `docker ps | grep n8n`
3. 检查日志 → `docker logs n8n-worker --tail 100`
4. 如果是worker挂了 → `docker compose up -d`
5. 如果是单个workflow失败 → 检查节点配置、数据库连接
6. 如果是队列堆积 → 检查是否有慢workflow阻塞

**升级条件**：
- worker反复挂掉 → 通知王晓虎
- 数据库连接异常 → 通知王晓虎
- 无法定位原因 → 通知熊旺 + 王晓虎

### 3.2 数据库异常

**症状**：查询慢、写入失败、数据不一致

**处理步骤**：
1. 检查数据库连接 → `pg_isready`
2. 检查慢查询 → 查PostgreSQL慢查询日志
3. 检查磁盘空间 → `df -h`
4. 检查索引 → 是否有缺失索引
5. 如果是数据不一致 → 确认影响范围，制定修复方案

**升级条件**：
- 数据库不可用 → P0，立即通知王晓虎
- 慢查询影响产线 → P1，通知王晓虎优化
- 数据不一致影响核心业务 → P1，通知熊旺决策

### 3.3 外部系统异常

**症状**：API调用失败、数据同步中断、Token过期

**处理步骤**：
1. 检查对应外部系统连通性 → `curl` 测试API
2. 检查Token状态 → 查Redis/数据库中的Token
3. 检查对应workflow → 是否活跃、是否有失败记录
4. 手动触发同步 → 验证是否恢复
5. 如果是外部系统问题 → 通知对应负责人

**升级条件**：
- 外部系统不可用影响核心业务 → 通知熊旺协调
- Token刷新失败 → 检查对应Token workflow

### 3.4 数据修复

**症状**：数据缺失、数据错误、需要补数据

**处理步骤**：
1. 确认问题数据范围
2. 评估影响（是否影响主流程）
3. 制定修复方案（SQL/n8n手动触发/脚本）
4. 执行修复
5. 验证修复结果
6. 记录修复过程

**决策点**（按 `_core/decision-framework.md`）：
- 影响工单/出库？→ P0立即修复
- 仅影响报表？→ P2排期修复
- 需要绕开正常流程？→ 需要熊旺确认

---

## 4. 验证与闭环

### 4.1 验证
- [ ] 问题是否已解决？
- [ ] 是否有副作用？
- [ ] 相关功能是否正常？

### 4.2 通知
- 通知报Bug的人：问题已解决
- 如果影响范围大 → 群内同步处理结果

### 4.3 记录
```markdown
## Bug处理记录

**问题**：
**根因**：
**处理方案**：
**处理人**：
**处理时间**：
**经验教训**：
```

---

## 5. 常见Bug模式库

详见 `references/common-bug-patterns.md`

---

## 6. 升级规则

详见 `references/escalation-rules.md`

---

*本文档是Bug处理的标准SOP。*
*所有Bug都应先经过 `_router/problem-router.md` 路由到此SOP。*
