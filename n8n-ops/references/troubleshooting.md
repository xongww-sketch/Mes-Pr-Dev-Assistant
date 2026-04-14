# n8n 故障排查手册

> 版本：1.0 | 最近更新：2026-04-14
> 用途：n8n故障排查的标准流程

---

## 1. 快速诊断流程

```
收到故障报告
  │
  ▼
1. 确认影响范围
   - 全部workflow？→ Worker/数据库问题
   - 单个workflow？→ 配置/连接问题
   - 部分workflow？→ 外部系统问题
  │
  ▼
2. 检查基础设施
   - docker ps | grep n8n
   - df -h
   - pg_isready
   - redis-cli ping
  │
  ▼
3. 检查日志
   - docker logs n8n-worker --tail 100
   - PostgreSQL慢查询日志
  │
  ▼
4. 定位问题
   - Worker挂了？→ 重启
   - 数据库问题？→ 修复
   - 外部系统？→ 检查连通性
  │
  ▼
5. 执行修复
  │
  ▼
6. 验证结果
```

---

## 2. 常见故障处理

### 2.1 Worker全挂

```bash
# 诊断
docker ps | grep n8n
docker logs n8n-worker --tail 200

# 修复
cd /path/to/n8n-v2/works
docker compose up -d

# 验证
docker ps | grep n8n
# 检查web UI
```

### 2.2 数据库连接失败

```bash
# 诊断
pg_isready -h localhost
psql -c "SELECT 1;"

# 修复
sudo systemctl restart postgresql

# 验证
pg_isready -h localhost
```

### 2.3 磁盘空间不足

```bash
# 诊断
df -h

# 修复
# 清理日志
sudo journalctl --vacuum-time=7d
# 清理docker日志
sudo find /var/lib/docker/containers -name "*.log" -exec truncate -s 0 {} \;

# 验证
df -h
```

### 2.4 Redis连接失败

```bash
# 诊断
redis-cli ping

# 修复
sudo systemctl restart redis

# 验证
redis-cli ping
```

---

## 3. 性能问题排查

### 3.1 慢Workflow

```sql
-- 查慢查询
SELECT query, duration 
FROM pg_stat_activity 
WHERE state = 'active' 
ORDER BY duration DESC 
LIMIT 10;
```

### 3.2 队列堆积

- 检查worker负载
- 检查高频workflow
- 考虑增加worker数量

---

## 4. 复盘模板

见 `n8n-ops/SKILL.md` 中的故障复盘模板。

---

*持续更新，每次故障后补充。*
