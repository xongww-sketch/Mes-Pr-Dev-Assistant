---
name: n8n-ops
version: 1.0
description: >
  n8n运维SOP。覆盖n8n平台日常运维、故障排查、workflow管理、
  性能优化、安全维护。是n8n相关问题的标准处理流程。
category: execution
tags: [n8n, ops, maintenance, troubleshooting, sop]
depends_on:
  - _core/mes-domain-knowledge
  - _core/n8n-process-map
  - _core/team-context
  - _core/decision-framework
  - _router/problem-router
---

# n8n 运维 SOP

> 版本：1.0 | 最近更新：2026-04-14
> 用途：n8n平台日常运维、故障排查、workflow管理

---

## 1. 日常巡检

### 1.1 每日检查清单

```bash
# 1. 检查worker状态
docker ps | grep n8n

# 2. 检查最近错误日志
docker logs n8n-worker --tail 50

# 3. 检查磁盘空间
df -h

# 4. 检查数据库连接
pg_isready -h localhost

# 5. 检查Redis
redis-cli ping
```

### 1.2 每周检查清单

- [ ] 检查非活跃workflow是否需要清理
- [ ] 检查慢查询日志
- [ ] 检查workflow执行失败率
- [ ] 检查Token是否即将过期
- [ ] 备份n8n数据库

---

## 2. 故障排查

### 2.1 Worker全挂（P0）

**症状**：所有workflow不执行，队列堆积

**处理步骤**：
```bash
# 1. 确认worker状态
docker ps | grep n8n

# 2. 查看日志
docker logs n8n-worker --tail 200

# 3. 重启服务
cd /path/to/n8n-v2/works
docker compose up -d

# 4. 确认恢复
docker ps | grep n8n
# 检查web UI中workflow是否恢复执行
```

**预防措施**：
- Docker Compose已配置 `restart: unless-stopped`
- 添加worker健康监控 + 飞书告警
- `resourceOnlineStatus` 添加限流

### 2.2 单个Workflow失败

**处理步骤**：
1. 在n8n UI中查看执行记录
2. 检查失败节点的错误信息
3. 检查数据库连接/外部API连通性
4. 手动重新执行验证
5. 如果持续失败 → 检查节点配置

### 2.3 队列堆积

**处理步骤**：
1. 检查是否有慢workflow阻塞
2. 检查worker负载（CPU/内存）
3. 检查是否有高频workflow（如resourceOnlineStatus）
4. 考虑增加worker数量
5. 考虑调整并发槽容量

### 2.4 数据库连接异常

**处理步骤**：
```bash
# 1. 检查PostgreSQL状态
pg_isready -h localhost

# 2. 检查连接数
psql -c "SELECT count(*) FROM pg_stat_activity;"

# 3. 检查慢查询
psql -c "SELECT query, duration FROM pg_stat_activity WHERE state = 'active' ORDER BY duration DESC LIMIT 10;"

# 4. 重启PostgreSQL（如需要）
sudo systemctl restart postgresql
```

---

## 3. Workflow管理

### 3.1 新增Workflow

**流程**：
1. 说明业务背景
2. 明确负责人
3. 标注生命周期（临时/永久）
4. 添加注释和文档
5. 测试通过后激活

**禁止**：
- 无文档的workflow上线
- 无负责人的workflow
- 绕开MES核心业务的workflow

### 3.2 清理Workflow

**可清理的Workflow**：
- 测试workflow（名称含"测试"/"test"）
- 个人workflow（"My workflow"/"我的工作流"）
- 空workflow（0-1节点）
- 重复版本（保留最新版）

**清理流程**：
1. 确认workflow非活跃
2. 确认无依赖
3. 导出备份
4. 删除
5. 记录清理日志

### 3.3 修改Workflow

**流程**：
1. 评估影响范围
2. 导出当前版本备份
3. 在测试环境验证
4. 生产环境修改
5. 验证修改结果
6. 更新文档

---

## 4. 性能优化

### 4.1 慢Workflow优化

**识别**：
- 执行时间 > 30秒的workflow
- 节点数 > 50的workflow

**优化手段**：
- 拆分大workflow为子流程
- 优化数据库查询（加索引）
- 减少不必要的HTTP请求
- 使用批量操作代替循环

### 4.2 高频Workflow限流

**识别**：
- 调用频率 > 1次/秒的workflow
- 如 `resourceOnlineStatus`

**优化手段**：
- 添加调用间隔
- 改为推送模式（Webhook）
- 添加缓存
- 合并多次调用

---

## 5. 安全维护

### 5.1 Token管理

| Token | Workflow | 刷新频率 | 负责人 |
|-------|---------|---------|--------|
| 飞书Token | updateLarkToken | 定时 | 熊旺 |
| 京东Token | 定时刷新京东Token | 定时 | — |
| 斗拱Sign Token | doucSign定时刷新token | 定时 | — |
| SRM Token | 定时获取 SRM TOKEN | 定时 | — |

### 5.2 权限管理

- 使用堡垒机访问服务器
- 不共享账号密码
- 定期审查workflow权限

### 5.3 备份

- 每日备份n8n数据库
- 每周导出workflow JSON备份
- 备份保留30天

---

## 6. 运维手册（Runbook）

### 6.1 服务器信息

| 项目 | 值 |
|------|-----|
| 服务器 | 10.128.11 |
| n8n路径 | /path/to/n8n-v2/works |
| 启动命令 | docker compose up -d |
| 日志命令 | docker logs n8n-worker --tail 100 |
| 负责人 | 王晓虎 |
| 备份人 | 黄启立 |

### 6.2 紧急联系人

| 角色 | 人员 | 联系方式 |
|------|------|---------|
| 技术负责人 | 王晓虎 | 飞书 |
| 产品经理 | 熊旺 | 飞书 |
| 运维备份 | 黄启立 | 飞书 |

---

## 7. 故障复盘模板

```markdown
## 故障复盘

**故障时间**：
**恢复时间**：
**影响范围**：
**严重程度**：P0/P1/P2/P3

### 根因分析
- 直接原因：
- 根本原因：

### 处理时间线
| 时间 | 操作人 | 动作 |
|------|--------|------|
|      |        |      |

### 暴露问题
1.
2.
3.

### 改进措施
| # | 措施 | 负责人 | 优先级 | 完成时间 |
|---|------|--------|--------|---------|
| 1 |      |        |        |         |

### 经验教训
1.
2.
3.
```

---

*本文档是n8n运维的标准SOP。*
*所有n8n相关问题都应先经过 `_router/problem-router.md` 路由到此SOP。*
