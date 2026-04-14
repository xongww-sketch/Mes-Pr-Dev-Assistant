# 常见 Bug 模式库

> 版本：1.0 | 最近更新：2026-04-14
> 用途：记录常见Bug模式，加速诊断

---

## 1. SN相关

### 1.1 SN生成失败
**症状**：包装环节SN生成报错
**常见原因**：
- 工单状态异常（未下发/已关闭）
- SN规则配置错误
- 数据库写入失败
**检查Workflow**：`SNgenerate_API`
**检查SQL**：`SELECT * FROM work_order WHERE work_order_id = 'xxx';`

### 1.2 SN绑定失败
**症状**：扫码绑定报错
**常见原因**：
- SN不存在
- SN已绑定
- 部件信息错误
- Redis分布式锁冲突
**检查Workflow**：`uuid_blind_api`, `barcode_blind_newmes`
**检查点**：Redis锁状态、PostgreSQL连接

### 1.3 SN查询慢
**症状**：SN查询响应时间>5秒
**常见原因**：
- 缺少索引
- 数据量大
- 关联查询过多
**检查Workflow**：`get_matialCode_by_sn` (105节点)
**优化**：添加索引、优化查询

---

## 2. 工单相关

### 2.1 工单查询超时
**症状**：工单查询无响应
**常见原因**：
- `get_jobmessage` (150节点) 过大
- 数据库慢查询
**优化**：拆分workflow、添加索引

### 2.2 工单关闭无审批
**症状**：工单被随意关闭
**处理**：确认工单关闭审批流程已启用

---

## 3. n8n相关

### 3.1 Worker全挂
**症状**：所有workflow不执行
**处理**：`docker compose up -d`
**预防**：restart: unless-stopped + 健康监控

### 3.2 队列堆积
**症状**：workflow排队等待
**常见原因**：
- 高频workflow（resourceOnlineStatus）
- 慢workflow阻塞
**处理**：限流、增加worker、优化慢workflow

### 3.3 Token过期
**症状**：外部API调用失败
**检查Workflow**：`updateLarkToken`, `定时刷新京东Token`
**处理**：手动刷新Token、检查定时任务

---

## 4. 外部系统相关

### 4.1 SAP同步失败
**症状**：工单/物料同步失败
**检查Workflow**：`定时任务-从sap中获取工单信息`
**处理**：检查SAP接口连通性、手动触发同步

### 4.2 OMS渠道同步失败
**症状**：渠道数据不同步
**检查Workflow**：`[Sync]` 系列
**处理**：检查OMS接口、手动触发同步

### 4.3 COROS后台连接失败
**症状**：设备锁机/激活失败
**检查Workflow**：`设备锁机状态`, `sn上传到国库`
**处理**：检查COROS后台API连通性

---

## 5. 数据库相关

### 5.1 慢查询
**症状**：查询响应时间>10秒
**检查**：PostgreSQL慢查询日志
**处理**：添加索引、优化查询

### 5.2 连接池耗尽
**症状**：数据库连接失败
**检查**：`SELECT count(*) FROM pg_stat_activity;`
**处理**：增加连接池、优化长连接

### 5.3 磁盘空间不足
**症状**：写入失败
**检查**：`df -h`
**处理**：清理日志、扩容

## 6. 边缘情况关键词

当用户描述包含以下关键词时，优先检查对应场景：

| 关键词 | 可能场景 | 优先检查 |
|--------|----------|----------|
| "卡住"/"不动" | Worker挂死/队列堆积 | `docker ps`, execution_data表 |
| "超时"/"慢" | 慢查询/大workflow | get_jobmessage, 数据库索引 |
| "重复"/"多次" | 重复触发/幂等问题 | webhook去重逻辑 |
| "丢失"/"找不到" | 数据未写入/SN不存在 | station_record, binding_record |
| "报错"/"异常" | 通用错误 | 先看错误信息关键词 |
| "锁了"/"锁定" | 分布式锁冲突 | Redis锁状态 |
| "同步失败" | 外部API问题 | SAP/OMS/COROS接口 |
| "打印失败" | 打印机/模板问题 | 打印workflow, 模板配置 |
| "扫码失败" | 扫码枪/解析问题 | 扫码workflow, 条码格式 |
| "权限"/"不能操作" | 权限配置问题 | 用户权限表, 工单状态 |

## 7. 诊断SQL速查

```sql
-- 检查SN是否存在
SELECT sn, status, work_order_id FROM sn_record WHERE sn = 'XXX';

-- 检查SN过站历史
SELECT station_name, result, created_at FROM station_record 
WHERE sn = 'XXX' ORDER BY created_at DESC LIMIT 20;

-- 检查绑定关系
SELECT sn, component_sn, binding_type, created_at FROM binding_record 
WHERE sn = 'XXX' OR component_sn = 'XXX';

-- 检查工单状态
SELECT work_order_id, status, product_code, plan_qty FROM work_order 
WHERE work_order_id = 'XXX';

-- 检查n8n执行失败
SELECT id, workflow_id, status, error, started_at FROM execution_data 
WHERE status = 'error' AND started_at > NOW() - INTERVAL '1 hour' 
ORDER BY started_at DESC LIMIT 20;

-- 检查Worker状态
SELECT id, name, status, last_seen FROM worker_status ORDER BY last_seen DESC;

-- 检查数据库连接
SELECT count(*), state FROM pg_stat_activity GROUP BY state;

-- 检查慢查询（>5秒）
SELECT query, duration FROM pg_stat_activity 
WHERE state = 'active' AND duration > interval '5 seconds';

-- 检查磁盘空间
-- 需在服务器执行: df -h
```

---

*持续更新，每次处理新Bug后补充。*
