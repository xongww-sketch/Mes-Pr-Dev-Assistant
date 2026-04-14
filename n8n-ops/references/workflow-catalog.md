# n8n Workflow 分类目录

> 版本：1.0 | 最近更新：2026-04-14
> 用途：322个workflow的快速查找目录

---

## 按业务域查找

### SN管理（23个）
- `SNgenerate_API` — SN生成主接口
- `uuid_blind_api` — UUID绑定
- `unblind_api` — 解绑
- `get_matialCode_by_sn` — SN查物料码
- 详见 `_core/n8n-process-map.md`

### 工单管理（10个）
- `get_jobmessage` — 工单消息（150节点，最大）
- `jobnumber_create_API` — 工单号创建
- `定时任务-从sap中获取工单信息` — SAP工单同步
- 详见 `_core/n8n-process-map.md`

### 包装管理（15个）
- `仓库扫码出库` — 仓库出库（135节点）
- `upload_box_info` — 箱信息上传
- 详见 `_core/n8n-process-map.md`

### 打印管理（13个）
- `updateprintinfo_API` — 更新打印信息
- `createprintinfo_API` — 创建打印信息
- 详见 `_core/n8n-process-map.md`

### 质检管理（8个）
- `insert_FQC_msg` — FQC消息
- `insert_OQC_msg` — OQC消息
- 详见 `_core/n8n-process-map.md`

### 维修管理（6个）
- `repair_message_line` — 维修产线消息（68节点）
- `repair_message` — 维修消息（50节点）
- 详见 `_core/n8n-process-map.md`

### 设备管理（10个）
- `resourceOnlineStatus` — 设备在线状态（⚠️高频）
- `ResouceLogin` — 资源登录
- 详见 `_core/n8n-process-map.md`

### 数据同步（20个）
- `[Sync]` 系列 — OMS渠道同步
- `手动同步` — 手动同步
- 详见 `_core/n8n-process-map.md`

### 外部对接（~70个）
- COROS后台（71个workflow涉及）
- SAP（21个workflow涉及）
- OMS（10个workflow涉及）
- 详见 `_core/n8n-process-map.md`

---

## 按紧急程度查找

### P0 关键
- `resourceOnlineStatus` — 曾导致worker崩溃
- `get_jobmessage` — 最大workflow
- `仓库扫码出库` — 出库核心

### P1 重要
- `SNgenerate_API` — SN生成
- `uuid_blind_api` — 绑定
- `unblind_api` — 解绑
- `upload_machine_data` — 数据上传

### P2 一般
- 报表查询类
- 通知类

---

*完整清单见 `_core/n8n-process-map.md`。*
