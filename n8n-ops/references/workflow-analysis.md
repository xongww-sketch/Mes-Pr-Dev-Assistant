# n8n 工作流分析参考

> 版本：1.0 | 2026-04-14
> 来源：322个workflow JSON批量解析

---

## 统计概览

| 指标 | 数值 |
|------|------|
| 总Workflow数 | 322 |
| 总节点数 | ~8,500+ |
| 活跃Workflow | ~200 |
| 测试/废弃 | ~30 |
| 无文档/负责人 | ~50+ |

## 节点类型Top 10

| 节点类型 | 数量 | 说明 |
|----------|------|------|
| n8n-nodes-base.if | 1,310 | 条件判断 |
| n8n-nodes-base.postgres | 1,081 | PostgreSQL查询 |
| n8n-nodes-base.code | 1,002 | JavaScript代码 |
| n8n-nodes-base.webhook | 242 | Webhook触发 |
| n8n-nodes-base.httpRequest | ~200 | HTTP请求 |
| n8n-nodes-base.set | ~180 | 变量设置 |
| n8n-nodes-base.switch | ~150 | 多路分支 |
| n8n-nodes-base.merge | ~120 | 数据合并 |
| n8n-nodes-base.executeQuery | ~112 | SQL Server查询 |
| n8n-nodes-base.cron | ~80 | 定时触发 |

## 业务域分类

### 1. 核心生产（~60个）
- 过站/报工/绑定/解绑
- SN管理/物料追溯
- 关键流程：`get_jobmessage`, `get_matialCode_by_sn`

### 2. 包装（~30个）
- 包装过站/装箱/封箱
- 标签打印

### 3. 打印（~25个）
- 标签打印/模板渲染
- 打印机状态监控

### 4. 质检（~20个）
- IQC/IPQC/OQC
- 不良品处理

### 5. 维修（~15个）
- 维修过站/不良分析
- 维修记录

### 6. 设备（~20个）
- 设备状态监控
- `resourceOnlineStatus`（P0风险）

### 7. 数据同步（~40个）
- MES↔SAP
- MES↔SRM
- MES↔WMS

### 8. 外部对接（~25个）
- 苹果MFi/CPLC
- NFC/MFI加密

### 9. 报表（~20个）
- 生产报表
- 质量报表

### 10. 系统管理（~15个）
- 用户/权限
- 系统配置

### 11. 测试/废弃（~30个）
- Date&Time测试
- My Sub-Workflow 1-4
- 各种临时测试流程

## 风险清单

### P0 - 立即处理
| Workflow | 风险 | 建议 |
|----------|------|------|
| resourceOnlineStatus | 高频触发导致Worker崩溃 | 限流或改事件推送 |

### P1 - 本周处理
| Workflow | 风险 | 建议 |
|----------|------|------|
| get_jobmessage | 150节点，难以维护 | 拆分为子流程 |
| 仓库扫码出库 | 135节点，难以维护 | 拆分为子流程 |
| get_matialCode_by_sn | 105节点 | 优化查询逻辑 |

### P2 - 本月处理
| 风险 | 建议 |
|------|------|
| ~30个测试/废弃workflow | 清理删除 |
| ~50个无文档流程 | 补充文档 |
| SQL Server依赖（112节点） | 制定迁移计划 |

## 常用SQL诊断查询

```sql
-- 检查过站记录
SELECT * FROM station_record WHERE sn = 'XXX' ORDER BY created_at DESC LIMIT 10;

-- 检查绑定关系
SELECT * FROM binding_record WHERE sn = 'XXX';

-- 检查物料追溯
SELECT * FROM material_trace WHERE sn = 'XXX';

-- 检查n8n执行日志
SELECT * FROM execution_data WHERE workflow_id = 'XXX' ORDER BY started_at DESC LIMIT 10;

-- 检查Worker状态
SELECT * FROM worker_status WHERE status != 'active';
```

---

*持续更新，每次分析新workflow后补充。*
