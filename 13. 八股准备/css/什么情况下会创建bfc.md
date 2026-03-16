 层叠上下文决定了元素在三维空间（X、Y、Z轴）中的堆叠顺序。

  以下情况会创建一个新的层叠上下文：

  1. 根元素 (`<html>`)

  html {
    /* 根层叠上下文 */
  }

  2. position + z-index（最常见）

  /* ❌ 创建层叠上下文 */
  .element {
    position: relative;  /* 或 absolute, fixed, sticky */
    z-index: 0;         /* 任何数值（包括0）都创建 */
  }

  /* ✅ 不创建层叠上下文 */
  .element {
    position: relative;
    z-index: auto;      /* auto 不创建 */
  }

  /* 或者 */
  .element {
    position: relative;  /* 没有 z-index */
  }

  这就是您的问题所在：
  // ❌ MainLayout.tsx 之前的问题:`
  ```tsx
  <main className="flex-1 ... relative z-0">
    {/* 创建了新的层叠上下文！ */}
    <JournalDetailPage>
      <PinVerifyDialog className="fixed z-[9999]">
        {/* 只能在这个层叠上下文内部堆叠 */}
        {/* 无法覆盖外面的 Sidebar (z-10) */}
      </PinVerifyDialog>
    </JournalDetailPage>
  </main>
  ```

  3. position: fixed 或 sticky

  /* 即使没有 z-index，也会创建层叠上下文 */
  .element {
    position: fixed;
  }

  .element {
    position: sticky;
  }

  4. opacity 小于 1

  /* 即使 opacity: 0.99，也会创建层叠上下文 */
  .element {
    opacity: 0.5;
  }

  5. CSS Transform

  .element {
    transform: translateX(10px);
  }

  .element {
    transform: scale(1.1);
  }

  6. filter 不为 none

  .element {
    filter: blur(5px);
  }

  7. Flex/Grid 容器的子元素

  /* 父元素 */
  .parent {
    display: flex;
    z-index: 1;  /* 父元素有 z-index，子元素创建新的层叠上下文 */
  }

  .child {
    /* 子元素在新的层叠上下文中 */
  }

  8. 其他情况

  .element {
    perspective: 1000px;
  }

  .element {
    isolation: isolate;
  }

  .element {
    mix-blend-mode: multiply;
  }

  .element {
    contain: strict;
  }

  ---
  层级问题示例

  场景 1：嵌套的层叠上下文（您的问题）

  /* DOM 结构 */
  <div class="sidebar" style="z-index: 10;">Sidebar</div>

  <div class="main" style="position: relative; z-index: 0;">
    <!-- 创建了新的层叠上下文 -->
    <div class="modal" style="position: fixed; z-index: 9999;">
      Modal
      <!-- 只能跟 main 内部的元素比 -->
      <!-- 无法覆盖 sidebar (z-10) -->
    </div>
  </div>

  问题：
  - main 有 relative + z-index: 0，创建了层叠上下文
  - modal 虽然用 fixed + z-index: 9999
  - 但它被限制在 main 的层叠上下文中
  - sidebar 在 main 的外部，有自己的 z-index: 10
  - 结果：modal 无法覆盖 sidebar

  解决方案 1：移除 main 的层叠上下文
  .main {
    /* 不设置 position: relative，或设置 z-index: auto */
  }

  解决方案 2：使用 Portal
  // 将 modal 渲染到 document.body
  return createPortal(<Modal />, document.body);

  ---
  场景 2：Transform 导致的层叠上下文
```tsx
<div class="parent">
    <div class="child" style="transform: scale(1); z-index: 1;">
      <!-- transform 创建了层叠上下文 -->
    </div>
    <div class="sibling" style="z-index: 2;">
      <!-- 无法覆盖 child，因为 child 在独立的层叠上下文中 -->
    </div>
  </div>
```
  <div class="parent">
    <div class="child" style="transform: scale(1); z-index: 1;">
      <!-- transform 创建了层叠上下文 -->
    </div>
    <div class="sibling" style="z-index: 2;">
      <!-- 无法覆盖 child，因为 child 在独立的层叠上下文中 -->
    </div>
  </div>

  ---
  场景 3：Opacity 导致的层叠上下文

  <div class="container" style="opacity: 0.9;">
    <!-- opacity 创建了层叠上下文 -->
    <div class="modal" style="position: fixed; z-index: 9999;">
      <!-- 无法覆盖容器外部的元素 -->
    </div>
  </div>
```tsx
<div class="container" style="opacity: 0.9;">
    <!-- opacity 创建了层叠上下文 -->
    <div class="modal" style="position: fixed; z-index: 9999;">
      <!-- 无法覆盖容器外部的元素 -->
    </div>
  </div>
```
  ---
  层叠上下文层级规则

  在同一个层叠上下文中，元素按以下顺序堆叠（从下到上）：

  1. 根元素的背景和边框
  2. 块级元素（按 DOM 顺序）
  3. 浮动元素
  4. 定位元素（按 z-index 顺序）
  5. 子层叠上下文（按父元素的 z-index 顺序）

  关键点：
  - 子元素的 z-index 只在当前层叠上下文内有效
  - 不同层叠上下文之间，比较的是父元素的 z-index
  - 这就是为什么 z-index: 9999 无法覆盖外部元素的原因

  ---
  调试层叠问题的方法

  1. 使用浏览器开发工具

  // 在控制台运行
  document.querySelectorAll('*').forEach(el => {
    const zIndex = window.getComputedStyle(el).zIndex;
    if (zIndex !== 'auto') {
      console.log(el, zIndex);
    }
  });

  2. 可视化层叠上下文

  * {
    outline: 1px solid red;
  }

  /* 或使用浏览器扩展 */

  3. 查找层叠上下文

  // 查找所有创建层叠上下文的元素
  const stackContexts = [];
  document.querySelectorAll('*').forEach(el => {
    const style = window.getComputedStyle(el);
    const zIndex = style.zIndex;
    const position = style.position;
    const opacity = style.opacity;
    const transform = style.transform;

    if ((position !== 'static' && zIndex !== 'auto') ||
        opacity < 1 ||
        transform !== 'none') {
      stackContexts.push({
        element: el,
        zIndex,
        position,
        opacity,
        transform
      });
    }
  });
  console.table(stackContexts);

  ---
  您的问题总结

  问题的根本原因

  // MainLayout.tsx
  <main className="relative z-0">
    {/* ^^^ 创建了新的层叠上下文 */}

    <JournalDetailPage>
      <PinVerifyDialog className="fixed z-[9999]">
        {/* 只能在这个层叠上下文内堆叠 */}
      </PinVerifyDialog>
    </JournalDetailPage>
  </main>

  <Sidebar className="z-10">
    {/* 在外部层叠上下文 */}
  </Sidebar>

  解决方案

  方案 1：移除层叠上下文 ✅（当前方案）
  <main className="flex-1 h-screen overflow-y-auto">
    {/* 不设置 relative z-0 */}
  </main>

  方案 2：提高父元素的 z-index
  <div className="main-wrapper" style="z-index: 100;">
    <main className="relative z-0">
      <PinVerifyDialog className="fixed z-[9999]">
        {/* 现在可以覆盖 Sidebar 了 */}
      </PinVerifyDialog>
    </main>
  </div>

  方案 3：使用 Portal ⚠️
  return createPortal(
    <PinVerifyDialog />,
    document.body  // 渲染到根层级
  );

  ---
  最佳实践建议

  4. 避免滥用层叠上下文
    - 不要随意添加 position: relative
    - 不要随意设置 z-index: 0
  5. 使用语义化的 z-index
  /* 推荐的层级 */
  --z-dropdown: 100;
  --z-sticky: 200;
  --z-fixed: 300;
  --z-modal-backdrop: 400;
  --z-modal: 500;
  --z-popover: 600;
  --z-tooltip: 700;
  6. 对于模态框/弹窗
    - 简单应用：调整 z-index 即可
    - 复杂应用：使用 Portal
  7. 调试技巧
    - 使用开发工具的 "Layers" 或 "3D" 视图
    - 避免嵌套的层叠上下文

  ---
  结论：您的问题是因为 main 元素的 relative z-0 创建了新的层叠上下文。移除它后，PIN 弹窗就回到了根层叠上下文，可以正常覆盖所有元素了！