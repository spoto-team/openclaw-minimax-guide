# OpenClaw 国内 API 配置指南

本指南帮助用户快速将 OpenClaw 配置为使用国内 LLM API 服务，支持 MiniMax 和智谱（GLM）两个主流国内 API。

## 目录

- [方法一：使用配置向导（推荐）](#方法一使用配置向导推荐)
- [方法二：手动配置](#方法二手动配置)
- [验证配置](#验证配置)
- [常见问题](#常见问题)

---

## 方法一：使用配置向导（推荐）

OpenClaw 内置配置向导，可以一键完成国内 API 的配置，无需手动编辑配置文件。

### 1.1 运行配置向导

```bash
openclaw onboard
```

或使用快捷命令：

```bash
openclaw onboard --provider minimax-cn
openclaw onboard --provider zhipu-cn
```

### 1.2 向导操作流程

运行命令后，向导会提示你完成以下步骤：

1. **选择 API 服务商**
   - 选择 `minimax-cn`（MiniMax 国内版）
   - 或选择 `zhipu-cn`（智谱 AI）

2. **输入 API Key**
   - 粘贴从对应平台获取的 API Key
   - MiniMax: https://www.minimaxi.com → 控制台 → API Keys
   - 智谱 AI: https://bigmodel.cn → 控制台 → API Keys

3. **选择模型**
   - MiniMax 推荐: `MiniMax-M2.1`
   - 智谱推荐: `glm-4.7`

4. **确认配置**
   - 向导会自动写入配置文件
   - 自动重启 Gateway

### 1.3 验证配置

配置完成后，验证是否成功：

```bash
openclaw models list
```

**成功输出示例**：

```
Model                                      Input      Ctx      Local Auth  Tags
minimax-cn/MiniMax-M2.1                    text       200k     no    yes   default,configured
```

### 1.4 测试模型

```bash
openclaw agent --message "你好，请介绍一下自己" --session-id test
```

---

## 方法二：手动配置

如果需要自定义配置，可以手动编辑配置文件。

### 2.1 查找配置文件

```bash
~/.openclaw/openclaw.json
```

### 2.2 备份配置

```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup
```

### 2.3 MiniMax 配置示例

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "minimax-cn": {
        "baseUrl": "https://api.minimaxi.com/anthropic",
        "apiKey": "YOUR_MINIMAX_API_KEY",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "MiniMax-M2.1",
            "name": "MiniMax M2.1",
            "reasoning": true,
            "input": ["text"],
            "contextWindow": 204800,
            "maxTokens": 131072
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "minimax-cn/MiniMax-M2.1"
      }
    }
  }
}
```

### 2.4 智谱配置示例

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "zhipu-cn": {
        "baseUrl": "https://open.bigmodel.cn/api/anthropic",
        "apiKey": "YOUR_ZHIPU_API_KEY",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "glm-4.7",
            "name": "GLM-4.7",
            "reasoning": true,
            "input": ["text"],
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "zhipu-cn/glm-4.7"
      }
    }
  }
}
```

### 2.5 重启 Gateway

```bash
openclaw gateway restart
```

---

## 验证配置

### 检查模型列表

```bash
openclaw models list
```

### 检查 Gateway 状态

```bash
openclaw gateway status
```

### 测试 API 连接

```bash
openclaw agent --message "Hello" --session-id test
```

---

## 常见问题

### Q1: 向导找不到怎么办？

确保 OpenClaw 版本是最新的：

```bash
openclaw --version
```

如需更新：

```bash
openclaw update
```

### Q2: API Key 获取地址？

| 服务商 | 地址 |
|--------|------|
| MiniMax | https://www.minimaxi.com → 控制台 → API Keys |
| 智谱 AI | https://bigmodel.cn → 控制台 → API Keys |

### Q3: 配置后模型未显示？

1. 重启 Gateway：`openclaw gateway restart`
2. 检查 API Key 是否正确
3. 查看日志：`tail -f ~/.openclaw/logs/gateway.log`

### Q4: 如何切换模型？

在对话中使用：

```bash
/model minimax-cn/MiniMax-M2.1
# 或
/model zhipu-cn/glm-4.7
```

---

## 支持的模型

| 服务商 | 模型 | 上下文 |
|--------|------|--------|
| MiniMax | MiniMax-M2.1 | 200k |
| 智谱 AI | glm-4.7 | 200k |

---

## 相关链接

- [OpenClaw 官方文档](https://docs.clawd.bot)
- [MiniMax 官网](https://www.minimaxi.com)
- [智谱 AI 官网](https://bigmodel.cn)

---

## 版本

- 更新日期: 2026-03-02
- 推荐方法: 使用 `openclaw onboard` 向导
