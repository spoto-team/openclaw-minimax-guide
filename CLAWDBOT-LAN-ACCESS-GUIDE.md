# Clawdbot 局域网访问配置指南

本指南帮助用户配置 Clawdbot Web UI，使其可以通过局域网访问，方便在同一网络内的其他设备上使用 Clawdbot。

## 目录

- [一、前置条件](#一前置条件)
- [二、配置步骤](#二配置步骤)
- [三、访问 Web UI](#三访问-web-ui)
- [四、安全建议](#四安全建议)
- [五、常见问题](#五常见问题)

---

## 一、前置条件

### 1.1 检查 Clawdbot 安装状态

```bash
# 检查 clawdbot 是否已安装
which clawdbot

# 查看已安装版本
clawdbot --version
```

### 1.2 检查配置文件位置

配置文件位置：`~/.clawdbot/clawdbot.json`

查看当前配置：

```bash
cat ~/.clawdbot/clawdbot.json
```

### 1.3 获取本机局域网 IP

```bash
# 获取局域网 IP 地址
ip addr show | grep -E "inet " | awk '{print $2}' | cut -d'/' -f1 | grep -v "^127"

# 示例输出：
# 192.168.0.241
```

记录此 IP 地址，后续用于局域网访问。

---

## 二、配置步骤

### 2.1 备份配置文件

在修改之前，强烈建议备份配置文件：

```bash
cp ~/.clawdbot/clawdbot.json ~/.clawdbot/clawdbot.json.backup
```

### 2.2 编辑配置文件

使用文本编辑器打开配置文件：

```bash
nano ~/.clawdbot/clawdbot.json
```

### 2.3 修改 Gateway 配置

找到 `gateway` 配置块，修改以下两项：

1. **修改 bind 值**：将 `loopback` 改为 `lan`
2. **添加 controlUi 配置**：允许不安全 HTTP 访问

**修改前**：

```json
{
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "your_token_here"
    },
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    }
  }
}
```

**修改后**：

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
      "allowInsecureAuth": true
    },
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    }
  }
}
```

### 2.4 重启 Gateway

保存配置后，重启 Gateway 使配置生效：

```bash
clawdbot gateway restart
```

**预期输出**：

```
Restarted systemd service: clawdbot-gateway.service
```

### 2.5 验证 Gateway 状态

```bash
# 检查 Gateway 运行状态
clawdbot gateway status
```

确认 Gateway 正常运行（状态为 running 或 active）。

---

## 三、访问 Web UI

### 3.1 获取访问地址

根据你的局域网 IP，访问地址为：

```
http://<你的局域网IP>:18789/
```

例如：`http://192.168.0.241:18789/`

### 3.2 获取登录 Token

查看配置文件中的 token：

```bash
grep -A2 '"token"' ~/.clawdbot/clawdbot.json
```

或直接在配置文件中查看：

```json
{
  "auth": {
    "mode": "token",
    "token": "7a2eac70cec4e01c3bed75782d9b927cd36f50073202ab55"
  }
}
```

**Token 示例**：

```
7a2eac70cec4e01c3bed75782d9b927cd36f50073202ab55
```

### 3.3 登录 Web UI

1. 打开浏览器，访问 `http://<你的局域网IP>:18789/`
2. 系统会要求输入认证 Token
3. 输入上一步获取的 Token
4. 点击登录，进入 Clawdbot 控制界面

---

## 四、安全建议

### 4.1 风险说明

⚠️ **重要提醒**：启用局域网访问后，同一网络内的任何设备都可以访问 Clawdbot Web UI。请务必注意以下安全风险：

- 未经授权的用户可能访问你的 AI 助手
- API Key 可能被滥用
- 工作区文件可能被访问或修改

### 4.2 安全最佳实践

#### 仅在受信任的网络中使用

- 家庭网络 ✓
- 办公室网络 ✓
- 公共 WiFi ✗

#### 使用强 Token

确保配置文件中的 token 足够复杂：

```json
{
  "auth": {
    "mode": "token",
    "token": "生成一个强随机token"
  }
}
```

生成强 Token：

```bash
# 使用 openssl 生成随机 token
openssl rand -hex 32
```

#### 限制网络访问

如果需要更严格的访问控制，可以使用以下方法：

**方法 1：使用 SSH 隧道**

```bash
# 在远程设备上执行
ssh -L 18789:127.0.0.1:18789 user@your-server-ip
```

然后访问 `http://127.0.0.1:18789/`

**方法 2：使用 Tailscale**

Tailscale 可以提供更安全的远程访问方式，同时保持内网访问的便利性。

### 4.3 定期检查

- 定期检查 Gateway 日志
- 监控异常访问
- 轮换 API Key 和 Token

---

## 五、常见问题

### Q1: 浏览器显示 "disconnected (1008)" 错误

**错误信息**：

```
disconnected (1008): control ui requires HTTPS or localhost (secure context)
```

**解决方案**：

确保已在配置文件中添加 `controlUi.allowInsecureAuth: true`：

```json
{
  "gateway": {
    "controlUi": {
      "allowInsecureAuth": true
    }
  }
}
```

然后重启 Gateway：

```bash
clawdbot gateway restart
```

### Q2: 局域网无法访问

**问题**：其他设备无法访问 Web UI

**解决方案**：

1. 检查防火墙设置：

   ```bash
   # 检查防火墙状态
   sudo ufw status

   # 开放 18789 端口
   sudo ufw allow 18789/tcp
   ```

2. 确认 Gateway 已启动：

   ```bash
   clawdbot gateway status
   ```

3. 确认 bind 配置为 `lan`：

   ```bash
   grep '"bind"' ~/.clawdbot/clawdbot.json
   # 应输出: "bind": "lan"
   ```

### Q3: Token 无效或被拒绝

**问题**：输入 Token 后无法登录

**解决方案**：

1. 检查 Token 是否正确复制（无多余空格或换行）
2. 确认 Gateway 已重启使配置生效
3. 重新获取 Token：

   ```bash
   grep '"token"' ~/.clawdbot/clawdbot.json
   ```

### Q4: 配置文件语法错误

**问题**：Gateway 无法启动，提示配置错误

**解决方案**：

1. 验证 JSON 语法：

   ```bash
   cat ~/.clawdbot/clawdbot.json | jq
   ```

2. 如果输出错误，修复 JSON 语法

3. 使用备份恢复（如有必要）：

   ```bash
   cp ~/.clawdbot/clawdbot.json.backup ~/.clawdbot/clawdbot.json
   ```

4. 重新编辑配置文件

### Q5: 如何切换回本地访问

如果需要禁用局域网访问，恢复仅本地访问：

```json
{
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "your_token_here"
    },
    "controlUi": {
      "allowInsecureAuth": false
    }
  }
}
```

然后重启 Gateway：

```bash
clawdbot gateway restart
```

---

## 附录

### 配置文件参数说明

| 参数 | 说明 | 可选值 |
|------|------|--------|
| `gateway.bind` | 网络绑定模式 | `loopback`(仅本地), `lan`(局域网) |
| `gateway.port` | 服务端口 | 默认 `18789` |
| `controlUi.allowInsecureAuth` | 允许 HTTP 认证 | `true`(允许), `false`(禁止) |
| `auth.mode` | 认证模式 | `token` |
| `auth.token` | 访问令牌 | 随机字符串 |

### 相关链接

- [Clawdbot 官方文档](https://docs.clawd.bot)
- [Clawdbot GitHub](https://github.com/clawdbot/clawdbot)
- [Tailscale 文档](https://tailscale.com/)

---

## 版本信息

- 文档版本: 1.0
- 创建日期: 2026-01-29
- 支持的 Clawdbot 版本: 2026.1.24-3+

**注意**：请妥善保管您的 Token 和 API Key，定期检查访问日志以确保安全。
