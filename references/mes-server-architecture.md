# COROS MES Server 架构文档

> 本文档描述 COROS MES（制造执行系统）Server 端的完整架构，涵盖技术栈、模块划分、数据模型、认证授权、核心业务流程及关键设计模式。

---

## 目录

- [1. 技术栈总览](#1-技术栈总览)
- [2. 项目结构](#2-项目结构)
- [3. 数据模型 (Prisma Schema)](#3-数据模型-prisma-schema)
- [4. 认证与授权](#4-认证与授权)
- [5. 自动审计机制](#5-自动审计机制)
- [6. n8n 集成](#6-n8n-集成)
- [7. 核心业务模块](#7-核心业务模块)
- [8. 公共基础设施](#8-公共基础设施)
- [9. 飞书集成](#9-飞书集成)
- [10. 设备管理](#10-设备管理)
- [11. 关键设计模式](#11-关键设计模式)
- [12. 环境配置](#12-环境配置)
- [13. 性能注意事项](#13-性能注意事项)

---

## 1. 技术栈总览

| 层级 | 技术 | 说明 |
|------|------|------|
| **后端框架** | NestJS | 企业级 Node.js 框架，模块化架构 |
| **ORM** | Prisma | 类型安全的数据库 ORM，含自动审计 Extension |
| **数据库** | PostgreSQL | 关系型数据库，主数据存储 |
| **缓存** | Redis | 用户信息缓存、Token 黑名单、验证码、限流 |
| **消息队列** | BullMQ | 基于 Redis 的队列，用于飞书事件异步处理 |
| **前端** | Vue3 + Vite | 基于若依 RuoYi-Vue3 全栈快速开发平台 |
| **外部集成** | 飞书 Lark SDK | OAuth 登录、审批流、事件推送 |
| **设备通信** | MQTT | 设备状态监控与指令下发 |
| **工作流引擎** | n8n | 大量业务逻辑通过 n8n HTTP 节点实现 |
| **文件传输** | FTP / SSH | 安装包分发、远程脚本执行 |
| **基础平台** | carole-admin | 全栈快速开发平台（基于若依体系） |

---

## 2. 项目结构

```
server/src/
├── admin/                    # Admin 后台管理模块（核心业务）
│   ├── admin.module.ts       # Admin 总模块，聚合所有子模块
│   ├── base-data/            # 基础数据管理（公司/工厂信息）
│   ├── common/               # 公共功能（验证码生成/文件上传）
│   ├── config/               # 工艺流程配置（ProcessConfig 表操作）
│   ├── dashboard/            # 仪表盘（产线监控/工单统计/生产数据汇总）
│   ├── data-query/           # 数据查询（SN/FQC/上位机/包装/维修/温湿度）
│   ├── device/               # 设备管理（MQTT 通信/SSH 远程执行/SSE 推送）
│   ├── installation-package/ # 安装包管理（版本控制/FTP 上传下载）
│   ├── operation/            # 工序管理（CRUD/SOP 文件/版本控制）
│   ├── print-template/       # 打印模板管理（标签打印模板 CRUD）
│   ├── process/              # 工艺流程管理
│   ├── product/              # 产品管理（物料/型号/颜色/类型）
│   ├── production-plan/      # 生产计划（排班/产线/工单关联）
│   ├── sn/                   # SN 管理（批量导入/编码规则/底壳旧 SN）
│   └── ticket/               # 工单管理（创建/批量创建/搜索）
│
├── common/                   # 公共基础设施（跨模块复用）
│   ├── auth/                 # 认证模块（JWT + 飞书 OAuth + RBAC）
│   ├── decorator/            # 自定义装饰器（@User() 等）
│   ├── domain/               # 公共 Domain 对象
│   ├── exception/            # 异常处理（BizException 统一业务异常）
│   ├── filter/               # 全局过滤器（异常过滤/请求过滤）
│   ├── ftp/                  # FTP 客户端封装
│   ├── guard/                # 守卫（AuthGuard / RoleGuard / ThrottleGuard）
│   ├── interceptor/          # 拦截器（ResponseInterceptor 统一响应格式）
│   ├── mqtt/                 # MQTT 通信封装
│   ├── service/              # 核心服务
│   │   ├── prisma/           # PrismaService（含自动审计 Extension）
│   │   └── redis/            # RedisService（缓存/锁/队列操作）
│   ├── ssh/                  # SSH 远程执行封装
│   └── utils/                # 工具函数（n8n-request 等）
│
├── lark/                     # 飞书集成模块
│   ├── approval/             # 飞书审批流（缺陷退货审批）
│   └── lark.module.ts        # Lark 事件 WebSocket 接收 + BullMQ 异步处理
│
├── search/                   # 搜索模块（FQC 超期预警）
│   └── search.service.ts     # FQC 超期检测服务
│
└── system/                   # 系统管理模块（RBAC 基础）
    ├── auth/                 # 登录/注销/Token 刷新
    ├── dept/                 # 部门管理
    ├── dict/                 # 字典管理
    ├── log/                  # 日志管理（操作日志/登录日志）
    ├── menu/                 # 菜单管理
    ├── role/                 # 角色管理
    └── user/                 # 用户管理
```

---

## 3. 数据模型 (Prisma Schema)

Prisma Schema 分为 **7 个文件**，按业务域拆分：

### 3.1 system.prisma — 系统管理

系统管理相关模型，支撑 RBAC 权限体系：

| 模型 | 说明 | 关键字段 |
|------|------|----------|
| **User** | 系统用户 | `userId` (BigInt), `userName`, `email`, `phonenumber`, `password` (bcrypt), `roleId`, `deptId`, `status` (0=正常/1=停用) |
| **Role** | 角色 | `roleId`, `roleName`, `roleKey`, `perms` (权限字符串) |
| **Menu** | 菜单/权限 | `menuId`, `menuName`, `perms`, `menuType` |
| **Dept** | 部门 | `deptId`, `deptName`, `parentId` |
| **Dict** | 字典 | 系统字典数据 |
| **Log** | 日志 | 操作日志、登录日志 |

### 3.2 commonPlan.prisma — 基类计划

- **CommonPlan**: 生产计划的基类，定义通用计划字段

### 3.3 productionPlan.prisma — 生产计划

- **ProductionPlan**: 继承 CommonPlan，扩展工单/产线/排班相关字段
  - 关联 `joborder`（工单）
  - 关联 `productionLine`（产线）
  - 排班信息

### 3.4 ticketLog.prisma — 工单日志

- 工单操作日志记录

### 3.5 dataLog.prisma — 数据审计日志

- **DataLog**: 自动记录所有写操作
  - `operator` — 操作者 ID
  - `requestId` — 请求 ID
  - `action` — 操作类型 (create/update/delete)
  - `modelName` — 模型名称
  - `recordId` — 记录 ID
  - `oldData` — 旧数据 (JSON)
  - `newData` — 新数据 (JSON)

### 3.6 deviceEnvironment.prisma — 设备环境

- 设备环境数据：温度、湿度、CO₂ 浓度等

### 3.7 introspected.prisma — 内省模型（核心业务实体）

| 模型 | 说明 | 关键字段 |
|------|------|----------|
| **Applemfi** | Apple MFi 认证记录 | — |
| **Resources** | 物料资源 | — |
| **Retroid** | **SN 设备标识（核心实体）** | `retroidId` — 关联所有生产数据的主键 |
| **MachineData** | 机器生产数据 | `data` (JSON) — 存储各工序的测试/包装数据 |
| **ProcedureDetail** | 工序详情 | `procedureName`, `currentVersion`, `checkrule` |
| **HistoryProcedureDetail** | 工序版本历史 | SOP、标准工时、标准人数、节拍 |
| **ProductInfo** | 产品信息 | `productCode` (物料号), `productModelId`, `productColor`, `productType` |
| **ProductModel** | 产品型号 | — |
| **Joborder** | 工单 | `joborderNumber`, `joborderType`, `productId`, `status` |
| **PrintTemplate** | 打印模板 | — |
| **InstallationPackage** | 安装包 | — |

> **Retroid 是核心枢纽实体**：所有生产数据（MachineData、ProcedureDetail、ProductInfo 等）均通过 `retroidId` 关联，形成 SN 全生命周期数据链。

---

## 4. 认证与授权

### 4.1 认证流程

```
用户登录 → 飞书 OAuth → 获取 email → 匹配 User 表 → 生成 JWT → 返回 Token
```

### 4.2 Token 机制

| Token 类型 | 有效期 | 用途 |
|-----------|--------|------|
| `access_token` | 30 分钟 | API 请求鉴权 |
| `refresh_token` | 较长 | 刷新 access_token |

### 4.3 Redis 缓存层

- **用户信息缓存**：JWT 解析后从 Redis 获取用户详情，避免重复查库
- **Token 黑名单**：注销/踢人时将 Token 加入黑名单
- **验证码**：登录验证码存储与校验

### 4.4 RBAC 权限模型

```
User → Role → Menu → Permission 字符串
```

- 权限字符串格式：`system:user:list`（模块:资源:操作）
- **超级管理员**：`roleKey = 'admin'` 或 `perms = '*:*:*'`，拥有所有权限
- **AuthGuard**：解析 JWT → Redis 查用户信息 → 注入 `@User()` 装饰器供 Controller 使用

### 4.5 安全机制

| 机制 | 实现 |
|------|------|
| IP 黑名单 | `SysBlackIp` 表，拦截黑名单 IP |
| 接口限流 | `ThrottlerModule`，防止接口滥用 |
| 密码加密 | bcrypt 哈希存储 |

---

## 5. 自动审计机制

### 5.1 实现原理

通过 **Prisma Client Extension** 拦截所有写操作：

```
create / update / delete / createMany / updateMany / deleteMany
  → 拦截器捕获
  → 提取 oldData / newData
  → 写入 DataLog 表
```

### 5.2 审计字段

| 字段 | 说明 |
|------|------|
| `operator` | 操作者 ID（从 RequestContext 获取） |
| `requestId` | 请求唯一 ID |
| `action` | 操作类型：create / update / delete |
| `modelName` | 被操作的模型名称 |
| `recordId` | 被操作记录的 ID |
| `oldData` | 操作前的完整数据 (JSON) |
| `newData` | 操作后的完整数据 (JSON) |

### 5.3 跳过审计的模型

以下模型的写操作 **不会** 记录审计日志：

- `DataLog`（避免递归）
- `Resources`
- `ProductionPlan`
- `ticketLog`
- `SysLogininfor`

### 5.4 上下文传递

使用 `cls-hooked`（`RequestContext`）在请求生命周期中传递 `userId` 和 `requestId`，确保审计日志能正确关联操作者。

---

## 6. n8n 集成

### 6.1 架构定位

n8n 作为 **工作流引擎**，承载了大量业务逻辑。MES Server 通过 HTTP 调用 n8n Webhook 实现业务编排。

### 6.2 技术实现

- **统一封装**：`n8n-request.ts` 工具函数，基于 axios
- **环境感知**：dev/test/staging 环境自动给 webhook 路径加 `_test` 后缀，prod 环境不加
- **调用模式**：同步 HTTP 请求 → n8n 执行工作流 → 返回结果

### 6.3 当前通过 n8n 实现的业务

| 业务模块 | n8n 接口 |
|---------|----------|
| 产品查询 | `getProductInfoSearch` |
| 产品创建 | `createProduct` |
| 工序管理 | 主要通过 n8n 工作流 |
| SN 管理 | 部分流程 |
| 工单管理 | 部分流程 |
| FQC 查询 | `get_fqc` |
| 包装查询 | `getPackageInfo` |
| 维修查询 | `get_repair_message` |
| FQC 超期预警 | UUID → retroid → FQC 工序查询链 |

### 6.4 迁移策略

**渐进式迁移**：将 n8n 工作流逐步迁移为 NestJS 原生 Prisma 实现。

- ✅ 已迁移：`updateProduct`、`deleteProduct`
- 🔄 进行中：其他模块逐步迁移
- 目标：减少对外部工作流引擎的依赖，提升性能和可维护性

---

## 7. 核心业务模块

### 7.1 SN 管理 (`admin/sn/`)

| 功能 | 说明 |
|------|------|
| 批量导入 | Excel 上传 → Multer 解析 → 逐条入库 |
| 编码规则 | 10 位数字 |
| 关联关系 | 关联 `productId` 和 `joborderId` |
| 底壳旧 SN | 支持底壳旧 SN 上传功能 |

### 7.2 工单管理 (`admin/ticket/`)

| 功能 | 说明 |
|------|------|
| CRUD | 工单增删改查 |
| 批量创建 | 支持批量创建工单 |
| 工单类型 | PACK（包装）等 |
| 状态管理 | `status`: 1=有效, 2=删除（逻辑删除） |
| 关联 | 关联生产计划和产品 |

### 7.3 产品管理 (`admin/product/`)

| 功能 | 说明 |
|------|------|
| 产品 CRUD | `productCode`（物料号）、`productModelId`（型号）、`productColor`（颜色）、`productType`（类型） |
| 三要素唯一性 | 模型 + 类型 + 颜色 组合唯一 |
| 物料号唯一性 | `productCode` 全局唯一 |
| 逻辑删除 | `status = 2` 标记删除 |
| 迁移状态 | `getProductInfoSearch`/`createProduct` 仍调 n8n；`updateProduct`/`deleteProduct` 已迁移 Prisma |

### 7.4 工序管理 (`admin/operation/`)

| 功能 | 说明 |
|------|------|
| 工序 CRUD | 增删改查 + SOP 文件上传（base64 编码） |
| 版本控制 | `ProcedureDetail`（当前版本）+ `HistoryProcedureDetail`（版本历史） |

#### 工序属性字典

| 属性 | 枚举值 |
|------|--------|
| `procedureType` | 1=加工, 2=搬运, 3=存储, 4=检验 |
| `procedureDataupload` | 1=无需上传, 2=需要上传 |
| `procedureScheduling` | 1=前加工, 2=组装测试, 3=成品测试, 4=包装 |
| `commonProcedure` | 是否通用工序 |
| `checkrule` | 检验规则 |
| `standardLabor` | 标准工时 |
| `standardPeople` | 标准人数 |
| `cycleTime` | 节拍 |

> 工序管理主要通过 n8n 工作流实现。

### 7.5 工艺流程配置 (`admin/config/`)

| 功能 | 说明 |
|------|------|
| ProcessConfig 表 | `processName`, `processId`, `config`（JSON 数组存储流程配置） |
| 软删除 | `isDeleted`: 0=未删除, 1=已删除 |
| 查询 | 支持分页查询 + 条件筛选 |

### 7.6 数据查询模块 (`admin/data-query/`)

这是系统中最复杂的模块，包含多个子服务：

#### 7.6.1 SN 查询

- 通过 `retroid` 查询 SN 全生命周期数据
- 串联所有关联的生产记录

#### 7.6.2 FQC 查询 (`fqc.service.ts`)

| 功能 | 说明 |
|------|------|
| 数据源 | n8n 的 `/get_fqc` 接口 |
| 查询条件 | 指定时间段的 FQC 测试数据 |
| 数据处理 | 内存过滤空记录 |

#### 7.6.3 上位机数据 (`host-computer.service.ts`) — 最复杂模块

**统计查询**：
- 按工序 + 产品型号 + 时间范围查询
- 计算指标：直通率 / 失败率 / 误测率

**状态判断逻辑**：

| 状态 | 判定条件 |
|------|----------|
| `DIRECT_PASS`（直通） | 第一次测试就 Pass |
| `MIS_TEST`（误测） | 多次测试，第一次非 Pass 但最后一次 Pass |
| `TRUE_FAIL`（真失败） | 第一次非 Pass 且最后一次也没过 |

**明细查询**：
- 返回失败列表和误测列表

**通用查询**：
- 支持 SN / UUID / 产品型号多种查询路径
- **UUID 解析**：支持 30 位 hex / 28 位 hex / URL 格式（`a=xxx&b=xxx&c=xxx`）
- SN 与 UUID 绑定/解绑记录追踪
- FTP 文件下载（测试数据文件）

**默认工序 ID 列表**：
```
[96, 95, 94, 93, 97, 125, 134, 135, 136, 148, 149, 155, 156, 140, 141]
```

#### 7.6.4 包装数据 (`package.service.ts`)

| 功能 | 说明 |
|------|------|
| 单工序查询 | 通过 n8n 的 `/getPackageInfo` |
| 聚合查询 | 制箱（工序 81）+ 称重（工序 79）按 SN 合并 |
| 包装工序列表 | 过滤 `procedure_dataupload=2 AND procedure_scheduling=4` |

**箱号解绑** (`deletePackageByBoxId`)：
1. 查询原始包装记录
2. 防重复检查
3. 插入解绑记录（`procedureDetailId = 91`）
4. 解绑记录 data 包含：`data_old`（完整备份）、`unbind_id`、`machine_data_id_old` 等回溯字段

**批量 SN 删除** (`deletePackageBatch`)：
1. Excel 上传 → 批量查询 → 逐 SN 处理
2. 每个 SN 取最近 2 条包装记录（称重 79 + 制箱 81）
3. 排除已有解绑记录（91）的数据
4. 使用原生 SQL（`NOT EXISTS` 子查询）

**删除历史查询**：
- 按 SN 或工单号查询 `procedureDetailId = 91` 的解绑记录

#### 7.6.5 维修数据 (`repair.service.ts`)

| 功能 | 说明 |
|------|------|
| 数据源 | n8n 的 `/get_repair_message` |
| 查询方式 | 支持 SN / UUID / 工号查询 |
| 分页 | 内存分页 |

### 7.7 FQC 超期预警 (`search/search.service.ts`)

**检测链路**：
```
输入 UUID → n8n 查 retroid_id → 查 FQC 工序（procedureName='FQC测试'）→ 查最新测试时间 → 判断是否超期
```

**超期标准**：距今 > 6 个月

### 7.8 Dashboard (`admin/dashboard/`)

| 功能 | 说明 |
|------|------|
| 产线实时监控 | 各产线运行状态、产量统计 |
| 工单状态统计 | 工单进度、完成率 |
| 生产数据汇总 | 综合生产指标 |

### 7.9 安装包管理 (`admin/installation-package/`)

| 功能 | 说明 |
|------|------|
| 版本管理 | 安装包版本控制 |
| FTP 传输 | 上传/下载通过 FTP |
| 元数据 CRUD | 软件包基本信息管理 |

### 7.10 打印模板 (`admin/print-template/`)

- 标签打印模板 CRUD

---

## 8. 公共基础设施

### 8.1 认证模块 (`common/auth/`)

- JWT 生成与验证
- 飞书 OAuth 集成
- RBAC 权限校验逻辑

### 8.2 异常处理 (`common/exception/`)

- **BizException**：统一业务异常类，携带错误码和消息
- 全局异常过滤器捕获并格式化响应

### 8.3 拦截器 (`common/interceptor/`)

- **ResponseInterceptor**：统一 API 响应格式
  ```json
  {
    "code": 200,
    "msg": "操作成功",
    "data": { ... }
  }
  ```

### 8.4 守卫 (`common/guard/`)

| 守卫 | 职责 |
|------|------|
| `AuthGuard` | JWT 解析 + 用户信息注入 |
| `RoleGuard` | 角色权限校验 |
| `ThrottleGuard` | 接口限流 |

### 8.5 PrismaService (`common/service/prisma/`)

- 封装 Prisma Client
- 集成自动审计 Extension
- 提供事务支持

### 8.6 RedisService (`common/service/redis/`)

- 用户信息缓存
- Token 黑名单管理
- 验证码存储
- 分布式锁

### 8.7 FTP 客户端 (`common/ftp/`)

- FTP 连接管理
- 文件上传/下载
- 用于安装包分发和测试数据文件下载

### 8.8 SSH 远程执行 (`common/ssh/`)

- SSH 连接管理
- 远程脚本执行
- 用于设备远程操作

### 8.9 MQTT 通信 (`common/mqtt/`)

- MQTT 客户端封装
- 设备状态订阅与发布
- 实时设备指令下发

### 8.10 工具函数 (`common/utils/`)

- `n8n-request.ts`：n8n HTTP 调用统一封装
- 其他通用工具函数

---

## 9. 飞书集成

### 9.1 审批流 (`lark/approval/`)

- **缺陷退货审批**：通过 Lark Approval API 发起/查询审批实例
- 审批结果回调处理

### 9.2 事件接收

- **WebSocket**：接收飞书推送的事件（审批回调、消息等）
- **BullMQ**：异步处理飞书消息，避免阻塞主流程

### 9.3 OAuth 登录

- 用户通过飞书扫码/授权登录
- 获取飞书用户 email → 匹配系统 User 表 → 生成 JWT

---

## 10. 设备管理

### 10.1 MQTT 通信

- 设备状态实时监控
- 指令下发与响应

### 10.2 SSH 远程执行

- 远程脚本执行
- 设备配置与维护

### 10.3 SSE 推送

- Server-Sent Events 实时推送设备状态更新到前端

### 10.4 Ping 检测

- 定期 Ping 检测设备在线状态

---

## 11. 关键设计模式

### 11.1 n8n → NestJS 渐进迁移

**现状**：系统处于混合架构状态，部分业务通过 n8n 工作流实现，部分已迁移为 NestJS 原生 Prisma 实现。

**迁移路径**：
1. 识别 n8n 工作流对应的业务逻辑
2. 在 NestJS 中用 Prisma 重写
3. 切换 API 路由指向新实现
4. 验证后下线 n8n 工作流

**已迁移示例**：`updateProduct`、`deleteProduct`

### 11.2 内存分页

**模式**：从 n8n 或数据库获取全量数据 → 在内存中进行分页/过滤/排序。

**影响**：
- ✅ 实现简单，快速迭代
- ⚠️ 数据量大时性能差，内存占用高
- ⚠️ 无法利用数据库索引优化

**改进方向**：迁移到数据库层分页（`skip`/`take`）

### 11.3 JSON data 字段

**模式**：`MachineData` 表的 `data` 字段使用 JSON 类型存储各工序的异构数据。

**优势**：
- 灵活适配不同工序的数据结构差异
- 无需为每种工序创建独立表

**劣势**：
- 无法利用数据库索引进行高效查询
- 数据校验依赖应用层

### 11.4 逻辑删除

**模式**：不执行物理删除，通过状态字段标记删除。

| 字段 | 值 | 含义 |
|------|-----|------|
| `status` | 1 | 有效 |
| `status` | 2 | 已删除 |
| `isDeleted` | 0 | 未删除 |
| `isDeleted` | 1 | 已删除 |

### 11.5 解绑记录模式

**模式**：删除操作不真删，而是插入一条解绑记录（`procedureDetailId = 91`），在 `data` 字段中保留完整原始数据备份。

**解绑记录 data 结构**：
```json
{
  "data_old": "完整原始数据备份",
  "unbind_id": "解绑操作 ID",
  "machine_data_id_old": "原机器数据 ID",
  "...": "其他回溯字段"
}
```

**优势**：
- 数据可追溯、可恢复
- 满足审计合规要求

---

## 12. 环境配置

### 12.1 多环境支持

| 环境 | 标识 | n8n Webhook 后缀 |
|------|------|-----------------|
| 开发 | `dev` | `_test` |
| 测试 | `test` | `_test` |
| 预发 | `staging` | `_test` |
| 生产 | `prod` | 无后缀 |

### 12.2 关键环境变量

| 变量 | 说明 |
|------|------|
| `DATABASE_URL` | PostgreSQL 连接字符串 |
| `REDIS_URL` | Redis 连接字符串 |
| `N8N_BASE_URL` | n8n 服务地址 |
| `JWT_SECRET` | JWT 签名密钥 |
| `LARK_APP_ID` / `LARK_APP_SECRET` | 飞书应用凭证 |
| `MQTT_BROKER_URL` | MQTT Broker 地址 |
| `FTP_HOST` / `FTP_USER` / `FTP_PASS` | FTP 连接配置 |
| `SSH_HOST` / `SSH_USER` / `SSH_KEY` | SSH 连接配置 |
| `NODE_ENV` | 运行环境标识 |

---

## 13. 性能注意事项

### 13.1 已知性能隐患

| 问题 | 位置 | 影响 |
|------|------|------|
| 内存分页 | 多数查询服务 | 大数据量时 OOM 风险 |
| n8n 同步调用 | 所有通过 n8n 的接口 | 增加延迟，n8n 故障时级联影响 |
| JSON 字段查询 | MachineData.data | 无法利用索引，全表扫描 |
| 全量拉取 + 内存过滤 | FQC 查询、维修查询 | 网络传输和内存双重浪费 |

### 13.2 优化建议

1. **数据库层分页**：将内存分页迁移为 Prisma `skip`/`take` 分页
2. **n8n 异步化**：非实时要求的 n8n 调用改为 BullMQ 异步任务
3. **JSON 字段索引**：对 MachineData.data 中的高频查询字段创建 PostgreSQL JSON 索引
4. **查询条件前置**：将过滤条件下推到 n8n/数据库层，减少传输数据量
5. **缓存热点数据**：产品型号、工序列表等低频变更数据加入 Redis 缓存

---

> **文档版本**: v1.0  
> **最后更新**: 2026-04-15  
> **维护者**: 虾王 🦐👑
