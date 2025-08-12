# QuickStar - History Control Agent

一个基于 ReAct（Reasoning and Acting）模式的 AI 代理系统，支持**实时流式输出**、**智能历史管理**、**成本跟踪**、工具调用和用户交互。

## 🚀 Chapter4 主要更新

### 1. 智能历史管理与压缩

Chapter4 的核心升级是**智能历史管理和压缩**功能：
- 🧠 **自动压缩**：当上下文窗口使用率超过阈值时，自动压缩历史对话
- 📊 **Token 使用监控**：实时显示当前上下文窗口使用率百分比
- 🔄 **多会话压缩**：智能删除最旧的对话会话，保留系统消息和最新对话
- ✂️ **单会话压缩**：在单个会话中删除部分中间的助手和工具响应
- 💾 **历史记录管理**：统一管理对话历史，支持分层存储

### 2. 💰 成本跟踪系统 **[NEW]**

#### 📈 智能成本监控
- **总成本跟踪**：自动累计所有 API 调用的成本
- **实时显示**：在每次交互后显示当前会话的总成本
- **成本可见性**：帮助用户实时了解 API 使用费用
- **精确计算**：基于模型返回的准确成本信息

#### 🎯 使用体验对比



**Chapter4（上下文窗口信息跟随，智能成本跟踪）✨**
```
🤖 (context window: 45.2%, total cost: 0.05$) 已优化上下文，继续对话...
```

### 3. 消息格式增强 **[NEW]**

#### 🔧 支持多模态内容格式
- **统一消息结构**：所有消息内容采用数组格式，支持多种内容类型
- **缓存控制**：为最新消息添加缓存标记，优化性能
- **向前兼容**：保持与现有工具和系统的完全兼容

#### 📝 新消息格式示例
```python
# 旧格式
{"role": "user", "content": "Hello"}

# 新格式（支持多模态和缓存控制）
{
    "role": "user", 
    "content": [
        {"type": "text", "text": "Hello", "cache_control": {"type": "ephemeral"}}
    ]
}
```

### 4. Token 使用量跟踪增强

#### 📊 实时监控功能
- **智能阈值管理**：可配置的压缩触发阈值（默认80%）
- **使用率显示**：每次API调用后显示当前上下文窗口使用百分比和总成本
- **成本控制**：帮助用户了解和控制API使用成本
- **性能优化**：避免因上下文过长导致的响应延迟

#### 🔧 配置选项
```env
# 模型最大token数（单位：k）
MODEL_MAX_TOKENS=200
# 压缩触发阈值（0.8 = 80%）
COMPRESS_THRESHOLD=0.8
```

### 5. 历史管理器架构

#### 🏗️ 新增组件
- **HistoryManager**：核心历史管理类
  - `add_message()` - 添加消息到历史记录
  - `update_token_usage()` - 更新token使用情况
  - `auto_messages_compression()` - 自动执行压缩
  - `get_current_messages()` - 获取当前消息列表
  - `current_context_window` - 获取当前上下文窗口使用率 **[NEW]**

#### 🎨 压缩策略
1. **多会话压缩**：
   - 保留系统消息
   - 删除最旧的完整对话会话
   - 保留最近的对话内容
   - 添加压缩通知消息

2. **单会话压缩**：
   - 保留用户输入和系统消息
   - 删除部分中间的助手/工具响应
   - 保留最新的几条消息
   - 添加压缩说明

### 6. API 客户端增强

#### ⚡ 成本跟踪集成 **[NEW]**
- **成本累计**：自动累计每次 API 调用的成本
- **总成本属性**：通过 `total_cost` 属性获取累计成本
- **流式模式增强**：在流式响应中包含成本统计
- **返回值扩展**：API调用返回消息和token使用情况的元组

```python
# 新的成本跟踪属性
api_client.total_cost  # 获取总成本

# API 返回格式保持不变
message, token_usage = api_client.get_completion(params)
```


ReAct（Reasoning and Acting）架构通过以下核心流程实现智能代理：

1. **思考（Think）**：AI 模型接收输入并进行推理
2. **行动（Act）**：基于推理结果调用相应工具
3. **观察（Observe）**：获取工具执行结果作为反馈
4. **历史管理（Manage）**：智能压缩和管理对话历史 **[Chapter4 新增]**
5. **成本监控（Monitor）**：实时跟踪和显示使用成本 **[Chapter4 新增]**
6. **循环迭代**：将观察结果输入下一轮思考，形成完整的推理-行动循环

这种架构使 AI 代理能够在复杂任务中保持连贯的推理链，并通过工具调用与外部环境交互，同时通过智能历史管理确保长对话的高效处理，通过成本监控帮助用户控制使用费用。

## 核心组件

### 🧠 HistoryManager - 智能历史管理器

[`HistoryManager`](src/core/history/history_manager.py) 是 Chapter4 的核心新组件：

**核心功能**：
```python
class HistoryManager:
    def add_message(self, message) -> None
    def update_token_usage(self, token_usage) -> None  
    def auto_messages_compression(self) -> None
    def get_current_messages(self) -> List[Message]
    
    @property
    def current_context_window(self) -> str  # [NEW] 获取当前窗口使用率
```

**智能压缩逻辑**：
- 监控token使用率，超过阈值自动触发压缩
- 多会话场景：删除最旧的完整对话会话
- 单会话场景：删除部分中间响应，保留关键信息
- 添加压缩通知，确保上下文连贯性

### 🌊 APIClient - 增强版流式API客户端

[`APIClient`](src/core/api_client.py) 现在支持成本跟踪：

**标准模式**（返回消息和使用量）：
```python
def get_completion(self, request_params) -> Tuple[Message, TokenUsage]
```

**🆕 流式模式**（包含成本统计）：
```python
def get_completion_stream(self, request_params) -> Generator[str, None, None]
# 最后yield完整的消息对象，包含token使用量和成本信息
```

**🆕 成本跟踪功能**：
- 自动累计所有 API 调用成本
- 通过 `total_cost` 属性访问总成本
- 支持流式和标准模式的成本统计
- 基于模型返回的准确成本数据

### 💬 Conversation - 历史感知的对话管理器

[`Conversation`](src/core/conversation.py) 集成历史管理和成本显示：

**🆕 历史管理集成**：
- `messages` 属性现在通过 HistoryManager 提供
- `add_message()` 统一通过历史管理器处理
- 自动压缩检查在每次消息处理前后执行
- Token使用量和成本自动更新到历史管理器

**🆕 消息格式升级**：
- 所有消息内容采用数组格式
- 支持多模态内容（文本、图像等）
- 自动添加缓存控制标记
- 保持向前兼容性

**核心流程增强**：
1. 🔄 发送消息到 AI 模型（流式）
2. 📺 实时显示AI回复内容
3. 📊 显示token使用率和总成本 **[NEW]**
4. 🧠 检查是否需要历史压缩
5. 🔍 检查响应是否包含工具调用
6. ☝️ 如需批准，等待用户确认
7. ⚡ 执行工具并将结果反馈给 AI
8. 🔁 递归继续对话

### ToolManager 工具管理器

[`ToolManager`](src/tools/tool_manager.py) 保持不变，完全兼容历史管理：

- **工具注册**：统一管理所有可用工具
- **描述生成**：为 AI 提供工具的 JSON Schema 描述
- **执行代理**：根据工具名称分发执行请求

### 工具系统

所有工具继承自 [`BaseAgent`](src/tools/base_agent.py)，目前实现了：

- **CmdRunner**：执行系统命令，支持超时控制和用户批准

## 快速开始

```bash
# 安装依赖
pip install -e .

# 配置环境变量
cp .env.example .env
# 编辑 .env 文件，填入实际的API配置和历史管理参数

# 运行程序
quickstar

# 测试历史压缩功能
python test/test_history_compress.py
```

## 🔧 技术实现细节

### 成本跟踪算法 **[NEW]**
```python
def get_completion_stream(self, request_params):
    for chunk in stream:
        if hasattr(chunk, 'usage') and chunk.usage:
            cost = getattr(chunk.usage, 'model_extra', {})
            if isinstance(cost, dict):
                self._total_cost += cost.get("cost", 0)
```

### 历史压缩算法
```python
def _compress_current_message(self):
    current_messages = self.messages_history[-1]
    user_indices = self._get_user_message_indices(current_messages)
    
    if len(user_indices) > 1:
        # 多会话：删除最旧的会话
        self._compress_multiple_sessions(current_messages, user_indices)
    elif len(user_indices) == 1:
        # 单会话：删除中间响应
        self._compress_single_session(current_messages, user_indices[0])
```

### Token使用量监控
```python
@property
def current_context_window(self):
    if not self.history_token_usage or self._model_max_tokens == 0:
        return "0.0"
    return f"{100 * self.history_token_usage[-1].total_tokens / self._model_max_tokens:.1f}"
```

### 缓存控制实现 **[NEW]**
```python
def _get_messages_with_cache_mark(self):
    messages = self._history_manager.get_current_messages()
    if messages and "content" in messages[-1] and messages[-1]["content"]:
        messages[-1]["content"][-1]["cache_control"] = {"type": "ephemeral"}
    return messages
```

## 🎯 Chapter4 vs Chapter3

| 特性 | Chapter3 | Chapter4 |
|------|----------|----------|
| 响应模式 | ✅ 实时流式 | ✅ 实时流式 |
| 用户体验 | ✅ 即时反馈 | ✅ 即时反馈 |
| 工具调用 | ✅ 流式支持 | ✅ 流式支持 |
| 错误处理 | ✅ 优雅降级 | ✅ 优雅降级 |
| 历史管理 | ❌ 无限制堆积 | 🆕 智能压缩 |
| Token监控 | ❌ 无追踪 | 🆕 实时显示 |
| 成本跟踪 | ❌ 无感知 | 🆕 总成本显示 |
| 长对话支持 | ❌ 容易超限 | 🆕 自动优化 |
| 成本控制 | ❌ 无感知 | 🆕 使用量可见 |
| 上下文优化 | ❌ 手动重启 | 🆕 自动压缩 |
| 缓存优化 | ❌ 无缓存 | 🆕 智能缓存 |

## 🧪 测试覆盖

Chapter4 包含完整的历史管理测试套件：

```bash
# 运行历史压缩测试
python test/test_history_compress.py
```

测试覆盖：
- ✅ 自动压缩触发条件
- ✅ 多会话压缩逻辑
- ✅ 单会话压缩逻辑
- ✅ Token使用量更新
- ✅ 压缩阈值判断
- ✅ 成本跟踪准确性 **[NEW]**
- ✅ 消息格式转换 **[NEW]**

这个框架的核心思想是让 AI 能够"思考"（通过对话）和"行动"（通过工具调用），并且在执行可能有风险的操作时需要用户确认。Chapter4 的智能历史管理进一步解决了长对话中的上下文管理难题，成本跟踪功能帮助用户实时了解API使用费用，新的消息格式为未来的多模态功能扩展奠定了基础，使AI代理能够在保持对话连贯性的同时，有效控制成本和性能，为构建真正实用的AI助手奠定了坚实基础。
