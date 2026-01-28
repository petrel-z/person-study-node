# xss攻击形成：
网站漏洞：input用户输入的内容，未经过特殊字符转义，直接通过innerHTML渲染到页面中，导致如果输入script标签，浏览器会直接执行该标签的内容
## 防止xss攻击的方式：
对input内容的特殊字符进行转义，展示时是纯文本字符，不会被浏览器执行
```js
<script>
  // 1. 新增：转义函数（将特殊字符转为HTML实体）
    function escapeHtml(str) {
        if (!str) return '';
        // 替换 < > " ' & 这5个关键字符
        return str.replace(/&/g, '&amp;')
                 .replace(/</g, '&lt;')
                 .replace(/>/g, '&gt;')
                 .replace(/"/g, '&quot;')
                 .replace(/'/g, '&#039;');
    }

   // 2. 修复提交留言函数：使用转义后的内容
    function addMessage() {
        const input = document.getElementById('messageInput').value;
        const escapedInput = escapeHtml(input); // 先转义！
        const messageList = document.getElementById('messageList');
        // 插入转义后的内容（此时 <script> 会变成纯文本，不会执行）
        messageList.innerHTML += `<p>用户留言：${escapedInput}</p>`;
        document.getElementById('messageInput').value = '';
    }
</script>
```
方式二：
使用浏览器的内容安全策略（CSP)
通过http响应头（`Content-Security-Policy`）或者html标签的（ `<meta http-equiv="Content-Security-Policy">`），来限制浏览器可加载资源的来源
明确指定可加载资源类型的来源
```http
Content-Security-Policy: 
  default-src 'self';  # 默认所有资源仅允许来自当前域名
  script-src 'self' https://cdn.jsdelivr.net;  # 脚本仅允许当前域名+jsDelivr CDN
  style-src 'self' 'unsafe-inline';  # 样式允许当前域名+内联样式（需谨慎启用）
  img-src 'self' data:;  # 图片允许当前域名+base64格式
  connect-src 'self';  # AJAX/fetch仅允许当前域名
  object-src 'none';  # 禁止加载插件（如Flash）
```
