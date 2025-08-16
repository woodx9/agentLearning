# 第三章：实时流式智能体

[English Version](./README.md)

## 第三章新增功能

第三章在第二章 ReAct 智能体基础上增加了**实时流式功能**和改进的配置管理：

### 🚀 实时流式输出
- **逐字符显示**：观看 AI 响应的实时生成过程
- **流式工具调用**：工具执行与流式输出无缝配合
- **优雅降级**：流式失败时自动回退到标准模式
- **用户体验提升**：无需等待完整响应

### 🔧 配置外部化
- **环境变量**：API 密钥和设置移至 `.env` 文件
- **安全改进**：源代码中无硬编码凭据
- **灵活部署**：不同环境易于配置

## 用户体验对比

**第二章（标准模式）**：
```
用户：帮我列出文件
[等待 3-5 秒]
AI：我来帮您列出当前目录下的文件...
```

**第三章（流式模式）** ✨：
```
用户：帮我列出文件
AI：我来帮您列出当前目录下的文件...
    [文本实时显示，无需等待]
```

## 技术实现

### 流式 API 客户端
[`APIClient`](src/core/api_client.py) 现在支持双模式：

```python
# 标准模式（向后兼容）
def get_completion(self, request_params) -> Message

# 流式模式（新功能）
def get_completion_stream(self, request_params) -> Generator[str, None, None]
```

### 增强的对话管理器
[`Conversation`](src/core/conversation.py) 流式功能：

```python
def print_streaming_content(self, content_chunk):
    """实时显示 AI 响应片段"""
    
def recursive_message_handling(self, message):
    """增强支持带工具调用的流式处理"""
```

### 配置管理
`.env` 中的环境变量：
```env
OPENAI_API_KEY=your_api_key_here
OPENAI_BASE_URL=https://openrouter.ai/api/v1
OPENAI_MODEL=anthropic/claude-3.5-sonnet
```

## 流式输出优势

- **即时反馈**：用户立即看到进度
- **更佳感知性能**：即使延迟相同也感觉更快
- **功能保持**：第二章所有功能在流式模式下正常工作
- **错误恢复**：自动回退确保可靠性

## 技术细节

### 流处理
```python
for chunk in stream:
    if chunk.choices[0].delta.content:
        content_chunk = chunk.choices[0].delta.content
        yield content_chunk  # 实时显示
```

### 工具调用流式处理
```python
# 在流式过程中增量构建工具调用
if hasattr(chunk.choices[0].delta, 'tool_calls'):
    # 累积工具调用数据
    # 完成时执行
```

## 下一步

→ **第四章**：为长期会话添加智能对话历史管理和成本跟踪。
