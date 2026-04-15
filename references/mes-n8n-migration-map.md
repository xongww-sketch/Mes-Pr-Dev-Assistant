# MES Server: n8n → NestJS 迁移状态图

## 迁移状态总览

### ✅ 已迁移到 NestJS (Prisma直连)

| 模块 | 功能 | 原n8n接口 | 迁移说明 |
|------|------|-----------|----------|
| 产品管理 | 更新产品 | - | 直接Prisma CRUD，含三要素唯一性校验 |
| 产品管理 | 删除产品 | - | 逻辑删除 status=2 |
| 产品管理 | 产品下拉列表 | - | Prisma findMany |
| 工序管理 | 搜索工序(工艺流程用) | - | Prisma + HistoryProcedureDetail关联查询 |
| 工序管理 | 工序下拉列表 | - | Prisma findMany |
| 工艺配置 | 全部CRUD | - | 完全Prisma实现 |
| 上位机 | 统计查询 | - | Prisma原生SQL，复杂的直通/误测/失败分析 |
| 上位机 | 明细查询 | - | Prisma原生SQL |
| 上位机 | 通用查询(新版) | - | Prisma原生SQL，支持SN/UUID/型号 |
| 上位机 | UUID转SN(新版) | - | Prisma原生SQL，完全本地实现 |
| 上位机 | 数据详情 | - | Prisma |
| 上位机 | 文件下载 | - | FTP直连 |
| 包装 | 箱号解绑 | - | Prisma，含防重复逻辑 |
| 包装 | 批量SN解绑 | - | Prisma + 原生SQL(NOT EXISTS子查询) |
| 包装 | 解绑历史查询 | - | Prisma，支持JSON path查询 |
| FQC预警 | FQC超期检查 | 部分 | n8n查UUID→SN + Prisma查FQC记录 |
| 系统管理 | 全部 | - | 完全Prisma实现(用户/角色/菜单/部门/字典/日志) |
| 认证 | 全部 | - | JWT + Redis + 飞书OAuth |

### 🔄 仍在使用n8n

| 模块 | 功能 | n8n接口 | 迁移建议 |
|------|------|---------|----------|
| 产品管理 | 获取产品模型 | /getproductmodel_API | 简单查询，易迁移 |
| 产品管理 | 搜索产品信息 | /getproductinfo_API | 含分页，中等复杂度 |
| 产品管理 | 创建产品 | /createproduct_API | 需要了解n8n内的额外逻辑 |
| 产品管理 | 创建产品模型 | /createproductmodel_API | 简单，易迁移 |
| 工序管理 | 搜索工序(全量) | /getprocedure_API | 已有Prisma版本(searchForProcess)，可替换 |
| 工序管理 | 创建工序 | /createprocedure_API | 含SOP文件处理，中等复杂度 |
| 工序管理 | 更新工序 | /updateprocedure_API | 含SOP文件处理，中等复杂度 |
| 工序管理 | 删除工序 | /deleteprocedure_API | 简单，易迁移 |
| 工序管理 | 下载工序文件 | /downloadprocedure_API | 需迁移到FTP直连 |
| FQC查询 | FQC数据查询 | /get_fqc | 需了解n8n查询逻辑 |
| 包装查询 | 单工序/聚合查询 | /getPackageInfo | 数据在MachineData表，可迁移 |
| 包装查询 | 包装工序列表 | /getprocedure_API | 同工序管理，已有替代 |
| 维修查询 | 维修记录 | /get_repair_message | 需了解n8n查询逻辑 |
| 上位机 | 通用查询(旧版) | /select_common_api | 已有新版Prisma实现(getHostComputerCommons) |
| FQC预警 | UUID转SN | /uuidGetSn | 上位机模块已有Prisma实现(uuidGetSn方法) |

## 关键业务逻辑详解

### 1. 上位机测试数据分析逻辑

这是系统中最复杂的业务逻辑，核心在于对SN(retroid)的测试结果进行分类统计。

**数据来源**: machine_data表，关联retroid/procedure_detail/joborder/product_info

**分析维度**: 按工序(procedure_detail_id)分组，再按SN(retroid_id)分组

**状态判定规则**:
1. 将同一SN在同一工序下的所有测试记录按时间升序排列
2. 看第一条的Result和最后一条的Result:
   - 第一条 = "pass" → **直通(DIRECT_PASS)** ✅
   - 第一条 ≠ "pass" + 最后一条 = "pass" + 记录数>1 → **误测(MIS_TEST)** ⚠️
   - 第一条 ≠ "pass" + (只有1条 或 最后一条 ≠ "pass") → **真失败(TRUE_FAIL)** ❌

**统计指标**:
- total_test_amount: 投入总数(唯一SN数)
- successful_passage_count: 直通数
- false_measurement_count: 误测数
- failed_count: 失败数
- success_count: 直通+误测(最终通过)
- pass_rate: 直通率
- good_rate: 最终通过率
- fail_rate: 失败率
- false_rate: 误测率

**明细输出**:
- failList: 真失败SN列表(取每个SN的第一条失败记录，去重)
- falseFailList: 误测SN列表(取每个SN的最后一条pass记录)

### 2. 包装数据解绑逻辑

**核心概念**: 解绑不是物理删除，而是插入一条 procedureDetailId=91 的特殊记录

**单箱解绑流程**:
1. 校验工单号存在(joborder表, joborderType='PACK')
2. 查询原始生产数据(排除procedureDetailId=91的记录)
3. 防重复: 检查是否已有对应的解绑记录(通过machine_data_id_old关联)
4. 只处理未解绑的记录
5. 插入解绑记录:
   ```json
   {
     "retroidId": "原始retroidId",
     "procedureDetailId": 91,
     "data": {
       "data_old": "{完整原始数据备份}",
       "unbind_id": "原始bind_id",
       "unbind_name": "箱号解绑",
       "machine_data_id_old": "原始记录ID(关联钥匙)",
       "user_id_old": "原始操作者",
       "createtime_old": "原始时间",
       ...
     }
   }
   ```

**批量SN解绑流程**:
1. Excel上传解析SN列表
2. 批量查询数据库中存在的SN
3. 不存在的SN → 记录失败原因
4. 每个SN: 原生SQL查最近2条包装记录(称重79+制箱81), 排除已解绑的(NOT EXISTS)
5. 无记录 → 跳过(skipCount)
6. 有记录 → 执行解绑插入
7. 返回结果: { successCount, failCount, skipCount, failDetails }

### 3. UUID解析与SN绑定追踪

**UUID格式支持**:
- 30位hex: 前16位 + 跳过2位 + 后12位 = 28位uuidUnique
- 28位hex: 直接使用
- URL格式: a=xxx{hex}&b={hex}&c={hex} → 拼接后同30位处理

**绑定查询流程**:
1. 解析UUID → 提取uuidUnique
2. 查工序ID: "SN与UUID绑定" 和 "物料解除绑定"
3. 用uuidUnique查最新绑定记录(machine_data表)
4. 查该retroid_id下的所有绑定/解绑记录
5. 如果最新记录是解绑记录 → 返回tag=1(无有效绑定)
6. 否则返回tag=0 + 绑定数据

### 4. 工序版本管理

**双表结构**:
- ProcedureDetail: 工序基本信息(名称/版本/状态)
- HistoryProcedureDetail: 工序版本历史(SOP/工时/人数/节拍/调度类型)

**查询时**: 
1. 先查ProcedureDetail获取基本信息
2. 再查HistoryProcedureDetail获取最新版本(按historyProcedureId DESC, DISTINCT procedureDetailId)
3. 关联ProductModel和User补充名称

### 5. 自动审计机制

**实现**: Prisma Client Extension
**触发**: 所有 create/update/delete/createMany/updateMany/deleteMany
**记录**: DataLog表 { operator, requestId, action, modelName, recordId, oldData, newData }
**排除**: DataLog自身, Resources, ProductionPlan, ticketLog, SysLogininfor
**上下文**: cls-hooked RequestContext 传递 userId + requestId

## 迁移优先级建议

### P0 (立即)
- /getprocedure_API → 已有 searchForProcess 可替代
- /uuidGetSn → 已有本地 uuidGetSn 方法
- /select_common_api → 已有 getHostComputerCommons 方法

### P1 (短期)
- /getproductmodel_API → 简单Prisma查询
- /createproductmodel_API → 简单Prisma插入
- /deleteprocedure_API → 简单Prisma更新status

### P2 (中期)
- /getproductinfo_API → 需理解n8n内的查询+过滤逻辑
- /createproduct_API → 需理解n8n内的额外校验
- /getPackageInfo → 查MachineData表，需确认n8n内的关联逻辑

### P3 (后期)
- /createprocedure_API, /updateprocedure_API → 含SOP文件处理
- /get_fqc → 需理解FQC数据结构
- /get_repair_message → 需理解维修数据来源
- /downloadprocedure_API → 迁移到FTP直连
