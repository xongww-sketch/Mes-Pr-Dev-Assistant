# Hermes Agent 部署指南

> 版本：1.0 | 2026-04-14
> 目标：高驰(COROS) MES Agent 部署

---

## 快速部署（5分钟）

### 1. 安装Hermes Agent

```bash
pip3 install hermes-agent
```

### 2. 配置模型API

编辑 `~/.hermes/.env`：

```bash
OPENROUTER_API_KEY=你的API_KEY
OPENROUTER_BASE_URL=https://你的中转站地址/v1
```

**注意**：`OPENROUTER_BASE_URL` 必须包含 `/v1` 后缀！

### 3. 配置飞书Gateway

编辑 `~/.hermes/.env`，添加：

```bash
FEISHU_APP_ID=cli_xxxxx
FEISHU_APP_SECRET=xxxxx
FEISHU_DOMAIN=feishu
FEISHU_CONNECTION_MODE=websocket
GATEWAY_ALLOW_ALL_USERS=true
```

### 4. 配置模型

编辑 `~/.hermes/config.yaml`：

```yaml
model: claude-sonnet-4-6
provider: auto
gateway:
  feishu:
    app_id: cli_xxxxx
    app_secret: xxxxx
    domain: feishu
    connection_mode: websocket
```

### 5. 灌入Skills

```bash
# 复制Skill体系到Hermes
mkdir -p ~/.hermes/skills/mes-agent
cp -r skills/_core/* ~/.hermes/skills/mes-agent/
cp -r skills/_router ~/.hermes/skills/mes-agent/
cp -r skills/bug-triage ~/.hermes/skills/mes-agent/
cp -r skills/n8n-ops ~/.hermes/skills/mes-agent/
cp -r skills/sn-management ~/.hermes/skills/mes-agent/
cp -r skills/requirement-analysis ~/.hermes/skills/mes-agent/
cp -r skills/sample-machine ~/.hermes/skills/mes-agent/
```

### 6. 启动Gateway

```bash
# 导出环境变量
export FEISHU_APP_ID=cli_xxxxx
export FEISHU_APP_SECRET=xxxxx
export FEISHU_DOMAIN=feishu
export FEISHU_CONNECTION_MODE=websocket
export GATEWAY_ALLOW_ALL_USERS=true

# 后台启动
nohup hermes gateway run > /tmp/hermes-gateway.log 2>&1 &

# 验证
tail -f /tmp/hermes-gateway.log
```

看到 `[Lark] connected to wss://msg-frontier.feishu.cn/ws/v2` 说明连接成功。

---

## Docker部署

如果在Docker容器中运行：

```dockerfile
FROM python:3.11-slim

RUN pip3 install hermes-agent lark-oapi

COPY skills/ /root/.hermes/skills/mes-agent/
COPY config.yaml /root/.hermes/config.yaml
COPY .env /root/.hermes/.env

CMD ["hermes", "gateway", "run"]
```

---

## 验证

### 测试模型连接

```bash
hermes chat -q "你好，测试连接" -Q
```

应该返回AI的回复。

### 测试飞书

给飞书Bot发消息，应该能收到回复。

---

## 常见问题

### 1. 模型返回空响应

**原因**：`OPENROUTER_BASE_URL` 缺少 `/v1` 后缀

**解决**：确保URL格式为 `https://你的地址/v1`

### 2. 飞书连接失败

**原因**：缺少 `lark-oapi` 包

**解决**：`pip3 install lark-oapi`

### 3. Gateway无法启动

**原因**：环境变量未导出

**解决**：使用 `export` 导出环境变量后再启动

---

## 文件结构

```
~/.hermes/
├── config.yaml          # 主配置
├── .env                 # 密钥和环境变量
├── skills/
│   └── mes-agent/       # MES Skill体系
│       ├── _core/
│       ├── _router/
│       ├── bug-triage/
│       ├── n8n-ops/
│       ├── sn-management/
│       ├── requirement-analysis/
│       └── sample-machine/
├── state.db             # 会话记忆
└── memories/            # 持久记忆
```

---

*部署完成后，Agent会自动开始学习并改进Skill。*
