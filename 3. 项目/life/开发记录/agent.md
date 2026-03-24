提示词：规划一下开发这些功能的思路和步骤，然后给我过目

Agent 模块功能开发规划

  开发思路与步骤

  阶段一：会话历史加载与隔离问题修复

  问题分析：
  - 当前 ChatPanel.tsx 使用单一 messages 状态存储所有消息
  - 切换会话时只更新 sessionId，未加载该会话的历史消息
  - 导致新会话可能显示其他会话的消息

  解决方案：
  1. 使用 Map<string, Message[]> 按会话 ID 存储消息
  2. 切换会话时从 Map 读取或调用 API 加载历史
  3. 发送消息时更新当前会话的消息列表

  修改文件：
  - src/renderer/components/agent/ChatPanel.tsx (主要修改)
  - src/renderer/hooks/useAgentApi.ts (已有 getHistory 方法，无需修改)

  实现步骤：
  1. 将 messages 状态改为 Map<string, Message[]>
  2. 修改 handleSendMessage 只更新当前会话的消息
  3. 在 handleSelectSession 中加载历史消息
  4. 添加会话初始化逻辑

  ---
  阶段二：消息操作功能

  功能拆解：

  1. 编辑消息
    - 在 ChatMessage 组件添加编辑按钮
    - 点击后显示可编辑输入框
    - 保存后更新消息状态
  2. 重新发送失败消息
    - 识别 isError: true 的消息
    - 添加重试按钮
    - 调用 chatStream 重新发送
  3. 复制消息内容
    - 使用 Clipboard API
    - 添加复制成功提示

  修改文件：
  - src/renderer/components/agent/ChatMessage.tsx (添加操作菜单)
  - src/renderer/components/agent/ChatPanel.tsx (添加编辑/重发处理)

  实现步骤：
  1. 为 ChatMessage 添加操作菜单 (编辑/复制/重发)
  2. 在 ChatPanel 添加编辑状态和处理函数
  3. 实现复制功能 (使用 navigator.clipboard.writeText)
4. 

  ---
  阶段三：快捷键支持

  功能拆解：

  5. Ctrl+shift+K 唤起 Agent
    - 全局键盘事件监听
    - 控制悬浮球/面板显示状态
  6. Enter 发送 / Shift+Enter 换行
    - 输入框键盘事件处理
    - 区分单行/多行输入模式
  7. Esc 关闭面板
    - 面板打开时监听 Esc 键
    - 关闭面板并清理状态

  修改文件：
  - src/renderer/components/agent/FloatingBall.tsx (Ctrl+K 监听)
  - src/renderer/components/agent/ChatPanel.tsx (Enter/Esc 处理)

  实现步骤：
  1. 在 FloatingBall 添加全局键盘事件监听器
  2. 在输入组件添加 onKeyDown 处理
  3. 使用 useEffect 清理监听器

  ---
  开发顺序建议

  4. 优先修复会话隔离问题 - 这是核心功能 bug，影响用户体验
  5. 实现会话历史加载 - 完善会话切换功能
  6. 实现消息操作功能 - 增强用户交互
  7. 实现快捷键支持 - 提升操作效率

  ---
  预期代码变更量

  | 文件             | 变更行数 | 说明                             |
  |------------------|----------|----------------------------------|
  | ChatPanel.tsx    | ~150 行  | 会话状态管理 + 消息操作 + 快捷键 |
  | ChatMessage.tsx  | ~80 行   | 操作菜单 UI                      |
  | FloatingBall.tsx | ~40 行   | 快捷键监听                       |
  | 总计             | ~270 行  |                                  |

  ---
  风险点

  8. 会话状态管理 - 改为 Map 存储后需确保所有操作都通过 selectedSessionId 访问正确的会话
  9. 编辑消息同步 - 编辑后可能需要调用后端 API 更新会话历史
  10. 快捷键冲突 - 需避免与其他页面快捷键冲突