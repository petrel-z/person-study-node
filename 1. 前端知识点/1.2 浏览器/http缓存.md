### 一、为什么需要缓存

1. 网络传输的瓶颈与延迟
如果没有缓存，加载网页时，浏览器每次都需要向服务端发送请求，增大服务器端的压力。如果有缓存，页面资源直接从缓存中获取，不仅可以可以减少请求服务端的次数了，还能提升页面加载速度。

2. 缓存的核心目标：减少请求次数 & 加快加载速度

---

### 二、HTTP 缓存位置和缓存策略

1. 缓存位置
    - 浏览器缓存
    - 代理服务器缓存（如 CDN）
2. 缓存策略分类
    - 强缓存（Freshness）
    - 协商缓存（Validation）

#### 浏览器缓存位置
1. 内存缓存
2. 磁盘缓存
3. Service Worker Cache
4. localStorage
5. **HTTP/2 Push Cache / Push Promises**：临时性缓存用于 HTTP/2 推送场景。

### 缓存发展史
提出缓存的概念后，虽然缓存为服务器扛下了很多，但又出现了新的问题，会出现服务器的资源更新了，浏览器还是从缓存中获取资源，导致浏览器获取的还是旧的资源，也就是说**客户端不知道资源何时过期**
![[Pasted image 20251018174507.png]]
Expires的出现，是用于给缓存设置一个过期时间（一个具体的时间点）
浏览器从缓存中拿资源的时候，根据这个过期时间判断缓存是否过期
但Expires有一个弊端：它依赖客户端系统时间，若服务器时间和客户端时间不一致（可能会导致缓存实际有效时长和服务端预期的有效时长不一致），这会导致**缓存不准确**

客户端（浏览器所在机器）的时间可能因为：
- 用户手动修改时间；
- 系统时区设置不同；
- 没有联网同步时间；
- 浏览器运行在不同地区（跨时区）；
导致与服务器时间相差几分钟甚至几小时。

于是Cache-Control就诞生了，它表示缓存的过期时长
可是cache-control的出现，服务器还是扛不住太多的请求
于是缓存被分为强缓存和协商缓存

### 三、强缓存机制（Freshness）

1. **概念：**
如果资源在本地缓存中且仍在有效期内，浏览器直接使用，不向服务器发送请求。

2. **常用字段：**
- `Cache-Control`（HTTP/1.1，优先级高）：
    - `max-age=<seconds>`：缓存过期时长，是相对时间，资源在本地被视为“新鲜”的时间长度。从响应生成开始计算，这段时间内，资源在本地是‘新鲜’的，不会过期，浏览器在这段时间内不用重复请求服务端，直接从缓存中获取
    - `public`：可以被任何缓存（包括代理、CDN、浏览器）缓存。
    - `private`：只允许浏览器缓存，代理不可缓存（适用于含有用户私有信息的响应）。
    - `no-cache`：有缓存，但在使用缓存前必须向服务端重新验证资源是否有效。
    - `no-store`：不允许缓存任何版本（最严格，不写入磁盘/内存）。
    - `must-revalidate`：超过 max-age 后必须重新向源服务器校验。
    - `immutable`：资源声明永不变（适合带 hash 的静态资源），浏览器在生命周期内不会再次请求。
    - `stale-while-revalidate=<seconds>` / `stale-if-error`：现代扩展，允许在后台重新验证的同时返回陈旧内容或在错误时返回陈旧内容（并非所有浏览器/代理都支持）。
- `Expires: <HTTP-date>`（（HTTP/1.0 旧标准））：缓存过期时间。（绝对时间，用于兼容，但会被 `Cache-Control` 覆盖）受本地时钟影响

`Cache-Control`的优先级大于`Expires`，如果`Cache-Control`设置了max-age，那么`Expires`会被忽略

*响应示例（部分）：*
HTTP/1.1 200 OK
Cache-Control: public, max-age=31536000, immutable
            
3. **强缓存命中流程**
核心流程：
浏览器先在缓存中检查是否有这个资源
	资源在本地：
		没过期 -> 直接从缓存中拿出来使用
		过期 -> 进入协商缓存阶段
	不在本地：
		直接请求，请求完毕将资源添加到缓存

    
---
#### 问题1：如何检验资源是否过期
根据http协议的源码，计算原理：
response_is_fresh = (freshness_lifetime > current_age)
freshness_lifetime是新鲜度有效期
current_age是**资源的实际存活时间**

有3种方式计算freshness_lifetime：
1. freshness_lifetime = max-age：
	从资源返回给客户端的时间开始计算，过了max-age时间后过期，是一个相对时间。
	但又有小伙伴问了，服务端返回响应时有网络延迟，延迟的时间不也是偏差吗？
	对的，http在设计的时候认为这个网络延迟是在合理的偏差内（通常在几十毫秒~几秒之间），相比如Expires产生的偏差，可能是几秒、几小时、甚至几天！
2. freshness_lifetime = `Expires`- date_value
	date_value的值是Date或或者response_time（客户端接受响应的时间，当Date不存在时，才用response_time）
	这两个字段都是服务端返回的，因此不存在时钟偏差
3. 兜底策略（有Last-modified）：
	freshness_lifetime  = (date_value - Last-modified) * 10%
	设计思路：如果资源更新频繁，那么’新鲜期‘应该更短

如何计算current_age呢？
它的计算需要now的参与
**`now`**：
    执行计算的主机（可能是客户端或代理服务器）的 “当前时间”。规范建议用 NTP 等协议同步到 UTC 时间（避免时钟偏差）。

https://github.com/chromium/chromium/blob/main/net/http/http_response_headers.cc#L1107-L1114

第1204行
源码：
```
// From RFC 2616 section 13.2.4:
//
// The max-age directive takes priority over Expires, so if max-age is present
// in a response, the calculation is simply:
//
//   freshness_lifetime = max_age
//
// Otherwise, if Expires is present in the response, the calculation is:
//
//   freshness_lifetime = expires_value - date_value
//
// Note that neither of these calculations is vulnerable to clock skew, since
// all of the information comes from the origin server.
//
// Also, if the response does have a Last-Modified time, the heuristic
// expiration value SHOULD be no more than some fraction of the interval since
// that time. A typical setting of this fraction might be 10%:
//
//   freshness_lifetime = (date_value - last_modified_value) * 0.10
//
// If the stale-while-revalidate directive is present, then it is used to set
// the |staleness| time, unless it overridden by another directive.

```

#### 问题2：浏览器第一次请求时，会在缓存中保存哪些信息？
服务端响应：
```
HTTP/1.1 200 OK
Cache-Control: max-age=600, public
ETag: "abc123"
Last-Modified: Wed, 17 Oct 2025 08:00:00 GMT
Content-Type: application/javascript

```

*缓存记录*：
1. 浏览器请求的资源
2. 响应头
	1. Cache-Control
	2. ETag
	3. Last-Modified
	4. 浏览器缓存该资源的时间戳

#### 问题3：浏览器在下次请求资源时，如何检查缓存中是否有该资源
根据想要获取的资源的完整url（包括查询参数、协议、端口），在缓存中查找，同时还会检查其他维度（请求方法、Vary头）来确认缓存是否匹配。
就检查是否过期，没过期就直接使用（强缓存命中）
如果过期，就进入协商缓存

#### 问题4：客户端和服务端时间不一致，或网络传输存在延迟时，max-age 会不会不准？
假设服务器返回：
`Cache-Control: max-age=600 Date: Fri, 17 Oct 2025 12:00:00 GMT`
但浏览器在 **12:00:05** 才收到响应（中间花了 5 秒网络传输时间）。
那问题来了：  
服务器认为资源有效期是 **从 12:00:00 起算的 600 秒**，  
而浏览器真正缓存是在 **12:00:05**。
👉 理论上，浏览器如果直接用自己的缓存时间来算，会**多缓存 5 秒**。

http协议考虑到了这一点，浏览器会根据服务端返回的响应头中的Date字段（Date是服务端生成响应的时间），来校正资源缓存的时间，浏览器在接收到响应时，会通过（当前时间-Date字段的时间，来计算网络延迟时间）

### 四、协商缓存机制（Validation）

1. 概念：缓存已过期，但可与服务器验证是否可用
    
2. 相关响应头
        
    - `ETag` / `If-None-Match`
	    - ETag是服务端返回的，它用来唯一标识资源是否改变，如果资源改变，这个值就会改变。
	    - 浏览器收到ETag会，会和资源缓存在一块，下次请求时会在请求头 `If-None-Match`中携带这个ETag标识
    - `Last-Modified` / `If-Modified-Since`
	    - `Last-Modified`是资源最后的修改时间，可以用来判断资源是否过期
	    - 该字段会被缓存，当没有ETag时，会检查是否有`Last-Modified`，如果有就下次请求时，将该字段的值放`If-Modified-Since`字段中，发送给服务端，服务端返回304，表示该资源没变更，可继续使用，如果返回200，表明资源变更，返回新的资源。
    
3. ETagLast-Modified的优劣比较
    

## 整体流程
浏览器发送请求，服务端响h应
如果有强缓存：**cache-control > Expires** ，浏览器判断强缓存是否过期，如果没有过期，直接从缓存中获取资源
1. 过期则先检查缓存信息中上次请求的响应头中是否有**Etag**，如果有则重新发送请求，
	1. 请求头中携带if-none-match, 服务端拿到这个if-none-match的值，判断服务端的文件是否修改，如果修改，则返回200状态，返回最新的资源；否则返回304，浏览器从缓存中获取资源。

2. 如果响应头中没有Etag，则看响应头中是否有last-modified
	1. 如果有则浏览器发送请求时，在请求头携带if-Modified-Since, 服务端根据这个字段检测资源是否更新，如果资源没更新，则返回304，浏览器从缓存中获取资源；如果资源更新，则返回200，返回更新后的资源。

如果既没有Etag，也没有last-modified，则说明没有缓存，重新发送http请求获取资源

