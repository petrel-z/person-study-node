url有长度限制，太长的字符串会传递失败，可改用[窗体通信](https://uniapp.dcloud.io/collocation/frame/communication)、[全局变量](https://ask.dcloud.net.cn/article/35021)，另外参数中出现空格等特殊字符时需要对参数进行编码，如下为使用`encodeURIComponent`对参数进行编码的示例

**核心用法**
- 使用 `encodeURIComponent`：对“参数值”做**编码**，确保**安全**地放入查询字符串或表单数据。
- 使用 `decodeURIComponent`：在接收端读取“参数值”后还原为原始字符串。

**常见场景**
- 路由携带参数：`uni.navigateTo({ url: '/pages/web-view/index?url=' + encodeURIComponent(link) })`
- 读取路由参数：`const decoded = decodeURIComponent(options.url)`
- 表单或查询参数组装：`key=value` 中的 `value` 用 `encodeURIComponent`
- 不是参数值而是“整条 URL”时，用 `encodeURI`（保留 `:/?#[]@!$&'()*+,;=` 等 URL 结构字符）

**区别说明**
- `encodeURIComponent` 编码范围更广，适合参数值；会编码 `?`, `&`, `=`, `#` 等。
- `encodeURI` 保留 URL 结构字符，适合整条 URL；不会编码 `?`, `&`, `=`, `#`。

**注意事项**
- 只编码一次，只解码一次：避免“二次编码/二次解码”导致错误。
- 先编码后解码：发送端编码，接收端解码；若未编码过就不要解码。
- 异常保护：`decodeURIComponent` 可能抛错，使用 `try...catch`。
- 安全校验：解码后先校验协议（例如只允许 `http/https`），再使用。

**在本项目中**
- 发送端（首页轮播）：`encodeURIComponent(link)` 放入 `url` 参数，避免 `?`、`&`、`#` 等字符破坏路由。
- 接收端（web-view 第 21 行）：`const decoded = decodeURIComponent(raw)` 还原原始链接，然后校验 `^https?://`，不合法跳转 `404`。
        