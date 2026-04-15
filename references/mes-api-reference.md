# COROS MES Server API 接口参考文档

> 最后更新: 2026-04-15

---

## Admin 模块 API

### 产品管理 `/admin/product`

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| GET | `/admin/product/model` | 获取产品模型列表 | n8n `/getproductmodel_API` |
| GET | `/admin/product/list` | 获取产品下拉列表(id+code) | Prisma `productInfo` |
| POST | `/admin/product/search` | 搜索产品信息(分页) | n8n `/getproductinfo_API` |
| POST | `/admin/product` | 创建产品 | n8n `/createproduct_API` |
| POST | `/admin/product/model` | 创建产品模型 | n8n `/createproductmodel_API` |
| PUT | `/admin/product` | 更新产品 | Prisma(已迁移) |
| DELETE | `/admin/product/:id` | 删除产品(逻辑删 status=2) | Prisma(已迁移) |

### 工序管理 `/admin/operation`

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| POST | `/admin/operation/search` | 搜索工序(全量) | n8n `/getprocedure_API` |
| POST | `/admin/operation/searchForProcess` | 搜索工序(工艺流程用,含版本历史) | Prisma(已迁移) |
| GET | `/admin/operation/list` | 工序下拉列表 | Prisma |
| POST | `/admin/operation` | 创建工序(含SOP文件上传) | n8n `/createprocedure_API` |
| PUT | `/admin/operation` | 更新工序 | n8n `/updateprocedure_API` |
| DELETE | `/admin/operation` | 删除工序 | n8n `/deleteprocedure_API` |
| GET | `/admin/operation/download` | 下载工序SOP文件 | n8n `/downloadprocedure_API` |

### 工艺流程配置 `/admin/config`

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| POST | `/admin/config` | 创建配置 | Prisma `processConfig` |
| PUT | `/admin/config` | 更新配置 | Prisma |
| GET | `/admin/config/:id` | 获取配置详情 | Prisma |
| DELETE | `/admin/config` | 删除配置(软删 isDeleted=1) | Prisma |
| GET | `/admin/config/list` | 分页查询配置 | Prisma |

### SN管理 `/admin/sn`

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| POST | `/admin/sn/import` | 批量导入SN(Excel) | Prisma + n8n |
| GET | `/admin/sn/list` | SN列表查询 | Prisma/n8n |

### 工单管理 `/admin/ticket`

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| POST | `/admin/ticket/search` | 搜索工单 | n8n/Prisma |
| POST | `/admin/ticket` | 创建工单 | n8n/Prisma |
| POST | `/admin/ticket/batch` | 批量创建工单 | n8n/Prisma |
| PUT | `/admin/ticket` | 更新工单 | n8n/Prisma |
| DELETE | `/admin/ticket` | 删除工单 | n8n/Prisma |

### 数据查询 `/admin/data-query`

#### FQC查询

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| POST | `/admin/data-query/fqc` | 查询FQC测试数据 | n8n `/get_fqc` |

#### 上位机数据

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| POST | `/admin/data-query/host-computer` | 上位机统计数据(直通率/失败率/误测率) | Prisma原生SQL |
| POST | `/admin/data-query/host-computer/detail` | 上位机明细(失败+误测列表) | Prisma原生SQL |
| POST | `/admin/data-query/host-computer/common` | 通用上位机查询(SN/UUID/型号) | Prisma原生SQL + n8n |
| GET | `/admin/data-query/host-computer/detail/:id` | 单条机器数据详情 | Prisma |
| GET | `/admin/data-query/host-computer/download` | 下载测试数据文件 | FTP |

#### 包装数据

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| POST | `/admin/data-query/package` | 单工序包装查询 | n8n `/getPackageInfo` |
| POST | `/admin/data-query/package/all` | 聚合包装查询(制箱+称重) | n8n(并行) |
| GET | `/admin/data-query/package/operations` | 包装工序列表 | n8n `/getprocedure_API` |
| POST | `/admin/data-query/package/delete-by-box` | 按箱号解绑 | Prisma |
| POST | `/admin/data-query/package/delete-batch` | 批量SN解绑(Excel) | Prisma+原生SQL |
| POST | `/admin/data-query/package/delete-history` | 查询解绑历史 | Prisma |

#### 维修数据

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| POST | `/admin/data-query/repair` | 查询维修记录 | n8n `/get_repair_message` |

### Dashboard `/admin/dashboard`

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| GET | `/admin/dashboard/stats` | 产线统计数据 | Prisma/n8n |

### 设备管理 `/admin/device`

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| GET | `/admin/device/list` | 设备列表 | MQTT/Prisma |
| POST | `/admin/device/ping` | Ping设备 | ping模块 |
| GET | `/admin/device/sse` | SSE实时推送 | EventEmitter2 |

### 基础数据

| 方法 | 路径 | 说明 |
|------|------|------|
| GET/POST | `/admin/company/*` | 公司管理 |
| GET/POST | `/admin/factory/*` | 工厂管理 |

### 公共功能

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/admin/captcha` | 获取验证码 |
| POST | `/admin/upload` | 文件上传 |

### 安装包管理 `/admin/installation-package`

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| GET | `/admin/installation-package/list` | 安装包列表 | Prisma |
| POST | `/admin/installation-package` | 上传安装包 | FTP + Prisma |
| GET | `/admin/installation-package/download/:id` | 下载安装包 | FTP |

### 打印模板 `/admin/print-template`

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| GET/POST/PUT/DELETE | `/admin/print-template/*` | 打印模板CRUD | Prisma |

---

## System 模块 API

### 认证 `/system/auth`

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/login` | 用户登录(账密+验证码) |
| POST | `/lark/login` | 飞书OAuth登录 |
| GET | `/getInfo` | 获取当前用户信息+权限 |
| GET | `/getRouters` | 获取路由菜单 |
| POST | `/logout` | 注销(Token加入黑名单) |
| POST | `/refreshToken` | 刷新Token |

### 用户管理 `/system/user`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/system/user/list` | 用户列表(分页+筛选) |
| GET | `/system/user/:id` | 用户详情 |
| POST | `/system/user` | 创建用户 |
| PUT | `/system/user` | 更新用户 |
| DELETE | `/system/user/:ids` | 删除用户 |
| PUT | `/system/user/resetPwd` | 重置密码 |
| PUT | `/system/user/changeStatus` | 修改状态 |
| GET | `/system/user/profile` | 个人信息 |
| PUT | `/system/user/profile` | 更新个人信息 |
| POST | `/system/user/profile/avatar` | 上传头像 |

### 角色管理 `/system/role`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET/POST/PUT/DELETE | `/system/role/*` | 角色CRUD |
| PUT | `/system/role/dataScope` | 数据权限 |
| PUT | `/system/role/changeStatus` | 修改角色状态 |

### 菜单管理 `/system/menu`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET/POST/PUT/DELETE | `/system/menu/*` | 菜单CRUD(树形结构) |

### 部门管理 `/system/dept`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET/POST/PUT/DELETE | `/system/dept/*` | 部门CRUD(树形结构) |

### 字典管理 `/system/dict`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET/POST/PUT/DELETE | `/system/dict/type/*` | 字典类型CRUD |
| GET/POST/PUT/DELETE | `/system/dict/data/*` | 字典数据CRUD |

### 日志管理

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/monitor/operlog/list` | 操作日志列表 |
| DELETE | `/monitor/operlog/*` | 删除/清空操作日志 |
| GET | `/monitor/logininfor/list` | 登录日志列表 |
| DELETE | `/monitor/logininfor/*` | 删除/清空登录日志 |

---

## Search 模块 API

| 方法 | 路径 | 说明 | 数据来源 |
|------|------|------|----------|
| POST | `/search/fqc-overdue` | FQC超期预警检查 | n8n + Prisma |

---

## Lark 模块 API

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/lark/approval` | 飞书审批回调 |
| WebSocket | — | 飞书事件订阅 |

---

## n8n 工作流接口汇总

| n8n路径 | 调用方 | 说明 |
|---------|--------|------|
| `/getproductmodel_API` | ProductService | 获取产品模型 |
| `/getproductinfo_API` | ProductService | 搜索产品信息 |
| `/createproduct_API` | ProductService | 创建产品 |
| `/createproductmodel_API` | ProductService | 创建产品模型 |
| `/getprocedure_API` | OperationService | 获取工序列表 |
| `/createprocedure_API` | OperationService | 创建工序 |
| `/updateprocedure_API` | OperationService | 更新工序 |
| `/deleteprocedure_API` | OperationService | 删除工序 |
| `/downloadprocedure_API` | OperationService | 下载工序文件 |
| `/get_fqc` | FQCService | FQC数据查询 |
| `/getPackageInfo` | PackageService | 包装数据查询 |
| `/get_repair_message` | RepairService | 维修数据查询 |
| `/select_common_api` | HostComputerService | 通用上位机查询 |
| `/uuidGetSn` | SearchService/HostComputerService | UUID转SN |

---

## 注意事项

- **内存分页**: 多数查询接口使用内存分页(性能隐患，数据量大时需优化)
- **n8n环境后缀**: n8n接口在非prod环境自动加 `_test` 后缀
- **解绑操作**: 使用 `procedureDetailId=91` 的特殊记录
- **MachineData.data**: 字段是JSON，不同工序存储不同结构的数据
- **FTP存储**: 上位机测试数据文件通过FTP存储在 `/mes/{工序名}/` 目录下
