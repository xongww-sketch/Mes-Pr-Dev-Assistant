---
name: n8n-process-map
version: 1.0
description: >
  n8n流程地图。322个workflow的完整分类、关键路径、风险登记、运维SOP。
  是所有涉及n8n运维、故障排查、流程分析的Skill的知识根基。
category: knowledge
tags: [n8n, workflow, ops, troubleshooting, coros, mes]
depends_on:
  - _core/mes-domain-knowledge
---

# n8n 流程地图

> 生成时间：2026-04-14 | 数据来源：MES主生产实例完整备份（322个workflow）
> 实例：10.128.11 (n8n.pd.coros.team) | 版本：1.113.2 | Node.js 22.19.0

---

## 1. 总览

| 指标 | 数值 |
|------|------|
| 总Workflow | 322 |
| 活跃 | 266 (82.6%) |
| 非活跃 | 56 (17.4%) |
| 总节点数 | ~8,500+ |
| 最大Workflow | `get_jobmessage` (150节点) |

### 1.1 按业务域分布

| 业务域 | 数量 | 核心Workflow |
|--------|------|-------------|
| SN管理 | 23 | SNgenerate_API, get_matialCode_by_sn |
| 绑定/解绑 | 20 | uuid_blind_api, unblind_api |
| 工序/工艺 | 25 | createprocedure_API, PPG_dispensing_api |
| 工单管理 | 10 | get_jobmessage, jobnumber_create_API |
| 包装管理 | 15 | 仓库扫码出库(135节点) |
| 打印管理 | 13 | updateprintinfo_API, createprintinfo_API |
| 质检管理 | 8 | insert_FQC_msg, insert_OQC_msg |
| 维修管理 | 6 | repair_message_line(68节点) |
| 设备/资源 | 10 | resourceOnlineStatus(⚠️高频) |
| 数据同步 | 20 | OMS渠道同步, SAP工单同步 |
| 外部对接 | ~70 | COROS后台71, SAP 21, OMS 10 |
| 报表看板 | ~10 | 工厂日报, 设备锁机状态 |
| 系统管理 | ~30 | Token管理, 配置, 产品管理 |
| 测试/废弃 | ~30 | 可清理 |

---

## 2. 关键Workflow清单（P0关注）

### 2.1 最高频/最关键

| Workflow | 节点 | 活跃 | 风险 | 说明 |
|----------|------|------|------|------|
| `resourceOnlineStatus` | 4 | ✅ | 🔴🔴🔴 | 设备在线状态，高频触发，曾导致worker崩溃P1事故 |
| `get_jobmessage` | 150 | ✅ | 🔴🔴 | 工单消息查询，最大workflow |
| `仓库扫码出库` | 135 | ✅ | 🔴🔴 | 仓库出库核心流程 |
| `get_matialCode_by_sn` | 105 | ✅ | 🔴🔴 | SN查物料码，高频调用 |
| `jobnumber_create_API` | 99 | ✅ | 🔴🔴 | 工单号创建 |
| `upload_box_info` | 47 | ✅ | 🔴 | 箱信息上传 |
| `upload_machine_data` | 49 | ✅ | 🔴 | 机器数据上传 |
| `unblind_api` | 68 | ✅ | 🔴 | 解绑API |
| `repair_message_line` | 68 | ✅ | 🔴 | 维修产线消息 |
| `uuid_blind_api` | 76 | ✅ | 🔴 | UUID绑定API |

### 2.2 SN管理核心

| Workflow | 节点 | 说明 |
|----------|------|------|
| SNgenerate_API | 41 | SN生成主接口 |
| SN_ virtual_generate | 27 | 虚拟SN生成 |
| get_matialCode_by_sn | 105 | 按SN查物料码（最大之一） |
| get_matialCode_by_sn_oversea | 86 | 海外按SN查物料码 |
| uuid_blind_api | 76 | UUID绑定API |
| unblind_api | 68 | 解绑API |
| barcode_blind_newmes | 28 | 条码绑定（新MES） |
| battery_blind_api | 46 | 电池绑定API |
| screen_blind_api | 38 | 屏幕绑定API |
| fpc_bind_to_sn | 28 | FPC绑定到SN |

### 2.3 外部系统对接

#### COROS后台（71个workflow涉及）
- `MesLoginApi` — MES登录
- `sn上传到国库` — SN上传国库
- `设备锁机状态` — 设备锁机
- `样机试用` / `样机试用到期提醒` — 样机管理
- `飞书试用机退款流程` — 飞书退款
- `updateLarkToken` — 飞书Token更新
- `定时刷新京东Token` — 京东Token
- `工厂日报` / `发货月报` — 报表

#### SAP（21个workflow涉及）
- `定时任务-从sap中获取工单信息` — SAP工单同步
- `get_sap_material` — SAP物料查询
- `schedule_get_component_info` — 组件信息SAP同步
- `sn上传到浙江校验平台` — SN上传浙江平台

#### OMS（10个workflow涉及）
- `[Sync]` 系列 — OMS渠道同步
- `[sync]同步日本/越南工单` — 海外工单
- `手动同步` — 手动OMS同步

---

## 3. 技术架构

### 3.1 节点类型分布

| 节点类型 | 数量 | 说明 |
|----------|------|------|
| If条件判断 | 1,310 | 业务逻辑分支 |
| PostgreSQL | 1,081 | 数据库操作（主力存储） |
| RespondToWebhook | 1,015 | Webhook响应 |
| Code代码节点 | 1,002 | 自定义JS逻辑 |
| Webhook触发 | 242 | 外部触发入口 |
| HTTP Request | 135 | 外部API调用 |
| SQL Server | 112 | 旧系统数据库 |
| ScheduleTrigger | 46 | 定时触发 |
| Email发送 | 96 | 邮件通知 |
| MongoDB | 24 | 文档存储 |
| FTP | 17 | 文件传输 |

### 3.2 系统依赖

```
触发层: Webhook(246) + 定时(46) + 手动(39) + 表单(16)
        ↓
数据层: PostgreSQL(248个workflow) + SQL Server(112节点) + MongoDB(24) + Redis(6)
        ↓
外部系统: MES(259) → COROS后台(71) → SAP(21) → OMS(10) → 管易/京东/抖音/飞书
        ↓
通知层: Email(53) + 飞书消息
```

---

## 4. 风险登记册

### 4.1 高风险项

| # | 风险 | 影响 | 状态 | 缓解措施 |
|---|------|------|------|---------|
| 1 | resourceOnlineStatus高频触发 | 全部产线 | ⚠️ 已知 | 添加限流/改为推送 |
| 2 | Worker无自动重启 | 全部业务 | ✅ 已修复 | Docker restart: unless-stopped |
| 3 | 超大workflow难维护 | 核心业务 | ⚠️ 待处理 | 拆分子流程 |
| 4 | 新旧MES并存 | 数据一致性 | ⚠️ 待迁移 | 制定迁移计划 |
| 5 | ~50个无文档流程 | 维护困难 | ⚠️ 待补充 | 编写文档 |
| 6 | 测试workflow未清理 | 生产环境 | ⚠️ 待清理 | 删除~15个测试流程 |
| 7 | SQL Server依赖 | 迁移风险 | ⚠️ 待迁移 | 迁移到PostgreSQL |
| 8 | 核心业务单点依赖n8n | 全部 | ⚠️ 长期 | 迁移到新MES NestJS |

### 4.2 可清理清单（~30个）

#### 测试workflow（~15个）
- `Date&Time测试`, `Mogodb测试`, `S3测试`, `for test`, `测试`
- `并行分支测试-获取ip地址`, `条码绑定程序_API-ForTEST`
- `get_sap_material_test`, `getPackageUpdateStatus`
- `cplc_test`, `testMachinesearch_API`
- `convert to file 中文文件名测试`, `test飞书试用机退款流程`

#### 个人workflow（~10个）
- `My Sub-Workflow 1-4`
- `我的工作流 6-17`（部分）

#### 空/废弃workflow（~5个）
- `get_uuid` (0节点)
- `工序防呆通用接口` (1节点)
- `job_info_cache` (1节点, 非活跃)
- `同步SN信息到京东旗舰店` (0节点, 非活跃)

---

## 5. 故障排查速查

### 5.1 常见问题 → 对应Workflow

| 问题 | 检查Workflow | 检查点 |
|------|-------------|--------|
| SN生成失败 | SNgenerate_API | 是否活跃、PostgreSQL连接 |
| 绑定失败 | uuid_blind_api, barcode_blind_newmes | Webhook是否正常、Redis锁 |
| 解绑失败 | unblind_api | 数据拆分节点、PostgreSQL |
| 工单查询慢 | get_jobmessage(150节点) | 数据库索引、查询优化 |
| 出库失败 | 仓库扫码出库(135节点) | COROS后台API、HTTP请求 |
| 物料码查询失败 | get_matialCode_by_sn(105节点) | SAP/OMS连接、MySQL连接 |
| 设备离线 | resourceOnlineStatus | 高频调用、worker负载 |
| 渠道同步失败 | [Sync]系列 | 定时任务、OMS接口 |
| Token过期 | updateLarkToken, 定时刷新京东Token | Redis缓存、定时触发 |
| 邮件未发送 | Email节点(96个) | SMTP配置、邮件队列 |

### 5.2 紧急处理流程

```
1. 确认问题范围 → 单个workflow还是全部？
2. 检查worker状态 → docker ps | grep n8n
3. 检查日志 → docker logs n8n-worker --tail 100
4. 如果是worker挂了 → docker compose up -d
5. 如果是单个workflow → 检查是否活跃、节点配置
6. 如果是数据库问题 → 检查PostgreSQL连接、慢查询
7. 如果是外部系统 → 检查对应API连通性
8. 记录处理过程 → 更新运维手册
```

---

## 6. 新旧MES对照

### 6.1 标注"旧MES"的Workflow
- `MFI写入工序-新mes` (❌ 非活跃)
- `MFI工序写入返回-新mes` (❌ 非活跃)
- `Wi-Fi_mac_API-newmes` (✅ 活跃)
- `barcode_blind_newmes` (✅ 活跃)
- `getSNbyuid-旧MES` (✅ 活跃)

### 6.2 迁移状态
- **已迁移**：大部分核心workflow已迁移到新MES
- **待迁移**：部分旧MES标注的workflow仍在运行
- **已废弃**：标注"新MES"但非活跃的旧版本

---

## 7. 运维建议

### 7.1 立即执行（P0）
1. `resourceOnlineStatus` 添加调用频率限制
2. Worker健康监控 + 飞书告警
3. 清理~30个测试/个人/空workflow

### 7.2 短期（1-2周）
4. 拆分超大workflow（>100节点）
5. 合并重复版本（selectBlind_api有3个版本等）
6. 建立workflow台账（用途/负责人/生命周期）

### 7.3 中期（1-3月）
7. 新旧MES迁移计划
8. SQL Server → PostgreSQL迁移
9. 编写运维Runbook

### 7.4 长期（3-6月）
10. 核心业务从n8n迁移到新MES NestJS
11. n8n定位收敛（仅保留接口编排/数据同步/通知）

---

*本文档是n8n流程知识的浓缩版，详细分析见 n8n-workflow-map.md。*
*所有涉及n8n的Skill都应引用本文档。*
