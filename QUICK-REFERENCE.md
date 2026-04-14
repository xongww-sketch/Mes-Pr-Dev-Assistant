# MES Agent 快速参考卡

> 打印此页，贴在显示器旁边

---

## 🚨 紧急处理

| 场景 | 动作 |
|------|------|
| Worker崩溃 | 重启Docker: `docker compose restart` |
| 过站失败 | 查station_record表，确认SN状态 |
| 绑定异常 | 查binding_record表，确认关联关系 |
| n8n卡死 | 查execution_data表，找卡住的工作流 |

## 📞 升级路径

| 级别 | 联系谁 | 场景 |
|------|--------|------|
| L1 | Agent自动处理 | 常见Bug/已知问题 |
| L2 | 信息化团队 | 新Bug/需代码修改 |
| L3 | Jackson决策 | 架构变更/资源申请 |

## 🔑 关键系统

| 系统 | 用途 | 位置 |
|------|------|------|
| MES主库 | 生产数据 | PostgreSQL |
| n8n | 自动化 | 10.128.11 |
| 旧MES | 历史数据 | SQL Server |
| 新MES | 未来迁移 | NestJS |

## 📋 常用SQL

```sql
-- 查SN过站
SELECT * FROM station_record WHERE sn = 'XXX' ORDER BY created_at DESC;

-- 查绑定
SELECT * FROM binding_record WHERE sn = 'XXX';

-- 查执行日志
SELECT * FROM execution_data WHERE workflow_id = 'XXX' ORDER BY started_at DESC;
```

## ⚡ 决策树

```
问题进来
  ├─ 是Bug？→ bug-triage SOP
  ├─ 是n8n？→ n8n-ops SOP
  ├─ 是SN？→ sn-management SOP
  ├─ 是需求？→ requirement-analysis SOP
  └─ 是样机？→ sample-machine SOP
```

## 🚫 禁止事项

- ❌ 直接删除生产数据
- ❌ 未经确认修改n8n工作流
- ❌ 跳过测试直接上线
- ❌ 隐瞒问题不升级

---

*保持简洁，保持有效。*
