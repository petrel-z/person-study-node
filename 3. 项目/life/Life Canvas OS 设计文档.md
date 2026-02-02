## 一、项目概述

### 1.1 项目定位
Life Canvas OS 是一款基于 Electron + Python 的桌面个人成长操作系统，通过八维生命平衡模型帮助用户可视化、管理和优化个人生活的各个方面。

### 1.2 核心理念
- **系统化思维**：将生活抽象为8个可量化的子系统
- **数据驱动**：通过量化评分和趋势分析辅助决策
- **AI赋能**：利用大模型提供个性化洞察和建议
- **极简美学**：借鉴 Notion/Linear 的设计语言，提供沉浸式体验

### 1.3 目标用户
- 追求自我提升和成长的个人用户
- 习惯量化自我（Quantified Self）的极客群体
- 需要系统化管理多维度生活的知识工作者

---

## 二、技术栈选择

### 2.1 前端技术栈
| 技术           | 版本  | 用途                             |
| ------------ | --- | ------------------------------ |
| Electron     | 最新版 | 桌面应用框架，跨平台支持                   |
| React        | 18+ | UI 框架                          |
| TypeScript   | 5+  | 类型安全                           |
| Vite         | 5+  | 构建工具                           |
| shadcn/ui    | 最新  | UI 组件库（基于 Radix UI + Tailwind） |
| TailwindCSS  | 3+  | 原子化 CSS 引擎                     |
| Recharts     | 3+  | 数据可视化（雷达图等）                    |
| Lucide React | 最新  | 图标库                            |
| Zustand      | 最新  | 轻量级状态管理                        |
| React Query  | 最新  | 服务端状态管理                        |

**shadcn/ui 选择理由：**
- 基于 Radix UI，无障碍访问性优秀
- 组件直接复制到项目中，完全可定制
- 与 TailwindCSS 深度集成，样式一致
- 提供 Button、Input、Modal、Toast、Select 等完整组件
- 设计风格极简，符合项目审美要求

### 2.2 后端技术栈
| 技术                 | 版本     | 用途                |
| ------------------ | ------ | ----------------- |
| Python             | 3.11+  | 后端服务              |
| FastAPI            | 0.100+ | 高性能异步 API 框架      |
| SQLAlchemy         | 2.0+   | ORM               |
| Python 标准库 sqlite3 | 内置     | SQLite 驱动（无需额外安装） |
| Alembic            | 最新     | 数据库迁移工具           |
| Pydantic           | 2+     | 数据验证              |

### 2.3 AI 集成
| 服务 | 用途 |
|------|------|
| DeepSeek API | 主力大模型，性价比高 |
| 智谱 GLM API | 备用方案 |
| 百度文心一言 | 备用方案 |
| 通义千问 API | 备用方案 |

### 2.4 数据存储
- **SQLite 3**：嵌入式数据库，存储所有用户数据
  - 无需独立数据库服务
  - 数据存储在用户数据目录 (`~/Library/Application Support/Life Canvas OS/data.db`)
  - 支持事务、外键约束
  - 单用户场景性能完全满足需求

### 2.5 开发工具
- ESLint + Prettier：代码规范
- pytest：Python 测试
- Vitest：前端测试
- electron-builder：应用打包

---

## 三、核心交互设计

### 3.1 信息架构

```
Life Canvas OS
├── 全局画布 (Canvas)
│   ├── 八维雷达图
│   ├── 子系统概览卡片
│   └── AI 摘要简报
├── 神经洞察 (Insights)
│   └── AI 分析报告卡片
├── 时间轴审计 (History)
│   └── 全系统历史日志时间轴
├── 八大子系统 (System Modules)
│   ├── 饮食系统 (FUEL)
│   ├── 运动系统 (PHYSICAL)
│   ├── 读书系统 (INTELLECTUAL)
│   ├── 工作系统 (OUTPUT)
│   ├── 梦想系统 (RECOVERY)
│   ├── 财务系统 (ASSET)
│   ├── 社交系统 (CONNECTION)
│   └── 环境系统 (ENVIRONMENT)
└── 内核配置 (Settings)
    ├── 用户档案
    │   ├── 基本信息（姓名、生日、MBTI 等）
    │   ├── 价值观设置
    │   └── 预期寿命
    ├── AI 配置
    │   ├── API 密钥管理（DeepSeek/智谱/文心/通义）
    │   ├── 模型选择（主力/备用）
    │   ├── 请求参数配置（温度、最大 Token 等）
    │   └── API 连通性测试
    └── 数据管理
        ├── 数据导出（JSON/CSV）
        ├── 数据导入
        └── 数据清除（重置应用）
```

**AI 配置说明：**
- 允许用户配置自己的大模型 API 密钥
- 支持多个大模型提供商，设置优先级
- 主力模型不可用时自动切换到备用模型
- 提供 API 测试功能，验证配置是否正确

### 3.2 交互流程

#### 3.2.1 启动流程
```
用户启动应用
    ↓
Electron 主进程启动
    ↓
启动 Python 后端进程（子进程）
    ↓
Python 初始化 SQLite 数据库（首次运行）
    ↓
建立 IPC 通信通道
    ↓
检查本地缓存登录状态
    ↓
[未登录] → 显示认证界面 → 输入信息 → 登录成功
    ↓
[已登录] → 验证会话有效性
    ↓
加载用户配置和系统状态
    ↓
显示全局画布
```

#### 3.2.2 核心操作流

**评分更新流**
```
用户点击子系统卡片
    ↓
进入子系统详情页
    ↓
点击 +/- 按钮调整评分
    ↓
前端通过 IPC 调用 Python 后端
    ↓
Python 更新 SQLite 数据库
    ↓
返回更新后的数据
    ↓
前端实时更新 UI 和雷达图动画
```

**AI 洞察流**
```
用户点击「启动 AI 洞察」
    ↓
收集当前八维评分数据
    ↓
前端通过 IPC → Python 后端 → 大模型 API
    ↓
等待状态（加载动画）
    ↓
Python 解析大模型响应 → 验证数据
    ↓
存储到 SQLite insights 表
    ↓
通过 IPC 返回结构化数据
    ↓
前端渲染洞察卡片（庆祝/警告/行动）
```

**历史记录流**
```
用户在子系统页添加日志
    ↓
填写标签 + 描述 + 时间
    ↓
前端通过 IPC 提交到 Python 后端
    ↓
Python 存入 SQLite logs 表
    ↓
返回新日志数据
    ↓
前端实时更新历史列表
    ↓
时间轴审计页同步更新
```

### 3.3 关键交互细节

#### 3.3.1 雷达图交互
- **悬停**：显示具体维度名称和数值
- **点击**：跳转至对应子系统详情页
- **动画**：数据变化时平滑过渡（1s duration）

#### 3.3.2 子系统卡片
- **快速调整**：悬停时显示 +/- 按钮，点击直接调整评分
- **查看详情**：点击卡片主体进入详情页
- **状态指示**：评分颜色编码（>80 绿色，50-80 橙色，<50 红色）

#### 3.3.3 AI 洞察卡片
- **类型区分**：
  - 庆祝型（celebration）：绿色，Trophy 图标
  - 警告型（warning）：红色，AlertCircle 图标
  - 行动型（action）：蓝色，Zap 图标
- **交互**：点击可展开/折叠详细建议

#### 3.3.4 响应式布局
- 侧边栏：固定 320px 宽度
- 主内容区：自适应，最大宽度 1400px
- 网格系统：基于 Tailwind 的 12 列网格

---

## 四、模块划分

### 4.1 整体架构（安全方案）

```
┌────────────────────────────────────────────────────────────────┐
│                     Electron Main Process                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Python Backend (FastAPI)                    │  │
│  │  ┌────────────┐  ┌────────────┐  ┌──────────────────┐   │  │
│  │  │   API      │  │    AI      │  │   SQLite DB      │   │  │
│  │  │  Router    │  │  Service   │  │   (本地文件)      │   │  │
│  │  └────────────┘  └────────────┘  └──────────────────┘   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                          ↕ IPC (进程间通信)                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │            IPC Bridge (python-bridge)                    │  │
│  │   将 FastAPI 调用转换为 IPC 消息，不暴露 HTTP 端口         │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
                           ↕ IPC (安全通道)
┌────────────────────────────────────────────────────────────────┐
│                    Electron Renderer                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              React Frontend                               │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────────────┐  │  │
│  │  │  Views  │  │Components│  │   IPC Client Layer      │  │  │
│  │  └─────────┘  └─────────┘  └─────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

**安全架构说明：**
- Python 后端与 Electron 通过 **IPC 通信**，不启动 HTTP 服务器
- 消除端口冲突风险
- 消除 CSRF 攻击面（恶意网页无法访问 IPC 通道）
- 进程间通信通过 Electron 的 `ipcMain` 和 `ipcRenderer` 进行
- Python 后端通过 **stdin/stdout** 与 Electron 主进程通信

### 4.2 前端模块 (Electron + React)

#### 4.2.1 页面模块 (`/src/pages`)
```
pages/
├── CanvasPage.tsx          # 全局画布主页
├── InsightsPage.tsx        # AI 洞察页
├── HistoryPage.tsx         # 时间轴审计页
├── SystemDetailPage.tsx    # 子系统详情页（通用）
└── SettingsPage.tsx        # 设置页
```

#### 4.2.2 组件模块 (`/src/components`)
```
components/
├── ui/                     # shadcn/ui 组件
│   ├── button.tsx
│   ├── input.tsx
│   ├── select.tsx
│   ├── dialog.tsx          # 模态框
│   ├── toast.tsx           # 通知提示
│   └── ...
├── layout/
│   ├── Sidebar.tsx         # 侧边导航栏
│   └── Header.tsx          # 顶部状态栏
├── canvas/
│   ├── RadarChart.tsx      # 雷达图组件
│   ├── SystemCard.tsx      # 系统概览卡片
│   └── AIBriefing.tsx      # AI 摘要简报
├── system/
│   ├── DietTracker.tsx     # 饮食追踪器
│   ├── PhysicalTracker.tsx # 运动追踪器
│   ├── IntellectualSystem.tsx  # 读书系统
│   ├── OutputSystem.tsx    # 工作产出系统
│   ├── RecoveryHub.tsx     # 梦想清单
│   ├── FinanceDashboard.tsx    # 财务仪表盘
│   ├── SocialEnergy.tsx    # 社交能量
│   └── EnvironmentManager.tsx  # 环境管理
├── insights/
│   └── InsightCard.tsx     # 洞察卡片
├── history/
│   └── TimelineLog.tsx     # 时间轴日志
└── auth/
    └── AuthForm.tsx        # 认证表单
```

#### 4.2.3 状态管理 (`/src/store`)
```
store/
├── index.ts                # Store 配置
├── slices/
│   ├── userSlice.ts        # 用户状态
│   ├── systemSlice.ts      # 系统状态
│   ├── insightSlice.ts     # AI 洞察状态
│   └── uiSlice.ts          # UI 状态
```

#### 4.2.4 Hooks (`/src/hooks`) - 与 store 同级
```
hooks/
├── useAppDispatch.ts       # 类型化 dispatch hook
├── useAppSelector.ts       # 类型化 selector hook
├── useIPC.ts               # IPC 通信封装
└── useSystemData.ts        # 系统数据获取 hook
```

#### 4.2.5 IPC 层 (`/src/ipc`)
```
ipc/
├── client.ts               # IPC 客户端封装
├── channels.ts             # 通道名称常量
└── types.ts                # IPC 消息类型定义
```

### 4.3 后端模块 (Python + FastAPI)

#### 4.3.1 项目结构
```
backend/
├── main.py                 # FastAPI 应用入口（IPC 模式）
├── config.py               # 配置管理
├── requirements.txt        # 依赖列表
├── api/
│   ├── __init__.py
│   ├── auth.py             # 认证相关 API
│   ├── system.py           # 系统数据 API
│   ├── insight.py          # AI 洞察 API
│   ├── log.py              # 日志 API
│   └── user.py             # 用户配置 API
├── core/
│   ├── __init__.py
│   ├── security.py         # 安全相关（密码哈希）
│   ├── database.py         # SQLite 连接配置
│   └── config.py           # 配置类定义
├── models/
│   ├── __init__.py
│   ├── user.py             # 用户数据模型
│   ├── system.py           # 系统数据模型
│   └── log.py              # 日志数据模型
├── schemas/
│   ├── __init__.py
│   ├── user.py             # 用户 Pydantic Schema
│   ├── system.py           # 系统 Pydantic Schema
│   ├── insight.py          # 洞察 Pydantic Schema
│   └── log.py              # 日志 Pydantic Schema
├── services/
│   ├── __init__.py
│   ├── ai_service.py       # AI 服务封装
│   ├── insight_service.py  # 洞察生成逻辑
│   └── system_service.py   # 系统数据处理
└── db/
    ├── __init__.py
    ├── session.py          # 数据库会话管理
    └── migrations/         # Alembic 迁移文件
```

#### 4.3.2 IPC 通信协议

**请求格式（Electron → Python）：**
```json
{
  "id": "unique-request-id",
  "method": "POST",
  "path": "/api/systems/abc123/score",
  "body": { "score": 85 }
}
```

**响应格式（Python → Electron）：**
```json
{
  "id": "unique-request-id",
  "status": 200,
  "body": { "success": true, "new_score": 85 }
}
```

#### 4.3.3 API 端点设计（通过 IPC 调用）

**认证模块**
| 方法 | 端点 | 描述 |
|------|------|------|
| POST | /api/auth/login | 用户登录/注册 |
| POST | /api/auth/logout | 用户登出 |
| GET | /api/auth/me | 获取当前用户信息 |

**系统数据模块**
| 方法 | 端点 | 描述 |
|------|------|------|
| GET | /api/systems | 获取所有系统状态 |
| GET | /api/systems/{system_id} | 获取单个系统详情 |
| PATCH | /api/systems/{system_id}/score | 更新系统评分 |
| PATCH | /api/systems/{system_id}/stats | 更新系统统计数据 |
| GET | /api/systems/{system_id}/logs | 获取系统日志 |

**AI 洞察模块**
| 方法 | 端点 | 描述 |
|------|------|------|
| POST | /api/insights/generate | 生成 AI 洞察报告 |
| GET | /api/insights/history | 获取历史洞察记录 |

**日志模块**
| 方法 | 端点 | 描述 |
|------|------|------|
| POST | /api/logs | 添加新日志 |
| GET | /api/logs | 获取日志列表（支持分页、筛选） |
| DELETE | /api/logs/{log_id} | 删除日志 |

**用户配置模块**
| 方法 | 端点 | 描述 |
|------|------|------|
| GET | /api/user/config | 获取用户配置 |
| PATCH | /api/user/config | 更新用户配置（含 AI API 配置） |

### 4.4 数据库设计（SQLite）

#### 4.4.1 数据库位置
```
macOS:  ~/Library/Application Support/Life Canvas OS/data.db
Windows: %APPDATA%/Life Canvas OS/data.db
Linux:  ~/.config/Life Canvas OS/data.db
```

#### 4.4.2 ER 图

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│    users     │         │   systems    │        │     logs     │
├──────────────┤         ├──────────────┤         ├──────────────┤
│ id (PK)      │    ┌───▶│ id (PK)      │         │ id (PK)      │
│ email        │    │    │ user_id (FK) │◀────────│ system_id(FK)│
│ display_name │    │    │ type         │         │ label        │
│ password_hash│    │    │ score        │         │ value        │
│ birthday     │    │    │ stats (JSON) │         │ timestamp    │
│ mbti         │    │    │ created_at   │         │ metadata     │
│ values       │    │    │ updated_at   │         │ created_at   │
│ life_expectancy│   │    └──────────────┘         └──────────────┘
│ created_at   │    │
│ updated_at   │    │
└──────────────┘    │
                    │
         ┌──────────┴──────────┐
         │                     │
┌──────────────┐      ┌──────────────┐
│  insights    │      │action_items  │
├──────────────┤      ├──────────────┤
│ id (PK)      │      │ id (PK)      │
│ user_id (FK) │      │ system_id(FK)│
│ content (JSON)│     │ text         │
│ system_scores│      │ completed    │
│ generated_at │      │ created_at   │
│ created_at   │      └──────────────┘
└──────────────┘
```

#### 4.4.3 表结构定义（SQLite DDL）

**users 表**
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    display_name TEXT NOT NULL,
    password_hash TEXT NOT NULL,
    birthday DATE,
    mbti TEXT,
    values JSON,  -- 存储价值观数组
    life_expectancy INTEGER DEFAULT 85,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
```

**systems 表**
```sql
CREATE TABLE systems (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    type TEXT NOT NULL CHECK(type IN ('FUEL', 'PHYSICAL', 'INTELLECTUAL',
                                      'OUTPUT', 'RECOVERY', 'ASSET',
                                      'CONNECTION', 'ENVIRONMENT')),
    score INTEGER DEFAULT 50 CHECK(score BETWEEN 0 AND 100),
    stats JSON,  -- 存储各系统特定数据
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, type),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_systems_user_type ON systems(user_id, type);
```

**logs 表**
```sql
CREATE TABLE logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    system_id INTEGER NOT NULL,
    label TEXT NOT NULL,
    value TEXT,
    metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (system_id) REFERENCES systems(id) ON DELETE CASCADE
);

CREATE INDEX idx_logs_system_created ON logs(system_id, created_at DESC);
```

**action_items 表**
```sql
CREATE TABLE action_items (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    system_id INTEGER NOT NULL,
    text TEXT NOT NULL,
    completed INTEGER DEFAULT 0,  -- SQLite 使用 0/1 表示布尔值
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (system_id) REFERENCES systems(id) ON DELETE CASCADE
);

CREATE INDEX idx_action_items_system_completed ON action_items(system_id, completed);
```

**insights 表**
```sql
CREATE TABLE insights (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    content JSON NOT NULL,  -- 存储 insights 数组
    system_scores JSON,     -- 生成时的系统快照
    generated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_insights_user_generated ON insights(user_id, generated_at DESC);
```

**user_ai_config 表（新增）**
```sql
CREATE TABLE user_ai_config (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL UNIQUE,
    provider TEXT NOT NULL DEFAULT 'deepseek',  -- deepseek, zhipu, wenxin, qianwen
    api_key TEXT NOT NULL,
    api_endpoint TEXT,  -- 自定义端点（可选）
    model_name TEXT,    -- 具体模型名称
    is_active INTEGER DEFAULT 1,
    priority INTEGER DEFAULT 0,  -- 优先级，数字越大优先级越高
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

#### 4.4.4 SQLite 配置要点

**Python SQLAlchemy 配置：**
```python
# backend/core/database.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
import os
from pathlib import Path

# 获取用户数据目录
if os.name == 'nt':  # Windows
    DATA_DIR = Path(os.environ['APPDATA']) / 'Life Canvas OS'
elif os.name == 'posix':  # macOS / Linux
    if sys.platform == 'darwin':
        DATA_DIR = Path.home() / 'Library' / 'Application Support' / 'Life Canvas OS'
    else:
        DATA_DIR = Path.home() / '.config' / 'Life Canvas OS'

DATA_DIR.mkdir(parents=True, exist_ok=True)
DB_PATH = DATA_DIR / 'data.db'

# SQLite 连接字符串
SQLALCHEMY_DATABASE_URL = f"sqlite:///{DB_PATH}"

# 创建引擎
engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    connect_args={"check_same_thread": False},  # SQLite 多线程必需
    echo=False  # 生产环境设为 False
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
```

### 4.5 Electron 主进程模块

```
main/
├── index.ts                # 主进程入口
├── window.ts               # 窗口管理
├── ipc/
│   ├── index.ts            # IPC 处理器注册
│   ├── handlers/
│   │   ├── system.ts       # 系统相关 IPC
│   │   └── database.ts     # 数据库操作 IPC
│   └── channels.ts         # 通道名称常量
├── python/
│   ├── manager.ts          # Python 后端进程管理
│   ├── bridge.ts           # IPC 与 Python 通信桥接
│   └── monitor.ts          # 后端健康监控
└── menu.ts                 # 应用菜单配置
```

#### 4.5.1 Python 进程启动与通信

**启动 Python 后端：**
```typescript
// main/python/manager.ts
import { spawn, ChildProcess } from 'child_process';
import path from 'path';

export class PythonManager {
  private process: ChildProcess | null = null;

  start() {
    const pythonPath = path.join(
      process.resourcesPath,
      'python-runtime',
      'backend'
    );

    this.process = spawn(pythonPath, [], {
      stdio: ['pipe', 'pipe', 'pipe'],
      env: { ...process.env, PYTHONUNBUFFERED: '1' }
    });

    // 监听 Python 输出（用于 IPC 通信）
    this.process.stdout?.on('data', (data) => {
      // 解析 Python 发送的 JSON 响应
      const responses = data.toString().split('\n').filter(Boolean);
      responses.forEach(line => {
        try {
          const response = JSON.parse(line);
          this.handleResponse(response);
        } catch (e) {
          console.error('Failed to parse Python response:', e);
        }
      });
    });

    this.process.stderr?.on('data', (data) => {
      console.error('Python Error:', data.toString());
    });
  }

  sendRequest(request: any) {
    if (this.process?.stdin) {
      this.process.stdin.write(JSON.stringify(request) + '\n');
    }
  }

  stop() {
    this.process?.kill();
  }
}
```

**Python 端 IPC 处理：**
```python
# backend/main.py
import sys
import json
from fastapi import FastAPI
from fastapi.routing import APIRoute

app = FastAPI()

# IPC 通信循环
def ipc_loop():
    """监听 stdin，处理来自 Electron 的请求"""
    for line in sys.stdin:
        try:
            request = json.loads(line.strip())
            response = handle_request(request)
            sys.stdout.write(json.dumps(response) + '\n')
            sys.stdout.flush()
        except Exception as e:
            error_response = {
                "id": request.get("id"),
                "status": 500,
                "body": {"error": str(e)}
            }
            sys.stdout.write(json.dumps(error_response) + '\n')
            sys.stdout.flush()

def handle_request(request):
    """将请求路由到 FastAPI 处理器"""
    # 模拟 HTTP 请求到 FastAPI
    method = request['method']
    path = request['path']
    body = request.get('body', {})

    # 查找匹配的路由
    for route in app.routes:
        if isinstance(route, APIRoute) and route.path == path and method in route.methods:
            # 调用路由处理函数
            return route.func(**body)

    return {"id": request['id'], "status": 404, "body": {"error": "Not found"}}

if __name__ == "__main__":
    # 在单独线程中启动 IPC 循环
    import threading
    ipc_thread = threading.Thread(target=ipc_loop, daemon=True)
    ipc_thread.start()

    # 保持主线程运行
    import time
    while True:
        time.sleep(1)
```

---

## 五、数据流设计

### 5.1 前端 → 后端通信（IPC 方案）

**IPC 请求流**
```
React Component
    ↓
调用 IPC Client 层
    ↓
ipcRenderer.send('python-request', { id, method, path, body })
    ↓
Electron Main Process 接收
    ↓
通过 stdin 发送给 Python 进程
    ↓
Python 解析 JSON → 路由到 FastAPI 处理器
    ↓
Pydantic Schema 验证
    ↓
业务逻辑处理（Service 层）
    ↓
SQLite 数据库操作
    ↓
Python 通过 stdout 返回 JSON 响应
    ↓
Electron 接收 → 通过 IPC 返回给渲染进程
    ↓
前端更新状态
```

### 5.2 AI 洞察生成流

```
用户点击「启动 AI 洞察」
    ↓
前端通过 IPC 调用 POST /api/insights/generate
    ↓
Python 后端收集当前用户系统评分
    ↓
从 user_ai_config 表获取用户配置的 API 密钥
    ↓
调用 AI Service（按优先级选择可用模型）
    ↓
构建 Prompt → 发送请求到大模型 API
    ↓
解析 JSON 响应 → 数据验证
    ↓
存储到 SQLite insights 表
    ↓
通过 IPC 返回结构化数据
    ↓
前端渲染洞察卡片
```

---

## 六、安全设计

### 6.1 IPC 通信安全
- **进程隔离**：Python 后端与渲染进程完全隔离
- **消息验证**：所有 IPC 消息携带唯一 ID，防止重放攻击
- **权限控制**：仅允许预定义的 IPC 通道通信
- **无 HTTP 暴露**：不监听任何网络端口，彻底消除 CSRF 风险

### 6.2 认证安全
- 本地应用采用简单会话机制
- 密码使用 bcrypt 哈希存储
- 会话存储在 localStorage，有效期无限（本地应用）
- 支持用户登出清除会话

### 6.3 数据安全
- SQLite 数据库文件存储在用户数据目录
- API Key 加密存储（使用系统 Keychain：macOS Keychain / Windows Credential Manager）
- 敏感数据（密码、API Key）不记录日志

### 6.4 Electron 安全
```typescript
// main/index.ts
const mainWindow = new BrowserWindow({
  webPreferences: {
    nodeIntegration: false,       // 禁用 Node 集成
    contextIsolation: true,       // 启用上下文隔离
    sandbox: true,                // 启用沙箱
    preload: path.join(__dirname, 'preload.js')
  }
});

// preload.js - 暴露安全的 IPC API
contextBridge.exposeInMainWorld('electronAPI', {
  invoke: (channel: string, ...args: any[]) => {
    // 仅允许白名单通道
    const validChannels = ['python-request', 'db-query'];
    if (validChannels.includes(channel)) {
      return ipcRenderer.invoke(channel, ...args);
    }
  }
});
```

---

## 七、部署与打包

### 7.1 开发环境
```bash
# 前端
npm install
npm run dev

# 后端（开发时可独立运行，便于调试）
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
python main.py --dev  # 开发模式，同时启动 HTTP 服务器和 IPC
```

### 7.2 生产打包
```bash
# 1. 使用 PyInstaller 打包 Python 后端
cd backend
pyinstaller --onefile \
  --add-data "api:api" \
  --add-data "core:core" \
  --add-data "models:models" \
  --add-data "schemas:schemas" \
  --add-data "services:services" \
  --add-data "db:db" \
  --hidden-import fastapi \
  --hidden-import sqlalchemy \
  --hidden-import pydantic \
  main.py

# 2. 打包 Electron 应用
npm run build
npm run package

# 产物
dist/
├── Life Canvas OS-dmg-x64.dmg     # macOS 安装包
├── Life Canvas OS-win-x64.exe     # Windows 安装包
└── Life Canvas OS-linux-x64.AppImage  # Linux 安装包

# Python 后端会被嵌入到
# Contents/Resources/python-runtime/backend (macOS)
# resources/python-runtime/backend (Windows/Linux)
```

### 7.3 开发依赖清单
```
# backend/requirements.txt
fastapi==0.104.1
uvicorn==0.24.0
sqlalchemy==2.0.23
alembic==1.12.1
pydantic==2.5.0
pydantic-settings==2.1.0
passlib[bcrypt]==1.7.4
python-jose[cryptography]==3.3.0
openai==1.3.5  # 兼容 DeepSeek API
httpx==0.25.2
```

---

## 八、后续扩展方向

### 8.1 功能扩展
- [ ] 数据导入/导出（CSV/JSON）
- [ ] 插件系统支持自定义子系统
- [ ] 社区分享子系统配置模板
- [ ] 数据可视化报告导出 PDF
- [ ] 移动端适配（Electron Mobile 或独立 App）

### 8.2 AI 增强
- [ ] 对话式 AI 助手集成
- [ ] 自动化目标拆解建议
- [ ] 基于历史数据的趋势预测
- [ ] 个性化习惯养成计划

### 8.3 数据同步
- [ ] 可选的云端备份功能（加密后上传）
- [ ] 多设备数据同步
- [ ] 团队/家庭共享模式

---

## 附录：开发优先级建议

### Phase 1: MVP（最小可行产品）
1. ✅ 基础项目搭建（Electron + React + Python）
2. ✅ SQLite 数据库设计与迁移
3. ✅ IPC 通信机制实现
4. ✅ 用户认证系统
5. ✅ 八大子系统 CRUD
6. ✅ 雷达图可视化
7. ✅ 基础 AI 洞察功能

### Phase 2: 核心功能完善
1. ⏳ 各子系统详细功能实现
2. ⏳ AI API 配置管理
3. ⏳ 历史记录与时间轴
4. ⏳ 用户配置管理
5. ⏳ 数据导入/导出

### Phase 3: 体验优化
1. ⏳ UI/UX 细节打磨
2. ⏳ 性能优化
3. ⏳ 错误处理与日志系统
4. ⏳ 打包与分发

---

*文档版本：v2.0*
*最后更新：2026-02-02*
