# 通用web项目规范总结

---

## 📋 目录

1. [项目结构规范](#1-项目结构规范)
2. [代码规范](#2-代码规范)
3. [命名规范](#3-命名规范)
4. [TypeScript 规范](#4-typescript-规范)
5. [组件开发规范](#5-组件开发规范)
6. [API 请求规范](#6-api-请求规范)
7. [状态管理规范](#7-状态管理规范)
8. [路由规范](#8-路由规范)
9. [样式规范](#9-样式规范)
10. [代码审查规范](#10-代码审查规范)
11. [测试规范](#11-测试规范)
12. [配置文件规范](#12-配置文件规范)
13. [开发流程规范](#13-开发流程规范)
14. [文档规范](#14-文档规范)

---

## 1. 项目结构规范

### 1.1 标准目录结构

```
project-name/
├── public/                  # 静态资源
│   └── favicon.ico
├── src/
│   ├── assets/             # 资源文件
│   │   ├── images/         # 图片
│   │   └── styles/         # 全局样式
│   ├── components/         # 组件
│   │   ├── common/         # 通用组件(Button、Input等)
│   │   ├── layout/         # 布局组件(Header、Footer等)
│   │   └── business/       # 业务组件
│   ├── composables/        # 组合式函数/自定义Hooks
│   ├── constants/          # 常量定义
│   ├── router/             # 路由配置
│   ├── services/           # API服务层
│   ├── stores/             # 状态管理
│   ├── types/              # TypeScript类型定义
│   ├── utils/              # 工具函数
│   ├── views/              # 页面组件
│   ├── App.vue             # 根组件
│   └── main.ts             # 应用入口
├── docs/                   # 项目文档
├── .env.development        # 开发环境变量
├── .env.production         # 生产环境变量
├── .eslintrc.cjs           # ESLint配置
├── .prettierrc             # Prettier配置
├── package.json
├── tsconfig.json           # TypeScript配置
├── tailwind.config.js      # Tailwind配置
└── vite.config.ts          # Vite配置
```

### 1.2 分层架构原则

```
┌─────────────────────────────────────┐
│         Views (视图层)                │
│    - 只负责 UI 渲染和用户交互          │
│    - 不包含业务逻辑                   │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│      Composables/Stores (业务层)      │
│    - 处理业务逻辑                     │
│    - 状态管理                         │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│       Services (服务层)               │
│    - 纯 API 调用                      │
│    - 不处理业务逻辑                   │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│      Utils/Types (基础层)             │
│    - 工具函数                         │
│    - 类型定义                         │
└─────────────────────────────────────┘
```

**核心原则**:
- ✅ 视图层只负责 UI 渲染
- ✅ 业务逻辑在 Composables/Store 中
- ✅ API 调用统一在 Service 层
- ✅ 类型定义在 types 目录
- ✅ 工具函数在 utils 目录

---

## 2. 代码规范

### 2.1 核心原则

#### 禁止事项

1. **禁止使用 `any` 类型**
   ```typescript
   // ❌ 错误
   const data: any = await api.getUser();

   // ✅ 正确
   const data: User = await api.getUser();
   ```

2. **禁止忽略类型错误**
   - 使用 `// @ts-ignore` 必须说明原因

3. **禁止在组件中直接调用 API**
   ```typescript
   // ❌ 错误
   import axios from "axios";
   const data = await axios.get("/api/user");

   // ✅ 正确
   import { userService } from "@/services/user.service";
   const data = await userService.getCurrentUser();
   ```

4. **禁止硬编码配置**
   ```typescript
   // ❌ 错误
   const apiUrl = "http://localhost:8080";

   // ✅ 正确
   const apiUrl = import.meta.env.VITE_API_BASE_URL;
   ```

#### 必须遵守

1. **所有异步操作必须包含 try-catch**
   ```typescript
   async function fetchData() {
     try {
       const data = await api.getData();
       return data;
     } catch (error) {
       console.error("获取数据失败:", error);
       throw error;
     }
   }
   ```

2. **组件 Props 必须定义类型**
   ```typescript
   interface Props {
     title: string;
     count?: number;
   }
   const props = withDefaults(defineProps<Props>(), {
     count: 0,
   });
   ```

3. **API 请求必须使用 Service 层**
   ```typescript
   // services/xxx.service.ts
   export const xxxService = {
     async getData() {
       // API 调用逻辑
     },
   };
   ```

### 2.2 代码质量标准

- 单个文件不超过 **300 行**
- 单个函数不超过 **50 行**
- 嵌套层级不超过 **3 层**
- 循环复杂度不超过 **10**

---

## 3. 命名规范

### 3.1 文件命名

```
组件文件：PascalCase.vue        例：UserProfile.vue
工具文件：camelCase.ts          例：formatDate.ts
类型文件：camelCase.types.ts   例：user.types.ts
样式文件：camelCase.module.css  例：card.module.css
```

### 3.2 变量命名

```typescript
// 组件名：PascalCase
const UserProfile: Component = ...

// 函数名：camelCase
function fetchUserData() {}
const handleSubmit = () => {}

// 常量：UPPER_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com'

// 接口/类型：PascalCase
interface UserInfo {}
type UserRole = 'student' | 'teacher'

// 布尔值：is/has/should 前缀
const isLoading = false
const hasError = true
const shouldUpdate = true
```

### 3.3 命名检查清单

- [ ] 变量名、函数名使用 camelCase
- [ ] 组件名使用 PascalCase
- [ ] 常量使用 UPPER_SNAKE_CASE
- [ ] 类型/接口使用 PascalCase
- [ ] 布尔值使用 is/has/should 前缀

---

## 4. TypeScript 规范

### 4.1 类型定义

```typescript
// ✅ 推荐：明确的类型定义
interface User {
  id: number;
  name: string;
  email: string;
  role: "student" | "teacher";
}

// ✅ 推荐：泛型使用
function fetchData<T>(url: string): Promise<T> {
  return axios.get<T>(url);
}

// ✅ 推荐：联合类型
type Status = "pending" | "success" | "error";

// ✅ 推荐：枚举
enum UserRole {
  Student = "student",
  Teacher = "teacher",
}
```

### 4.2 TypeScript 配置

```json
{
  "compilerOptions": {
    "strict": true,                    // 启用所有严格类型检查选项
    "noUnusedLocals": true,           // 未使用的局部变量报错
    "noUnusedParameters": true,       // 未使用的参数报错
    "noFallthroughCasesInSwitch": true, // switch 语句中遇到 break 报错
    "noImplicitReturns": true,        // 函数所有分支都必须有返回值
    "noUncheckedIndexedAccess": true, // 索引访问必须检查 undefined
    "noImplicitOverride": true,       // 标记覆盖的方法必须使用 override 关键字
    "allowUnusedLabels": false,       // 禁止未使用的标签
    "allowUnreachableCode": false     // 禁止不可达代码
  }
}
```

---

## 5. 组件开发规范

### 5.1 组件定义模板

```vue
<template>
  <div class="component-name">
    <!-- 模板内容 -->
  </div>
</template>

<script setup lang="ts">
// 1. 导入依赖
import { ref, computed, onMounted } from "vue";
import { useRouter } from "vue-router";

// 2. 定义 Props（withDefaults）
interface Props {
  title: string;
  count?: number;
}
const props = withDefaults(defineProps<Props>(), {
  count: 0,
});

// 3. 定义 Emits
interface Emits {
  (e: "update", value: string): void;
  (e: "delete", id: number): void;
}
const emit = defineEmits<Emits>();

// 4. 响应式数据
const isLoading = ref(false);
const dataList = ref<Item[]>([]);

// 5. 计算属性
const totalCount = computed(() => dataList.value.length);

// 6. 方法
const fetchData = async () => {
  try {
    isLoading.value = true;
    // API 调用
  } catch (error) {
    console.error("获取数据失败:", error);
  } finally {
    isLoading.value = false;
  }
};

// 7. 生命周期
onMounted(() => {
  fetchData();
});
</script>

<style scoped lang="scss">
/* 使用 Tailwind 类，复杂动画用 CSS */
</style>
```

### 5.2 组件分层

```
common/     - 通用组件(Button、Input、Modal、Loading、Message)
layout/     - 布局组件(HeaderBar、PageLayout、Sidebar)
business/   - 业务组件(OJCard、LeaderboardCard、UserCard)
```

### 5.3 组件通信

```typescript
// Props: 父 → 子
defineProps<{ title: string }>()

// Emits: 子 → 父
const emit = defineEmits<{ (e: 'update', value: string): void }>()

// Provide/Inject: 跨层级
provide('theme', theme)
const theme = inject('theme')

// Store: 跨组件
const store = useStore()
```

---

## 6. API 请求规范

### 6.1 核心原则

**重要**:
1. ✅ **Service 层只负责纯 API 调用**，不处理业务逻辑
2. ✅ **401 错误统一在 axios 拦截器中处理**（自动刷新 Token）
3. ✅ **业务逻辑在页面/组件中处理**（错误提示、跳转、状态更新）
4. ❌ **禁止在 Service 中使用 `try-catch` 包装业务逻辑**

### 6.2 Service 层规范

```typescript
// services/auth.service.ts
import apiClient from "@/utils/request";
import type { LoginRequest, LoginResponse } from "./types";

export const authService = {
  /**
   * 用户登录
   * Service 层只负责 API 调用，不处理业务逻辑
   */
  login(data: LoginRequest): Promise<LoginResponse> {
    return apiClient.post<LoginResponse>("/user/login", data);
  },

  /**
   * 用户注册
   */
  register(data: RegisterRequest): Promise<LoginResponse> {
    return apiClient.post<LoginResponse>("/user/register", data);
  },
};
```

### 6.3 Axios 封装规范

```typescript
// utils/request.ts
import axios from "axios";

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
  headers: {
    "Content-Type": "application/json",
  },
});

// 请求拦截器
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem("access_token");
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// 响应拦截器
apiClient.interceptors.response.use(
  (response) => response.data,
  async (error) => {
    // 401 错误统一处理：自动刷新 Token
    if (error.response?.status === 401) {
      // 刷新 Token 逻辑
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

### 6.4 API 配置选项

| 配置项             | 类型      | 说明               |
| --------------- | ------- | ---------------- |
| `skipTip`       | boolean | 完全静默（成功/失败都不弹提示） |
| `skipSuccTip`   | boolean | 成功不弹提示，失败正常提示    |
| `skipErrTip`    | boolean | 失败不弹提示，成功正常提示    |
| `customErrTip`  | string  | 自定义失败提示文案        |
| `customSuccTip` | string  | 自定义成功提示文案        |
|                 |         |                  |

**优先级（失败提示）**：`customErrTip` > `前端预设文案` > `后端 message` > `默认文案`

### 6.5 使用场景

```typescript
// 场景1：列表查询 → 完全静默
const response = await getRankingList('luogu', 1, 20, {
  skipTip: true
})

// 场景2：登录 → 成功不弹提示
const result = await authStore.login(phone, password, captcha, captchaId, {
  skipSuccTip: true
})

// 场景3：表单提交 → 自定义成功提示
const result = await registerApi(data, {
  customSuccTip: '注册成功！正在自动登录...'
})

// 场景4：验证码 → 成功不弹，失败自定义
const response = await getCaptcha(undefined, {
  skipSuccTip: true,
  customErrTip: '验证码加载失败，请检查网络'
})
```

---

## 7. 状态管理规范

### 7.1 Pinia Store 模板

```typescript
// stores/auth.ts
import { defineStore } from "pinia";
import { ref, computed } from "vue";

export const useAuthStore = defineStore("auth", () => {
  // 1. State
  const user = ref<User | null>(null);
  const token = ref<string>("");

  // 2. Getters
  const isLoggedIn = computed(() => !!user.value && !!token.value);

  // 3. Actions
  async function login(data: LoginRequest) {
    try {
      const response = await authService.login(data);
      user.value = response.user;
      token.value = response.accessToken;
      return response;
    } catch (error) {
      throw error;
    }
  }

  function logout() {
    user.value = null;
    token.value = "";
  }

  return {
    user,
    token,
    isLoggedIn,
    login,
    logout,
  };
});
```

### 7.2 状态管理原则

- ✅ 复杂状态使用 Pinia 管理
- ✅ 简单状态使用组件内部 `ref`
- ✅ 跨组件共享状态使用 Store
- ❌ 避免过度使用全局状态

---

## 8. 路由规范

### 8.1 路由配置

```typescript
// router/index.ts
const routes: RouteRecordRaw[] = [
  {
    path: "/login",
    name: "Login",
    component: () => import("@/views/Auth/LoginView.vue"),
    meta: { title: "登录", requiresAuth: false },
  },
  {
    path: "/",
    name: "Home",
    component: () => import("@/views/Home/HomeView.vue"),
    meta: { title: "首页", requiresAuth: true },
  },
];
```

### 8.2 路由守卫

```typescript
// router/guards.ts
router.beforeEach((to, from, next) => {
  const authStore = useAuthStore();
  const requiresAuth = to.meta.requiresAuth !== false;
  const isLoggedIn = authStore.isLoggedIn;

  if (requiresAuth && !isLoggedIn) {
    next({ name: "Login", query: { redirect: to.fullPath } });
  } else {
    next();
  }
});

router.afterEach((to) => {
  document.title = `${to.meta.title || "应用"}`;
});
```

---

## 9. 样式规范

### 9.1 Tailwind CSS 配置

```javascript
// tailwind.config.js
export default {
  theme: {
    extend: {
      colors: {
        primary: {
          DEFAULT: "#1890FF",
          hover: "#40A9FF",
          active: "#096DD9",
        },
        success: "#52C41A",
        warning: "#FAAD14",
        error: "#F5222D",
      },
      borderRadius: {
        card: "12px",
        button: "6px",
      },
      boxShadow: {
        card: "0 2px 8px rgba(0,0,0,0.08)",
      },
      animation: {
        flip: "flip 600ms ease-in-out",
      },
    },
  },
};
```

### 9.2 样式使用原则

```vue
<template>
  <!-- ✅ 推荐：使用 Tailwind 工具类 -->
  <div class="flex items-center justify-between p-4 bg-white rounded-lg shadow-md">
    <h2 class="text-lg font-semibold text-gray-900">标题</h2>
  </div>

  <!-- ❌ 避免：内联样式 -->
  <div style="display: flex; padding: 16px;">...</div>
</template>

<!-- 对于复杂动画，使用 CSS -->
<style scoped lang="scss">
@keyframes flip {
  0% { transform: rotateY(0deg); }
  100% { transform: rotateY(180deg); }
}

.flip-card {
  animation: flip 0.6s ease-in-out;
}
</style>
```

---

## 10. 代码审查规范

### 10.1 审查清单

#### 代码质量
- [ ] 代码通过 ESLint 检查（无警告）
- [ ] 代码通过 TypeScript 类型检查
- [ ] 代码通过 Prettier 格式化
- [ ] 没有注释掉的代码或调试代码
- [ ] 没有 console.log 或 debugger

#### 命名规范
- [ ] 变量、函数名使用 camelCase
- [ ] 组件名使用 PascalCase
- [ ] 常量使用 UPPER_SNAKE_CASE
- [ ] 类型/接口使用 PascalCase
- [ ] 布尔值使用 is/has/should 前缀

#### 代码结构
- [ ] 遵循项目分层结构
- [ ] 单个文件不超过 300 行
- [ ] 单个函数不超过 50 行
- [ ] 嵌套层级不超过 3 层
- [ ] 循环复杂度不超过 10

#### 性能优化
- [ ] 大列表使用虚拟滚动
- [ ] 图片使用懒加载
- [ ] 避免不必要的响应式数据
- [ ] 合理使用 computed 缓存
- [ ] 组件按需加载

#### 可维护性
- [ ] 关键逻辑有注释说明
- [ ] 复杂算法有文档说明
- [ ] 没有魔法数字
- [ ] 配置使用环境变量
- [ ] 错误处理完善

### 10.2 审查标准

1. **命名是否符合领域语言**
   - ✅ 变量名、函数名、组件名是否符合业务领域语言
   - ✅ 是否使用专业术语而非技术实现细节

2. **是否遵循项目分层结构**
   - ✅ 视图层只负责 UI 渲染和用户交互
   - ✅ 业务逻辑在 Composables 或 Store 中
   - ✅ API 调用统一在 Service 层

3. **是否有冗余或硬编码**
   - ✅ 是否有重复的代码逻辑
   - ✅ 配置是否使用环境变量
   - ✅ 魔法数字是否提取为常量

4. **是否有测试覆盖**
   - ✅ 核心业务逻辑是否有单元测试
   - ✅ 工具函数是否有测试用例
   - ✅ 组件是否有基础测试

---

## 11. 测试规范

### 11.1 单元测试

```typescript
// utils/validate.test.ts
import { validatePhone, validatePassword } from "../validate";

describe("validatePhone", () => {
  it("should validate correct phone number", () => {
    expect(validatePhone("13800138000")).toBe(true);
  });

  it("should reject invalid phone number", () => {
    expect(validatePhone("12345")).toBe(false);
  });
});
```

### 11.2 组件测试

```typescript
// components/common/Button/Button.test.ts
import { mount } from "@vue/test-utils";
import Button from "./Button.vue";

describe("Button", () => {
  it("renders button text", () => {
    const wrapper = mount(Button, { slots: { default: "Click me" } });
    expect(wrapper.text()).toContain("Click me");
  });

  it("emits click event", async () => {
    const wrapper = mount(Button);
    await wrapper.trigger("click");
    expect(wrapper.emitted("click")).toBeTruthy();
  });
});
```

### 11.3 测试覆盖要求

- 工具函数必须有测试
- 核心业务逻辑必须有测试
- 公共组件必须有基础测试
- API 调用需要有 Mock 测试
- 测试覆盖率建议达到 **80%** 以上

---

## 12. 配置文件规范

### 12.1 TypeScript 配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "moduleResolution": "bundler",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

### 12.2 Vite 配置

```typescript
// vite.config.ts
export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      imports: ["vue", "vue-router", "pinia"],
    }),
  ],
  resolve: {
    alias: {
      "@": resolve(__dirname, "src"),
    },
  },
  build: {
    target: "esnext",
    outDir: "dist",
    sourcemap: false,
    cssCodeSplit: true,
    assetsInlineLimit: 4096,
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          if (id.includes("node_modules")) {
            return "vendor";
          }
        },
      },
    },
    esbuild: {
      drop: ["console", "debugger"],
    },
  },
});
```

### 12.3 ESLint 配置

```javascript
// .eslintrc.cjs
module.exports = {
  root: true,
  env: {
    node: true,
    browser: true,
    es2022: true,
  },
  extends: [
    "plugin:vue/vue3-essential",
    "eslint:recommended",
    "@vue/typescript/recommended",
  ],
  parserOptions: {
    ecmaVersion: "latest",
  },
  rules: {
    "no-console": process.env.NODE_ENV === "production" ? "warn" : "off",
    "no-debugger": process.env.NODE_ENV === "production" ? "warn" : "off",
    "vue/multi-word-component-names": "off",
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": ["error", { argsIgnorePattern: "^_" }],
  },
};
```

### 12.4 Prettier 配置

```json
// .prettierrc
{
  "semi": false,
  "singleQuote": true,
  "printWidth": 100,
  "trailingComma": "none",
  "arrowParens": "avoid",
  "endOfLine": "auto"
}
```

---

## 13. 开发流程规范

### 13.1 模块化开发流程

```
1. 需求分析
   ├─ 阅读需求文档
   ├─ 理解用户故事
   └─ 确认验收标准

2. 设计阶段
   ├─ 组件设计
   ├─ API 接口设计
   └─ 类型定义

3. 开发阶段
   ├─ 创建类型定义 (types/)
   ├─ 创建 API 服务 (services/)
   ├─ 创建 Store (stores/)
   ├─ 创建组件 (components/)
   └─ 创建页面 (views/)

4. 测试阶段
   ├─ 单元测试
   ├─ 组件测试
   └─ 集成测试

5. 代码审查
   ├─ 自查代码
   ├─ 提交 PR
   └─ Code Review

6. 部署上线
   ├─ 构建生产版本
   ├─ 部署到服务器
   └─ 验证功能
```

### 13.2 开发优先级

```
Phase 1: 项目初始化
  - 创建项目
  - 配置开发环境
  - 搭建基础架构

Phase 2: 公共组件
  - Button
  - Input
  - Modal
  - Loading
  - Message

Phase 3: 核心功能
  - 用户认证
  - 业务功能
  - 数据展示

Phase 4: 优化与测试
  - 性能优化
  - 单元测试
  - E2E 测试
```

---

## 14. 文档规范

### 14.1 项目文档结构

```
docs/
├── CLAUDE.md                      # 项目主文档
├── requirement.md                 # 产品需求文档
├── 需求定义阶段文档.md             # 需求定义
├── API_CONFIGURATION_GUIDE.md     # API 配置指南
├── STATUS_CODE_USAGE.md           # 状态码使用指南
├── AXIOS_CONFIG_USAGE.md          # Axios 配置详解
├── AI前端开发提示词文档.md         # AI 开发提示词
├── AI-Vue模块化开发提示词.md       # Vue 模块化开发
└── UI需求与优化记录.md             # UI 需求文档
```

### 14.2 文档编写规范

1. **项目主文档 (CLAUDE.md)**
   - 项目概述
   - 技术栈
   - 项目结构
   - 代码规范
   - 开发流程
   - API 接口文档

2. **需求文档 (requirement.md)**
   - 功能需求
   - 用户角色
   - 业务流程
   - 验收标准

3. **API 文档**
   - 接口列表
   - 请求参数
   - 响应格式
   - 错误码说明
   - 使用示例

4. **开发提示词文档**
   - 模块化开发提示词
   - 按顺序开发
   - 包含完整代码示例
   - 进度追踪表

### 14.3 文档维护

- ✅ 每个功能模块完成后更新文档
- ✅ API 变更时及时更新接口文档
- ✅ 代码规范变更时更新规范文档
- ✅ 定期审查和优化文档

---

## 15. Git 提交规范

### 15.1 提交信息格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 15.2 Type 类型

```
feat:     新功能
fix:      修复 Bug
docs:     文档更新
style:    代码格式（不影响代码运行的变动）
refactor: 重构（既不是新增功能，也不是修复 Bug）
perf:     性能优化
test:     测试相关
chore:    构建过程或辅助工具的变动
```

### 15.3 示例

```bash
# 新功能
git commit -m "feat(auth): 添加用户登录功能"

# 修复 Bug
git commit -m "fix(api): 修复 Token 刷新失败的问题"

# 文档更新
git commit -m "docs: 更新 API 接口文档"

# 重构
git commit -m "refactor(components): 重构 Button 组件代码"
```

---

## 16. 环境变量规范

### 16.1 环境变量文件

```bash
# .env.development
VITE_API_BASE_URL=http://localhost:8080
VITE_API_PREFIX=/api

# .env.production
VITE_API_BASE_URL=https://api.example.com
VITE_API_PREFIX=/api
```

### 16.2 环境变量类型定义

```typescript
// env.d.ts
interface ImportMetaEnv {
  readonly VITE_API_BASE_URL: string;
  readonly VITE_API_PREFIX: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

### 16.3 使用原则

```typescript
// ✅ 正确：使用环境变量
const apiUrl = import.meta.env.VITE_API_BASE_URL;

// ❌ 错误：硬编码
const apiUrl = "http://localhost:8080";
```

---

## 17. 性能优化规范

### 17.1 代码分割

```typescript
// ✅ 路由懒加载
const Home = () => import("@/views/Home/HomeView.vue");

// ✅ 组件懒加载
const HeavyComponent = defineAsyncComponent(() =>
  import("@/components/HeavyComponent.vue")
);
```

### 17.2 资源优化

```typescript
// ✅ 图片懒加载
<img v-lazy="imageSrc" />

// ✅ 虚拟滚动（大列表）
<virtual-list :size="40" :remain="8" :data="list" />
```

### 17.3 构建优化

```typescript
// vite.config.ts
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        "vue-vendor": ["vue", "vue-router", "pinia"],
        "http-vendor": ["axios"],
      },
    },
  },
  esbuild: {
    drop: ["console", "debugger"],
  },
}
```

---

## 18. 常见问题与解决方案

### 18.1 类型问题

**问题**: 使用 `any` 类型
**解决**: 定义明确的接口或类型

**问题**: 类型断言滥用
**解决**: 使用类型守卫或泛型

### 18.2 性能问题

**问题**: 组件渲染过慢
**解决**:
- 使用 `computed` 缓存计算结果
- 使用 `v-show` 替代频繁的 `v-if`
- 大列表使用虚拟滚动

**问题**: 首屏加载慢
**解决**:
- 路由懒加载
- 组件按需加载
- 图片懒加载
- 开启 Gzip 压缩

### 18.3 代码质量问题

**问题**: 代码重复
**解决**: 提取公共组件或工具函数

**问题**: 函数过长
**解决**: 拆分为多个小函数

**问题**: 嵌套过深
**解决**: 使用早返回、 Guard Clauses

---

## 19. 最佳实践总结

### 19.1 代码编写

1. **优先使用 TypeScript 严格模式**
2. **遵循单一职责原则**
3. **保持函数简单纯粹**
4. **避免过早优化**
5. **先让代码工作，再让代码优雅**

### 19.2 团队协作

1. **统一的代码风格**
2. **完善的 Code Review**
3. **详细的文档说明**
4. **充分的测试覆盖**
5. **规范的 Git 提交**

### 19.3 项目管理

1. **模块化开发**
2. **版本控制规范**
3. **需求管理清晰**
4. **进度可追踪**
5. **问题及时记录**

---

## 20. 附录

### 20.1 推荐工具

- **IDE**: VSCode / WebStorm
- **代码格式化**: Prettier
- **代码检查**: ESLint
- **Git 工具**: GitKraken / SourceTree
- **API 测试**: Postman / Apifox
- **原型设计**: Figma

### 20.2 推荐插件

```
- Vue - Official
- TypeScript Vue Plugin (Volar)
- ESLint
- Prettier - Code formatter
- Auto Import
- Path Intellisense
```

### 20.3 学习资源

- [Vue 3 官方文档](https://cn.vuejs.org/)
- [TypeScript 官方文档](https://www.typescriptlang.org/zh/)
- [Vite 官方文档](https://cn.vitejs.dev/)
- [Pinia 官方文档](https://pinia.vuejs.org/zh/)
- [Tailwind CSS 官方文档](https://tailwindcss.com/)

---

**文档版本**: v1.0.0
**最后更新**: 2025-01-20
**维护人**: AI Assistant
**来源项目**: algorithm-platform

---

## 快速开始使用

1. **复制本文档到新项目**
   ```bash
   cp 通用项目规范总结.md your-project/docs/
   ```

2. **根据项目调整**
   - 技术栈差异
   - 团队习惯
   - 项目规模

3. **团队学习**
   - 代码审查时对照执行
   - 新人入职培训材料
   - 定期回顾更新

4. **持续优化**
   - 根据实际使用情况调整
   - 补充项目特定的规范
   - 记录最佳实践案例
