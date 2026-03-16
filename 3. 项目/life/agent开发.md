⏺ 现在我来帮你梳理整个 Agent 模块的实现架构：
  
  
  ---
  Agent 模块架构详解

  一、整体架构图

  ┌─────────────────────────────────────────────────────────────────┐
  │                        前端 (React)                              │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
  │  │ FloatingBall │  │  ChatPanel   │  │   ConfirmDialog      │  │
  │  └──────────────┘  └──────────────┘  └──────────────────────┘  │
  │         │                  │                      │             │
  │         └──────────────────┼──────────────────────┘             │
  │                            │                                    │
  │                  ┌─────────▼─────────┐                          │
  │                  │   useAgentApi     │                          │
  │                  │      (Hook)       │                          │
  │                  └─────────┬─────────┘                          │
  └────────────────────────────┼────────────────────────────────────┘
                               │ HTTP / IPC
  ┌────────────────────────────▼────────────────────────────────────┐
  │                    后端 (FastAPI)                                │
  │  ┌──────────────────────────────────────────────────────────┐   │
  │  │                  API Routes (/api/agent)                  │   │
  │  │  - POST /chat          - POST /chat/stream               │   │
  │  │  - POST /confirm       - GET /history                    │   │
  │  │  - GET /sessions       - GET/DELETE /session/{id}        │   │
  │  └──────────────────────────────────────────────────────────┘   │
  │                            │                                    │
  │                  ┌─────────▼─────────┐                          │
  │                  │   init.py         │                          │
  │                  │  - execute_chat   │                          │
  │                  │  - execute_stream │                          │
  │                  └─────────┬─────────┘                          │
  │                            │                                    │
  │  ┌─────────────────────────▼────────────────────────────────┐   │
  │  │                    ReAct Executor                         │   │
  │  │  ┌─────────────────────────────────────────────────────┐  │   │
  │  │  │  1. 安全检查 (Gatekeeper)                            │  │   │
  │  │  │  2. 构建消息列表 (System Prompt + History)           │  │   │
  │  │  │  3. 调用 LLM (带 Function Calling)                   │  │   │
  │  │  │  4. 执行 Skill / 工具                                │  │   │
  │  │  │  5. 返回结果 / 需要确认                              │  │   │
  │  │  └─────────────────────────────────────────────────────┘  │   │
  │  └───────────────────────────────────────────────────────────┘   │
  │                            │                                    │
  │         ┌──────────────────┼──────────────────┐                 │
  │         │                  │                  │                 │
  │  ┌──────▼──────┐   ┌──────▼──────┐   ┌──────▼──────┐           │
  │  │    Skills   │   │  LLM Client │   │  Context    │           │
  │  │  Registry   │   │  (DeepSeek/ │   │  Manager    │           │
  │  │             │   │   Doubao)   │   │             │           │
  │  │ - Journal   │   │             │   │ - Session   │           │
  │  │ - Memory    │   │ - Fallback  │   │   History   │           │
  │  │ - System    │   │ - Streaming │   │ - Token     │           │
  │  └─────────────┘   └─────────────┘   │   Count     │           │
  │                                      └─────────────┘           │
  └─────────────────────────────────────────────────────────────────┘

  ---
  二、核心模块说明

  1. 前端层 (src/renderer/components/agent/)

  | 组件              | 功能                             |
  |-------------------|----------------------------------|
  | FloatingBall.tsx  | 右下角悬浮球，Agent 入口         |
  | ChatPanel.tsx     | 聊天面板，处理用户输入和消息展示 |
  | ChatMessage.tsx   | 消息气泡组件                     |
  | ConfirmDialog.tsx | 风险操作确认对话框               |

  核心 Hook: useAgentApi.ts
  - chat() - 发送聊天请求
  - chatStream() - 流式聊天（SSE）
  - confirm() - 确认操作

  ---
  2. 后端 API 层 (backend/api/agent.py)

  # 5 个核心端点
  POST /api/agent/chat          # 普通聊天
  POST /api/agent/chat/stream   # 流式聊天 (SSE)
  POST /api/agent/confirm       # 确认操作
  GET  /api/agent/history       # 获取会话历史
  GET  /api/agent/sessions      # 获取会话列表

  ---
  3. Agent 入口层 (backend/agent/init.py)

  ┌────────────────────────────────────────────┐
  │              initialize_agent()            │
  │  1. 创建 LLM 客户端 (带故障转移)             │
  │  2. 获取上下文管理器                        │
  │  3. 注册 Skills (14 个)                     │
  │  4. 创建 ReActExecutor                      │
  └────────────────────────────────────────────┘
                │
                ▼
  ┌────────────────────────────────────────────┐
  │            execute_stream_chat()           │
  │  1. 构建消息列表                            │
  │  2. 调用 LLM.chat()                         │
  │  3. 检查工具调用                            │
  │  4. 执行技能                                │
  │  5. 流式返回结果                            │
  └────────────────────────────────────────────┘

  ---
  4. ReAct 执行器 (backend/agent/core/executor.py)

  执行流程:

  用户消息 → ReActExecutor.execute()
                │
                ▼
      ┌───────────────────┐
      │ 1. 安全检查        │ ← Gatekeeper.check()
      └─────────┬─────────┘
                ▼
      ┌───────────────────┐
      │ 2. 构建消息列表    │ ← System Prompt + History
      └─────────┬─────────┘
                ▼
      ┌───────────────────┐
      │ 3. 调用 LLM        │ ← llm.chat(tools=[...])
      └─────────┬─────────┘
                ▼
      ┌───────────────────┐
      │ 4. 分析响应        │
      │ - 有 tool_calls?  │───Yes───→ 执行技能 → 返回结果
      │ - 无 tool_calls?  │───No────→ 直接返回
      └───────────────────┘

  ReAct 循环 (最多 5 次迭代):
  Thought → Action → Observation → Thought → Action → ... → Final Answer

  ---
  5. Skill 系统 (backend/agent/skills/)

  Skill 基类 (base.py):
  class BaseSkill(ABC):
      name: str           # 技能唯一标识
      description: str    # 技能描述 (给 LLM 看)
      parameters: List    # 参数定义
      risk_level: RiskLevel  # 风险等级

      @abstractmethod
      async def execute(self, **kwargs) -> SkillResult:
          pass

  风险等级:
  | 等级     | 说明       | 示例               |
  |----------|------------|--------------------|
  | LOW      | 只读查询   | 查询日记、查询记忆 |
  | MEDIUM   | 创建/更新  | 创建日记、更新日记 |
  | HIGH     | 删除操作   | 删除日记           |
  | CRITICAL | 系统级操作 | 数据导出、重置     |

  已实现的 Skills (共 14 个):
  - 日记 Skills (4 个): CreateJournalSkill, QueryJournalsSkill, UpdateJournalSkill, DeleteJournalSkill
  - 记忆 Skills (4 个): CreateMemorySkill, QueryMemoriesSkill, SummarizeMemoriesSkill, ForgetMessageSkill
  - 系统 Skills (6 个): GetSystemScoreSkill, UpdateSystemScoreSkill, AddSystemLogSkill, AddSystemActionSkill, CompleteSystemActionSkill, ListSystemActionsSkill

  ---
  6. LLM 客户端层 (backend/agent/llm/)

  ┌─────────────────────────────────────────┐
  │       LLMClientWithFallback             │
  │  (故障转移客户端 - 自动切换提供商)       │
  └─────────────────┬───────────────────────┘
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
  ┌───────────────┐   ┌───────────────┐
  │ DeepSeekClient│   │ DoubaoClient  │
  │               │   │               │
  │ - chat()      │   │ - chat()      │
  │ - stream_chat │   │ - stream_chat │
  │ - Function    │   │ - Function    │
  │   Calling     │   │   Calling     │
  └───────────────┘   └───────────────┘

  LLM 基类 (base.py) 定义统一接口:
  class LLMClient(ABC):
      @abstractmethod
      async def chat(messages, tools, temperature, max_tokens) -> LLMResponse:
          pass

      @abstractmethod
      async def stream_chat(messages, tools) -> AsyncIterator[str]:
          pass

  ---
  7. 上下文管理器 (backend/agent/core/context.py)

  管理会话状态:
  class ContextState:
      session_id: str
      messages: List[Dict]      # 消息历史
      last_operations: List     # 最近操作
      token_count: int          # Token 计数
      last_accessed: datetime   # 最后访问时间

  class ContextManager:
      get(session_id) -> ContextState
      get_or_create(session_id) -> ContextState
      delete(session_id)

  ---
  三、数据流示例

  用户: "帮我写一篇日记"

  1. 前端 ChatPanel → chatStream("帮我写一篇日记")
  2. API Request → POST /api/agent/chat/stream
  3. init.execute_stream_chat()
     │
     ├─ 获取上下文 (session_id)
     ├─ 构建消息列表 (System Prompt + History)
     ├─ 调用 LLM.chat(tools=[CreateJournalSkill, ...])
     │
     └─ LLM 响应 (Function Calling):
        {
          "name": "create_journal",
          "arguments": {
            "title": "今日总结",
            "content": "用户想要写一篇日记..."
          }
        }
     │
     ├─ ReActExecutor._execute_tool_calls()
     │  └─ CreateJournalSkill.execute(title="今日总结", content="...")
     │     └─ 调用 backend.services.journal_service.create_diary()
     │        └─ 数据库插入记录
     │
     ├─ 流式返回结果:
     │  data: {"type": "content", "data": "日"}
     │  data: {"type": "content", "data": "记"}
     │  data: {"type": "content", "data": "已"}
     │  data: {"type": "content", "data": "创"}
     │  data: {"type": "content", "data": "建"}
     │  data: {"type": "done", "data": {...}}
     │
  4. 前端累积字符 → 显示 "日记已创建"

  ---
  四、关键技术点

  1. Function Calling

  LLM 返回工具调用格式:
  {
    "role": "assistant",
    "content": null,
    "tool_calls": [{
      "id": "call_123",
      "type": "function",
      "function": {
        "name": "create_journal",
        "arguments": "{\"title\": \"xxx\", \"content\": \"xxx\"}"
      }
    }]
  }

  2. 流式响应 (SSE)

  # 后端
  async def generate():
      async for chunk in execute_stream_chat(...):
          yield f"data: {json.dumps(chunk)}\n\n"
      yield "data: [DONE]\n\n"

  # 前端
  for await (const chunk of chatStream(message)) {
      if (chunk.type === 'content') {
          fullContent += chunk.data
      }
  }

  3. 确认流程

  高风险操作 → SkillResult(requires_confirmation=True)
           → 后端存储到 _pending_confirmations
           → 前端显示 ConfirmDialog
           → 用户确认 → POST /api/agent/confirm
           → 执行实际操作

  ---
  五、模块依赖关系

  ChatPanel.tsx
      │
      ▼
  useAgentApi.ts
      │
      ▼
  api/client.ts (HTTP / IPC)
      │
      ▼
  backend/api/agent.py
      │
      ▼
  backend/agent/init.py
      │
      ├─► ReActExecutor
      │       │
      │       ├─► LLMClientWithFallback
      │       │       │
      │       │       ├─► DeepSeekClient
      │       │       └─► DoubaoClient
      │       │
      │       ├─► SkillRegistry
      │       │       │
      │       │       ├─► JournalSkills
      │       │       ├─► MemorySkills
      │       │       └─► SystemSkills
      │       │
      │       └─► ContextManager
      │
      └─► execute_stream_chat()

  ---
  这就是整个 Agent 模块的完整实现架构。核心是 ReAct 模式 + Function Calling，让 LLM 能够自主决定调用哪个 Skill 来完成用户任务。
