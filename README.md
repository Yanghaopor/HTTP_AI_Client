# 下载地址
https://wwbjj.lanzn.com/iLbTr3efbayd

# ai_core服务

ai_core模块的HTTP服务封装，提供RESTful API接口，支持指定端口运行和心跳检测机制。

## 功能特性

- **HTTP API接口**：提供RESTful风格的API接口
- **指定端口运行**：支持自定义服务端口和监听地址
- **心跳检测机制**：支持心跳检测，超时自动退出（默认500秒）
- **流式响应**：支持流式聊天响应
- **多AI服务商**：支持OpenAI和阿里云等多种AI服务提供商

## 快速开始

### 运行exe程序

```bash
# 使用默认参数启动（端口8080，超时500秒）
ai_core_service.exe

# 自定义参数启动
ai_core_service.exe --port 9000 --timeout 300

# 指定监听地址
ai_core_service.exe --host 127.0.0.1 --port 8080
```

## 命令行参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--port` | 服务端口号 | 8080 |
| `--host` | 服务监听地址 | 0.0.0.0 |
| `--timeout` | 心跳超时时间（秒） | 500 |

## API接口

### 1. 健康检查

**接口**: `GET /health`

**响应**:
```json
{
  "success": true,
  "status": "ok",
  "message": "服务正常运行"
}
```

### 2. 心跳检测

**接口**: `GET /heartbeat`

**响应**:
```json
{
  "success": true,
  "message": "心跳已更新",
  "elapsed_time": 10.5
}
```

### 3. 配置AI服务

**接口**: `POST /config`

**请求体**:
```json
{
  "provider_type": "openai",
  "api_key": "your-api-key",
  "model": "gpt-3.5-turbo",
  "base_url": "https://api.openai.com/v1",
  "timeout": 120,
  "max_retries": 3
}
```

**响应**:
```json
{
  "success": true,
  "message": "配置成功",
  "provider_type": "openai",
  "model": "gpt-3.5-turbo"
}
```

### 4. 聊天请求（非流式）

**接口**: `POST /chat`

**请求体**:
```json
{
  "messages": [
    {
      "role": "user",
      "content": "你好，请介绍一下你自己。"
    }
  ],
  "generation_config": {
    "temperature": 0.7,
    "top_p": 1.0,
    "max_tokens": 500
  }
}
```

**响应**:
```json
{
  "success": true,
  "content": "你好！我是一个AI助手...",
  "model": "gpt-3.5-turbo",
  "finish_reason": "stop",
  "usage": {
    "prompt_tokens": 20,
    "completion_tokens": 50,
    "total_tokens": 70
  }
}
```

### 5. 聊天请求（流式）

**接口**: `POST /chat/stream`

**请求体**: 与 `/chat` 相同

**响应**: Server-Sent Events (SSE) 格式

```
data: {"content": "你好", "delta": "你好", "finish_reason": null, "model": "gpt-3.5-turbo"}

data: {"content": "你好！", "delta": "！", "finish_reason": null, "model": "gpt-3.5-turbo"}

data: [DONE]
```

## 心跳检测机制

服务内置心跳检测机制，用于监控客户端连接状态：

- **心跳超时时间**: 默认500秒，可通过 `--timeout` 参数自定义
- **心跳接口**: `GET /heartbeat`
- **超时处理**: 超过指定时间未收到心跳，服务自动退出

**使用建议**：
- 客户端应定期调用心跳接口（建议间隔60秒）
- 可使用测试客户端的自动心跳功能：`client.start_heartbeat(interval=60)`

## 使用示例

### cURL示例

```bash
# 健康检查
curl http://127.0.0.1:8080/health

# 心跳检测
curl http://127.0.0.1:8080/heartbeat

# 配置AI服务
curl -X POST http://127.0.0.1:8080/config \
  -H "Content-Type: application/json" \
  -d "{\"provider_type\":\"openai\",\"api_key\":\"your-api-key\",\"model\":\"gpt-3.5-turbo\"}"

# 聊天请求
curl -X POST http://127.0.0.1:8080/chat \
  -H "Content-Type: application/json" \
  -d "{\"messages\":[{\"role\":\"user\",\"content\":\"你好\"}]}"
```

### Python示例

```python
import requests
import time

# 服务地址
BASE_URL = "http://127.0.0.1:8080"

# 健康检查
response = requests.get(f"{BASE_URL}/health")
print(response.json())

# 配置AI服务
config_data = {
    "provider_type": "openai",
    "api_key": "your-api-key",
    "model": "gpt-3.5-turbo"
}
response = requests.post(f"{BASE_URL}/config", json=config_data)
print(response.json())

# 启动心跳（建议60秒间隔）
def keep_heartbeat():
    while True:
        time.sleep(60)
        requests.get(f"{BASE_URL}/heartbeat")

# 聊天请求
chat_data = {
    "messages": [
        {"role": "user", "content": "你好，请介绍一下你自己。"}
    ]
}
response = requests.post(f"{BASE_URL}/chat", json=chat_data)
print(response.json()['content'])
```

## 注意事项

1. **API密钥安全**: 请妥善保管API密钥，不要在代码中硬编码
2. **端口占用**: 启动前确保指定端口未被占用
3. **心跳保持**: 长期运行时需保持心跳，否则服务会自动退出
4. **防火墙**: 如需远程访问，请确保防火墙允许相应端口

## 故障排除

### 服务无法启动
- 检查端口是否被占用
- 查看控制台错误信息
- 尝试使用其他端口

### 心跳超时
- 确保客户端定期调用心跳接口
- 检查网络连接是否正常
- 可适当增加超时时间参数

### API请求失败
- 确保已正确配置AI服务
- 检查API密钥是否有效
- 查看服务端日志获取详细错误信息

