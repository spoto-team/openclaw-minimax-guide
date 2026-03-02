# OpenClaw 局域网访问配置指南

本指南帮助用户配置 OpenClaw Web UI，使其可以通过局域网访问，方便在同一网络内的其他设备上使用 OpenClaw。

## 目录

- [方法一：使用配置向导（推荐）](#方法一使用配置向导推荐)
- [方法二：手动配置](#方法二手动配置)
- [访问 Web UI](#访问-web-ui)
- [安全建议](#安全建议)
- [常见问题](#常见问题)

---

## 方法一：使用配置向导（推荐）

OpenClaw 内置配置向导，可以一键开启局域网访问功能。

### 1.1 运行配置向导

```bash
openclaw onboard
```

### 1.2 向导操作流程

1. 选择 `Enable LAN Access` 或类似选项
2. 向导会自动配置 `bind: lan` 和 `controlUi` 相关设置
3. 自动重启 Gateway

### 1.3 验证配置

```bash
openclaw gateway status
```

---

## 方法二：手动配置

### 2.1 备份配置文件

```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup
```

### 2.2 编辑配置文件

```bash
nano ~/.openclaw/openclaw.json
```

### 2.3 修改 Gateway 配置

将 `bind` 改为 `lan`，并添加完整的 `controlUi` 配置：

```json
{
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "lan",
    "auth": {
      "mode": "token",
      "token": "your_token_here"
    },
    "controlUi": {
      "dangerouslyAllowHostHeaderOriginFallback": true,
      "allowInsecureAuth": true,
      "dangerouslyDisableDeviceAuth": true
    },
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    }
  }
}
```

### 2.4 重启 Gateway

```bash
openclaw gateway restart
```

---

## 访问 Web UI

### 获取局域网 IP

```bash
ip addr show | grep -E "inet " | awk '{print $2}' | cut -d'/' -f1 | grep -v "^127"
```

### 访问地址

```
http://<你的局域网IP>:18789/
```

例如：`http://192.168.0.241:18789/`

### 获取 Token

```bash
grep '"token"' ~/.openclaw/openclaw.json
```

---

## 安全建议

⚠️ **重要提醒**：启用局域网访问后，请注意以下安全风险：

- 仅在受信任的网络中使用（家庭/办公室网络）
- 不要在公共 WiFi 下使用
- 定期检查 Gateway 日志
- 轮换 Token 和 API Key

---

## 常见问题

### Q1: 浏览器显示 "disconnected (1008)" 错误

**解决方案**：确保配置文件中添加了完整的 controlUi 设置：

```json
"controlUi": {
  "dangerouslyAllowHostHeaderOriginFallback": true,
  "allowInsecureAuth": true,
  "dangerouslyDisableDeviceAuth": true
}
```

然后重启 Gateway：

```bash
openclaw gateway restart
```

### Q2: 局域网无法访问

1. 检查防火墙：`sudo ufw allow 18789/tcp`
2. 确认 `bind` 为 `lan`
3. 重启 Gateway

### Q3: Token 无效

1. 确认 Token 复制正确
2. 重启 Gateway 使配置生效

---

## 相关链接

- [OpenClaw 官方文档](https://docs.clawd.bot)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)

---

## 版本

- 更新日期: 2026-03-02
