---
name: sn-management
version: 1.0
description: >
  SN管理SOP。覆盖SN生成、绑定、解绑、查询、虚拟SN的全流程。
  SN是MES的核心数据实体，此SOP处理所有SN相关问题。
category: execution
tags: [sn, serial-number, binding, unbinding, mes, sop]
depends_on:
  - _core/mes-domain-knowledge
  - _core/n8n-process-map
  - _core/team-context
  - _core/decision-framework
  - _router/problem-router
---

# SN 管理 SOP

> 版本：1.0 | 最近更新：2026-04-14
> 用途：SN生成、绑定、解绑、查询、虚拟SN的标准处理流程

---

## 1. SN 生命周期

```
生成 → 绑定(中框/FPC/电池/屏) → 过站 → 包装 → 出库 → 激活
  ↑        ↓
  └── 解绑 ←── 维修/返修
```

---

## 2. SN 生成

### 2.1 正常SN生成

**触发时机**：包装环节

**相关Workflow**：
- `SNgenerate_API` (41节点) — SN生成主接口
- `SN_ virtual_generate` (27节点) — 虚拟SN生成
- `deviceparts_retroid_generate` (27节点) — 设备部件返修SN生成

**处理流程**：
1. 确认工单状态（是否已下发）
2. 检查SN生成规则（机型/批次）
3. 调用SN生成接口
4. 验证SN生成结果
5. 记录生成日志

**常见问题**：
- SN生成失败 → 检查工单状态、数据库连接
- SN重复 → 检查唯一索引、清理重复数据
- SN规则变更 → 确认新规则、更新配置

### 2.2 虚拟SN生成

**使用场景**：测试、返修、样机

**相关Workflow**：
- `SN_ virtual_generate` (27节点)
- `get_v_sn_series` (16节点)

**注意事项**：
- 虚拟SN不能用于正式出库
- 虚拟SN需要特殊标记
- 虚拟SN转正式SN需要审批

---

## 3. SN 绑定

### 3.1 绑定类型

| 绑定类型 | 相关Workflow | 节点数 | 说明 |
|---------|-------------|--------|------|
| UUID绑定 | uuid_blind_api | 76 | 通过UUID绑定 |
| 条码绑定 | barcode_blind_newmes | 28 | 条码绑定（新MES） |
| 电池绑定 | battery_blind_api | 46 | 电池绑定 |
| 屏幕绑定 | screen_blind_api | 38 | 屏幕绑定 |
| FPC绑定 | fpc_bind_to_sn | 28 | FPC绑定到SN |
| 重新组装绑定 | reassy_blind_api | 30 | 重新组装绑定 |
| 备注绑定 | remark_blind_api | 24 | 备注绑定 |
| 加工前物料绑定 | binding_materials_before_processing | 33 | 加工前物料绑定 |
| 双物料绑定 | two_binding_materials_before_processing | 32 | 双物料加工前绑定 |

### 3.2 绑定流程

```
1. 确认SN状态（是否已生成、是否已绑定）
2. 确认绑定类型（中框/FPC/电池/屏）
3. 检查部件信息（批次、型号）
4. 调用绑定接口
5. 验证绑定结果
6. 记录绑定日志
```

### 3.3 绑定失败处理

**常见原因**：
- SN不存在 → 检查SN是否已生成
- SN已绑定 → 确认是否需要先解绑
- 部件信息错误 → 检查部件编码
- 数据库写入失败 → 检查PostgreSQL连接
- n8n workflow异常 → 检查对应workflow状态

**处理步骤**：
1. 查SN状态 → `select_all_by_sn`
2. 查绑定记录 → `selectBlind_api`
3. 确认失败原因
4. 制定处理方案
5. 执行修复
6. 验证结果

---

## 4. SN 解绑

### 4.1 解绑场景

| 场景 | 处理Workflow | 说明 |
|------|-------------|------|
| 维修解绑 | unblind_api (68节点) | 维修时部件解绑 |
| 返修解绑 | unblind_api | 返修时解绑 |
| 批量解绑 | unblind_api | 批量解绑操作 |

### 4.2 解绑流程

```
1. 确认SN和绑定记录
2. 确认解绑原因（维修/返修/错误绑定）
3. 检查解绑条件（是否允许解绑）
4. 调用解绑接口
5. 验证解绑结果
6. 记录解绑日志
```

### 4.3 解绑注意事项

- **返投工序测试的SN**：返投前测试数据不能解除绑定（需新增功能）
- **维修决策需要拆机时**：不能进行解绑操作（需新增功能）
- **已出库SN**：解绑需要审批

---

## 5. SN 查询

### 5.1 查询类型

| 查询类型 | 相关Workflow | 说明 |
|---------|-------------|------|
| 按SN查全部 | select_all_by_sn (20节点) | 查询SN全部信息 |
| 按SN查物料码 | get_matialCode_by_sn (105节点) | 按SN查物料码（最大） |
| 按SN查工单 | sn_get_joborder_API (7节点) | SN查工单 |
| 按SN查机型 | sn_get_modal_API (7节点) | SN查机型 |
| 按UUID查SN | uuidGetSn (12节点) | UUID查SN |
| 按UID查SN | get_sn_by_uid_API (13节点) | UID查SN |
| SN查绑定数据 | sn查绑定数据通用接口 (13节点) | SN查绑定数据 |
| 模糊查询 | uuid模糊查询报表 (10节点) | UUID模糊查询 |

### 5.2 查询流程

```
1. 确认查询条件（SN/UUID/UID）
2. 选择对应查询接口
3. 执行查询
4. 返回结果
```

---

## 6. SN 数据修复

### 6.1 常见数据问题

| 问题 | 原因 | 处理方案 |
|------|------|---------|
| SN数据缺失 | 同步失败 | 手动补数据/重新同步 |
| SN重复 | 生成规则问题 | 清理重复数据 |
| 绑定记录错误 | 操作失误 | 解绑后重新绑定 |
| SN状态不一致 | 新旧MES差异 | 数据校验+修复 |

### 6.2 修复流程

```
1. 确认问题数据范围
2. 评估影响（是否影响主流程）
3. 制定修复方案
4. 执行修复（SQL/n8n/脚本）
5. 验证修复结果
6. 记录修复过程
```

**决策点**（按 `_core/decision-framework.md`）：
- 影响工单/出库？→ P0立即修复
- 仅影响报表？→ P2排期修复
- 需要绕开正常流程？→ 需要熊旺确认

---

## 7. SN 上传外部系统

### 7.1 上传目标

| 目标 | 相关Workflow | 说明 |
|------|-------------|------|
| 国库 | sn上传到国库 (60节点) | SN上传国库 |
| 京东自营 | 同步SN信息到京东自营 (45节点) | 京东SN同步 |
| 管易 | 同步SN唯一码到管易 (39节点) | 管易SN同步 |
| 浙江校验平台 | sn上传到浙江校验平台 (43节点) | 浙江平台SN校验 |
| 抖音国补 | 抖音国补SN同步 | SN上传抖音后台 |

### 7.2 上传失败处理

```
1. 检查对应workflow状态
2. 检查外部系统连通性
3. 检查Token状态
4. 手动触发上传
5. 验证上传结果
```

---

*本文档是SN管理的标准SOP。*
*所有SN相关问题都应先经过 `_router/problem-router.md` 路由到此SOP。*
