有了构建工具（如 Vite）后，项目代码的处理和执行方式会有显著的优化，以下从不同方面详细说明：

### 1. 开发阶段
#### （1）快速启动开发服务器
以 Vite 为例，它基于原生 ES 模块提供了开发服务器。当启动开发服务器时，Vite 不需要像传统构建工具那样对整个项目进行打包，而是直接启动服务器，速度极快。例如，在项目根目录下运行 `yarn dev` 即可启动 Vite 开发服务器。

#### （2）模块热更新（HMR）
Vite 支持轻量快速的热模块重载。当修改项目中的某个模块时，Vite 会只更新该模块，而不需要重新加载整个页面。例如，修改一个 Vue 组件的样式，页面会立即更新显示新的样式，而无需刷新整个页面。

#### （3）代码处理
- **语法转换**：构建工具可以将现代 JavaScript 语法（如 ES6+）转换为浏览器兼容的语法。例如，使用 Babel 插件将 `async/await` 语法转换为 ES5 兼容的代码。
- **代码分割**：构建工具可以将大的代码文件分割成多个小的文件，实现按需加载。例如，Vite 会自动对代码进行分割，提高页面加载速度。

### 2. 生产阶段
#### （1）代码打包
构建工具会将项目中的所有代码和资源打包成一个或多个文件，减少浏览器的请求次数。例如，Vite 使用 Rollup 打包代码，输出用于生产环境的优化过的静态资源。

#### （2）代码压缩
构建工具会对代码进行，去除不必要的空格、注释等，减小文件体积。例如，在 Vite 的 `vite.config.ts` 中可以配置 `terserOptions` 来压缩代码：
```typescript
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true
      }
    }
  }
})
```

#### （3）资源优化
构建工具可以对图片、字体等资源进行优化，例如压缩图片大小、转换字体格式等。

#### （4）环境变量配置
构建工具支持不同环境的配置，例如开发环境和生产环境可以使用不同的配置文件。在 Vite 中，可以在项目根目录下创建 `.env.development` 和 `.env.production` 文件来配置不同环境的变量：
```plaintext
# .env.development
NODE_ENV=development
VITE_APP_WEB_URL= 'YOUR DEV WEB URL'

# .env.production
NODE_ENV=production
VITE_APP_WEB_URL= 'YOUR PROD WEB URL'
```

### 3. 代码风格约束
#### （1）ESLint 支持
可以使用 ESLint 来检查代码的语法和风格。在项目中安装 ESLint 及其相关插件，并配置 `.eslintrc.js` 文件：
```bash
# 安装 ESLint
yarn add eslint --dev
# 安装 ESLint 插件
yarn add eslint-plugin-vue --dev
yarn add @typescript-eslint/eslint-plugin --dev
yarn add eslint-plugin-prettier --dev
# 安装 TypeScript 解析器
yarn add @typescript-eslint/parser --dev
```
```javascript
// .eslintrc.js
module.exports = {
  root: true,
  env: {
    browser: true,
    node: true,
    es2021: true,
  },
  parser: 'vue-eslint-parser
  extends: [
    'eslint:recommended',
    'plugin:vue/vue3-recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:prettier/recommended',
    'prettier',
  ],
  // 其他配置...
}
```

#### （2）Prettier 支持
使用 Prettier 来格式化代码，解决 ESLint 和 Prettier 冲突可以安装 `eslint-config-prettier` 插件：
```bash
# 安装 Prettier
yarn add prettier --dev
# 安装解决冲突的插件
yarn add eslint-config-prettier --dev
```
```javascript
// .prettier.js
module.exports = {
  tabWidth: 2,
  jsxSingleQuote: true,
  jsxBracketSameLine: true,
  printWidth: 100,
  singleQuote: true,
  semi: false,
  overrides: [
    {
      files: '*.json',
      options: {
        printWidth: 200,
      },
    },
  ],
  arrowParens: 'always',
}
```

### 4. 其他优化
#### （1）配置文件引用别名
可以在构建工具的配置文件中配置引用别名，方便代码中引用文件。例如，在 Vite 的 `vite.config.ts` 中配置别名：
```typescript
import { defineConfig } from 'vite'
import path from 'path'

export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },
})
```

#### （2）CSS 预处理器支持
构建工具支持使用 CSS 预处理器（如 SCSS），可以安装相应的预处理器依赖并配置全局样式文件。例如，在 Vite 中配置 SCSS：
```bash
# 安装 SCSS 预处理器
yarn add dart-sass --dev
yarn add sass --dev
```
```typescript
// vite.config.ts
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: '@import "@/assets/style/mian.scss";'
      }
    }
  }
})
```

综上所述，有了构建工具后，项目的开发和部署过程更加高效、优化，能够解决没有构建工具时存在的各种问题。