# database-ops — MES数据库操作

> 版本：1.0 | 2026-04-15
> 用途：MES数据库查询、同步、排障

---

## 数据库连接信息

| 数据库 | 类型 | 地址 | Schema |
|--------|------|------|--------|
| mes (主库) | PostgreSQL 18 | pgsql.mes.coros.team:5432 | mes, mes_jp, public, sync |
| warehouseDelivery | PostgreSQL 18 | pgsql.mes.coros.team:5432 | public |
| YFConsumer (旧MES) | SQL Server 10.50 | 192.168.128.7:1433 | dbo |
| oms-service | MongoDB 6.0 | localhost:27017 | - |

---

## 常用查询

### 按SN查全生命周期

```sql
-- 1. 旧MES绑定记录
SELECT * FROM YFConsumer.dbo.consm_barcodebindinfo 
WHERE retroid = 'SN1234567890' OR uuid = 'xxx';

-- 2. 日本生产数据
SELECT * FROM mes_jp.machine_data 
WHERE retroid = 'SN1234567890';

-- 3. 激活记录
SELECT * FROM public.activation_records 
WHERE retroid = 'SN1234567890';

-- 4. 发货记录
SELECT * FROM warehouseDelivery.public.mes_delivery_order_details 
WHERE retroid = 'SN1234567890';

-- 5. OMS库存
db.sninventories.findOne({ sn: 'SN1234567890' });
```

### 按工单号查生产数据

```sql
-- SAP工单缓存
SELECT * FROM sync.sap_job_info_cache 
WHERE job_number = 'WO20260415001';

-- SN物料映射
SELECT * FROM sync.sn_material_code 
WHERE job_number = 'WO20260415001';

-- 日本生产数据
SELECT * FROM mes_jp.machine_data 
WHERE job_number = 'WO20260415001';

-- OMS库存
db.sninventories.find({ workOrderNo: 'WO20260415001' });
```

### 按订单号查发货

```sql
-- 发货订单
SELECT * FROM warehouseDelivery.public.delivery_order 
WHERE order_number = 'ORD20260415001';

-- 发货明细
SELECT * FROM warehouseDelivery.public.mes_delivery_order_details 
WHERE order_number = 'ORD20260415001';

-- 发货日志
SELECT * FROM warehouseDelivery.public.delivery_log 
WHERE order_number = 'ORD20260415001';

-- OMS订单
db.customerorders.findOne({ orderNo: 'ORD20260415001' });
```

### 同步任务状态

```sql
-- 查看任务状态
SELECT j.name, j.status, j.last_nosql_id, j.last_n8n_execution_id
FROM sync.jobs j;

-- 查看待处理任务
SELECT pt.*, j.name as job_name
FROM sync.pending_task pt
JOIN sync.jobs j ON pt.job_id = j.id
WHERE pt.task_state != 1
ORDER BY pt.task_start_time DESC;

-- 查看最近执行记录
SELECT t.*, j.name as job_name
FROM sync.task t
JOIN sync.jobs j ON t.job_id = j.id
ORDER BY t.task_start_time DESC
LIMIT 20;
```

### 激活/不良分析

```sql
-- 某设备激活历史
SELECT * FROM public.activation_records 
WHERE retroid = 'SN1234567890'
ORDER BY activatetime DESC;

-- 不良记录
SELECT dr.*, ar.firmware_type, ar.country_code
FROM public.defective_records dr
LEFT JOIN public.activation_records ar ON dr.active_id = ar.id
WHERE dr.retroid = 'SN1234567890';

-- 按国家统计激活
SELECT country_code, COUNT(*) as count
FROM public.activation_records
GROUP BY country_code
ORDER BY count DESC
LIMIT 20;

-- 按固件类型统计不良
SELECT firmware_type, COUNT(*) as defective_count
FROM public.defective_records dr
JOIN public.activation_records ar ON dr.active_id = ar.id
GROUP BY firmware_type
ORDER BY defective_count DESC;
```

---

## n8n 工作流关联

| n8n流程 | 数据库 | 表 | 说明 |
|---------|--------|-----|------|
| kRCqKO9jDbPn09W9 | mes_jp | machine_data | 日本CSV上传 |
| get_channel | warehouseDelivery | mes_delivery_order_details | 渠道拉取 |
| get_channel | public | delivery_log | 国内发货数据 |
| get_mespackingdata | warehouseDelivery | delivery_order_details | 包装数据 |
| get_mesProductionData | sync | yf_pcba_data | PCBA数据 |
| 每日同步 | mes_daily_data | - | 同步给软件部 |

---

## 排障指南

### 同步任务卡住

```sql
-- 1. 检查pending_task
SELECT * FROM sync.pending_task 
WHERE task_state = 0 
ORDER BY task_start_time ASC;

-- 2. 检查jobs状态
SELECT * FROM sync.jobs WHERE status != 1;

-- 3. 检查最近执行结果
SELECT * FROM sync.task 
WHERE task_result LIKE '%error%' 
ORDER BY task_start_time DESC 
LIMIT 10;
```

### SN找不到

```sql
-- 1. 旧MES
SELECT * FROM YFConsumer.dbo.consm_barcodebindinfo 
WHERE retroid = 'SN' OR uuid_unique = 'SN';

-- 2. 新MES
SELECT * FROM mes_jp.machine_data WHERE retroid = 'SN';

-- 3. OMS
db.sninventories.findOne({ sn: 'SN' });
db.sns.findOne({ sn: 'SN' });
db.snmaps.findOne({ sn: 'SN' });
```

### 发货状态异常

```sql
-- 检查发货状态
SELECT * FROM warehouseDelivery.public.mes_delivery_order_details 
WHERE delivery_status != 1 
ORDER BY create_time DESC 
LIMIT 20;

-- 检查退款
SELECT * FROM warehouseDelivery.public.refund 
WHERE refund_status != 1;
```

---

*持续更新，每次新增查询或排障经验后补充。*
