UnoCSS 是一个具有高性能且极具灵活性的即时原子化 CSS 引擎，它采用按需生成的方式，能够极大地提升开发效率。
**常用的 UnoCSS 语法：**

### 1. 基础工具类
- **尺寸与间距**
  - `p-4`：设置内边距为 1rem（假设使用默认配置）
  - `m-2`：设置外边距为 0.5rem
  - `mt-8`：设置顶部外边距为 2rem
  - `px-6`：设置左右内边距为 1.5rem
- **颜色**
  - `text-red-500`：设置文本颜色为红色（中等深度）
  - `bg-blue-100`：设置背景颜色为浅蓝色
  - `border-green-700`：设置边框颜色为深绿色
- **排版**
  - `text-lg`：设置文本大小为大尺寸
  - `font-bold`：设置字体为粗体
  - `italic`：设置字体为斜体
  - `tracking-wider`：设置字符间距为较宽
- **边框与圆角**
  - `border`：添加默认边框
  - `rounded-lg`：设置圆角为大尺寸
  - `border-2`：设置边框宽度为 2px
  - `rounded-t-none`：设置顶部圆角为无

### 2. 响应式设计
UnoCSS 支持通过前缀来实现响应式设计：
- `sm:text-center`：在小屏幕上设置文本居中
- `md:flex`：在中等屏幕上使用 flex 布局
- `lg:w-1/2`：在大屏幕上设置宽度为 50%
- `xl:p-8`：在超大屏幕上设置内边距为 2rem

### 3. 状态变体
可以通过前缀来处理不同的状态：
- `hover:bg-gray-200`：鼠标悬停时背景变为浅灰色
- `focus:outline-none`：元素获得焦点时移除轮廓
- `active:scale-95`：元素被激活时缩小为 95%
- `disabled:opacity-50`：元素禁用时透明度为 50%

### 4. 任意值
UnoCSS 允许使用方括号语法设置任意值：
- `text-[#3B82F6]`：设置文本颜色为自定义十六进制颜色
- `w-[240px]`：设置宽度为 240px
- `rotate-[15deg]`：旋转 15 度
- `bg-[url('/image.jpg')]`：设置背景图片

### 5. 组合与缩写
UnoCSS 支持组合多个工具类，也有一些缩写形式：
- `flex items-center justify-between`：设置 flex 布局并垂直居中、水平两端对齐
- `shadow-md`：添加中等阴影
- `transition-all duration-300`：添加过渡效果，持续时间 300ms
- `grid-cols-3`：设置网格布局为 3 列

### 6. 自定义工具类
可以通过配置文件定义自己的工具类：
```javascript
// unocss.config.js
export default {
  shortcuts: {
    'btn-primary': 'bg-blue-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-blue-600 transition-colors',
    'card': 'bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow',
  },
}
```
使用自定义工具类：
```html
<button class="btn-primary">点击我</button>
<div class="card">这是一个卡片</div>
```

### 7. 动画与过渡
- `animate-spin`：应用旋转动画
- `animate-pulse`：应用脉冲动画
- `transition-transform`：为变换添加过渡效果
- `duration-500`：设置过渡持续时间为 500ms

### 8. 布局
- `flex`：启用 flex 布局
- `grid`：启用网格布局
- `justify-center`：水平居中
- `items-end`：垂直底部对齐
- `gap-4`：设置间距为 1rem

### 9. 自定义配置
在 `unocss.config.js` 中可以自定义主题、工具类等：
```javascript
export default {
  theme: {
    extend: {
      colors: {
        primary: '#165DFF',
        secondary: '#36CFC9',
        neutral: {
          100: '#F5F7FA',
          200: '#E4E7ED',
          300: '#C0C4CC',
        }
      },
      fontFamily: {
        inter: ['Inter', 'sans-serif'],
      },
    },
  },
}
```

