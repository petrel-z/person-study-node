# TCP/UDP 面试知识点分享会

## 一、开场引入

### 为什么面试总问 TCP/UDP？

1. **基础中的基础** — 网络编程是后端开发的必备知识，TCP/UDP 是网络协议栈的核心
2. **考察系统性思维** — 从三次握手到四次挥手，考察你对一个连接生命周期的理解
3. **实际问题的折射** — 网络问题（超时、丢包、连接断开）的根因往往在 TCP 层面
4. **区分度明显** — 背八股文的人多，能深入原理的人少，这是拉开差距的好题目

### 常见的面试问题类型

| 类型   | 示例问题                      |
| ---- | ------------------------- |
| 概念对比 | TCP 和 UDP 的区别？各适用什么场景？    |
| 原理深挖 | 为什么是三次握手？不是两次或四次？         |
| 状态分析 | TIME_WAIT 是什么？过多怎么处理？     |
| 实战问题 | CLOSE_WAIT 过多怎么排查？        |
| 扩展延伸 | 如何基于 UDP 实现可靠传输？QUIC 了解吗？ |
| 编程实践 | socket 编程中 TCP/UDP 的区别？   |

---

## 二、TCP vs UDP 核心区别

### 1. 连接方式

|      | TCP            | UDP                       |
| ---- | -------------- | ------------------------- |
| 连接方式 | 面向连接（三次握手建立连接） | 无连接（直接发送数据）               |
| 实现   | 通过握手建立端到端逻辑通道  | 无需建立连接，直接 sendto/recvfrom |

**生活中的类比：**
- TCP 像打电话，必须等对方接起来才能说话（建立连接 → 通话 → 挂断）
- UDP 像发快递，把包裹扔给快递员就不管了（直接发送，不保证送达）

### 2. 可靠性

| | TCP | UDP |
|--|-----|-----|
| 可靠性 | 可靠传输（确认、重传、排序） | 尽力而为（不保证送达、不排序） |
| 具体机制 | ARQ、滑动窗口、超时重传、序列号 | 无 |

### 3. 拥塞控制与流量控制

**TCP 有，UDP 没有：**

- **拥塞控制**：防止发送方过多数据压垮网络
  - 慢启动（cwnd 从小到大指数增长）
  - 拥塞避免（达到阈值后线性增长）
  - 快恢复（丢包后快速调整）

- **流量控制**：防止发送方过快导致接收方 buffer 溢出
  - 通过滑动窗口（rwnd）协调双方发送速率

**UDP 为什么不支持？**
- UDP 设计初衷是"简单"，把复杂度交给应用层
- 实时性要求高的场景（如直播、视频会议）宁可丢包也不愿重传带来的延迟

### 4. 首部开销

```
TCP 首部（20字节起步）：
┌─────────────────────────────────────────────────────────┐
│  源端口(16)  │  目标端口(16)                              │
├─────────────────────────────────────────────────────────┤
│                      序列号(32)                          │
├─────────────────────────────────────────────────────────┤
│                    确认号(32)                             │
├────────┬─────────┬───────────────────────────────────────┤
│offset(4)│ 标志位  │              窗口大小(16)             │
├────────┴─────────┼───────────────────────────────────────┤
│    校验和(16)    │           紧急指针(16)                  │
├─────────────────┴───────────────────────────────────────┤
│                      选项（可变长度）                      │
└─────────────────────────────────────────────────────────┘

UDP 首部（8字节）：
┌─────────────────────────────────────────────────────────┐
│  源端口(16)  │  目标端口(16)                              │
├─────────────────────────────────────────────────────────┤
│    长度(16)   │          校验和(16)                        │
└─────────────────────────────────────────────────────────┘
```

### 5. 传输效率与使用场景

| 特性 | TCP | UDP |
|------|-----|-----|
| 传输效率 | 较低（因拥塞控制、重传） | 高（无多余开销） |
| 首部开销 | 20-60字节 | 8字节 |
| 数据顺序 | 保证顺序 | 不保证 |
| 全双工 | 是 | 是 |

**典型应用场景：**

| TCP 适用于 | UDP 适用于 |
|-----------|-----------|
| HTTP/HTTPS | DNS 查询 |
| 文件传输 (FTP) | 实时音视频（直播、视频会议） |
| SSH/邮件 | 在线游戏（对实时性要求高） |
| 数据库连接 | 广播/多播通信 |
| 金融交易（强一致性） | QUIC（基于 UDP 的可靠传输） |

---

## 三、TCP 三次握手（深挖点1）

### 每一次握手的状态变化与含义

先看完整时序图：

```
Client                                          Server
  │                                               │
  │───────────  SYN=1, seq=x  (客户端发送SYN)  ──────────→│  LISTEN
  │  CLOSED                                         │  (服务器监听)
  │      ↓                                          │
  │  SYN_SENT  ←───────────────────────────────  │  SYN_RCVD
  │  (已发送SYN)                                    │  (收到SYN)
  │                                                │
  │      ←────  SYN=1, ACK=1, seq=y, ack=x+1  ──────────│
  │  ESTABLISHED                                   │  (服务器发送SYN+ACK)
  │      ↓                                          │
  │  ESTABLISHED  ───────  ACK=1, seq=x+1, ack=y+1  ─────→│
  │  (确认收到)                                     │  ESTABLISHED
  │                                                │  (双方建立连接)
```

**详细状态变化：**

| 步骤    | Client 状态              | Server 状态              | 关键动作                                   |
| ----- | ---------------------- | ---------------------- | -------------------------------------- |
| 初始    | CLOSED                 | LISTEN                 | Server 被动打开，开始监听端口                     |
| 第一次握手 | CLOSED → SYN_SENT      | LISTEN                 | Client 发送 SYN，序列号=x，进入 SYN_SENT        |
| 第二次握手 | SYN_SENT               | LISTEN → SYN_RCVD      | Server 发送 SYN+ACK，序列号=y，确认号=x+1        |
| 第三次握手 | SYN_SENT → ESTABLISHED | SYN_RCVD → ESTABLISHED | Client 发送 ACK，确认号=y+1，双方进入 ESTABLISHED |
|       |                        |                        |                                        |

### 为什么是三次？不是两次或四次？

**核心结论：三次握手是建立可靠连接的最小次数。**

**如果只有两次握手：**
- Client 发 SYN，可能超时重传，最终 Server 收到并建立连接
- 如果 Server 收到后直接建立连接，此时 Client 根本没收到 Server 的响应
- **问题**：Client 不知道 Server 是否收到了自己的 SYN，连接是否真的建立成功

**为什么不需要第四次？**
- 第三次握手已经让 Client 确认了 Server 能收到自己的消息
- 第四次握手（Server 确认 Client 的 ACK）理论上可以让 Client 再发一次 ACK，但这多余了
- 因为 Server 在收到第三次握手的 ACK 后，已经可以确认：
  1. Client 发送了 SYN
  2. Client 收到了 Server 的 SYN+ACK
  3. Client 愿意建立连接

**更本质的解释（序列号机制）：**
- Client 和 Server 各自生成初始序列号（ISN）
- 两次握手只能交换一端的序列号，三次握手可以交换双方的序列号
- TCP 是全双工通信，需要双方的序列号才能保证可靠传输

### SYN Flood 攻击原理与防御

**攻击原理：**
```
正常流程：Client 发 SYN → Server 返回 SYN+ACK → Client 回 ACK → 连接建立
攻击流程：Client 发大量 SYN → Server 返回 SYN+ACK → Client 不回 ACK → 连接挂起
```

攻击者发送大量 SYN 包，但不完成第三次握手，导致 Server 半连接队列被占满，无法响应正常请求。

**防御手段：**

| 防御方法 | 原理 |
|---------|------|
| SYN Cookie | 不立即分配半连接队列，通过 Cookie 验证第三次握手 |
| SYN Proxy | 防火墙/负载均衡器代为完成三次握手验证 |
| 减少 SYN+ACK 重试次数 | 加快超时释放 |
| 限制单 IP 连接数 | 通过防火墙/iptables 限制 |
| tcp_syncookies 参数 | Linux 内核参数 `net.ipv4.tcp_syncookies = 1` |

### 半连接队列与全连接队列

```
Client                    Server
  │                         │
  │──── SYN ──────────────→ │  →  半连接队列（SYN Queue）
  │                         │    (收到SYN，还未完成握手)
  │                         │
  │←── SYN+ACK ─────────── │  ←  全连接队列（Accept Queue）
  │                         │    (完成握手，等待accept)
  │──── ACK ──────────────→ │
  │                         │
  │                   accept() 从全连接队列取走
```

**相关参数：**

```bash
# 半连接队列大小（Linux）
net.ipv4.tcp_max_syn_backlog

# 全连接队列大小
# 由 listen(fd, backlog) 的 backlog 参数决定
# 但实际受限于 somaxconn 和 tcp_max_syn_backlog 的最小值

# 查看队列溢出统计
netstat -s | grep -i "overflow"
netstat -s | grep -i "reset"
```

**常见问题：**
- `accept()` 不及时调用 → 全连接队列满 → 客户端收到 RST
- 半连接队列满 → 丢弃新 SYN → 客户端重试

### 常见面试题

**Q1: 第三次握手丢失了会发生什么？**

```
Client                                          Server
  │                                               │
  │───────────  SYN=1, seq=x  ────────────────→  │
  │                                               │
  │      ←────  SYN=1, ACK=1, ack=x+1  ──────────│
  │                                               │
  │──── ACK ──────────────→ │  ← 丢失！
  │                         │
  │  （Client: ESTABLISHED）    （Server: SYN_RCVD）
  │                         │
  │  Client 认为连接已建立    Server 等待 ACK
  │  可以正常发送数据 ──────→ │  Server 回复 RST（数据带错误标志）
```

**实际情况：**
- Client 认为自己已建立连接，可以正常 send data
- Server 收到数据后会发送 RST
- 如果 Client 一直不重发 ACK，Server 会超时释放这个半连接

**Q2: 如果 Client 在第三次握手时宕机，Server 怎么处理？**

- Server 在发送 SYN+ACK 后等待 ACK
- 收到多次 ACK 超时后，释放该半连接
- Linux 默认重试次数和超时时间由 `tcp_synack_retries` 控制

---

## 四、TCP 四次挥手（深挖点2）

### 每一次挥手的状态变化与含义

```
Client                                          Server
  │                                               │
  │  ESTABLISHED  ───────  FIN=1, seq=u  ───────→ │  ESTABLISHED
  │  CLOSE_WAIT                                        │
  │      ↓                                              │
  │  FIN_WAIT_1   ←──────  ACK=1, ack=u+1  ────────── │  CLOSE_WAIT
  │  (等待Server回应)                                  │  ( Server 通知应用关闭 )
  │      ↓                                              │
  │  FIN_WAIT_2   ───────  FIN=1, seq=w  ────────→   │  LAST_ACK
  │                 ←──────  ACK=1, ack=w+1  ──────────│
  │  TIME_WAIT                                        │  CLOSED
  │  (等待 2MSL 后 CLOSED)                              │
  │      ↓                                              │
  │  CLOSED                                            │
```

**详细状态变化：**

| 步骤 | Client 状态 | Server 状态 | 说明 |
|------|------------|------------|------|
| 初始 | ESTABLISHED | ESTABLISHED | 正常通信中 |
| 第一次挥手 | ESTABLISHED → FIN_WAIT_1 | ESTABLISHED | Client 发 FIN，请求关闭 |
| 第二次挥手 | FIN_WAIT_1 → CLOSE_WAIT | ESTABLISHED → CLOSE_WAIT | Server 发 ACK，确认收到 FIN |
| 第二次挥手后 | FIN_WAIT_1 → FIN_WAIT_2 | CLOSE_WAIT | Client 等待 Server 的 FIN |
| 第三次挥手 | FIN_WAIT_2 | CLOSE_WAIT → LAST_ACK | Server 发 FIN，通知关闭 |
| 第四次挥手 | FIN_WAIT_2 → TIME_WAIT | LAST_ACK → CLOSED | Client 发 ACK，双方关闭 |
| 最终 | TIME_WAIT → CLOSED（等2MSL后） | CLOSED | Client 等待 2MSL 后关闭 |

### 为什么挥手需要四次？不是三次？

**关键点：TCP 是全双工通信，每一端都需要单独关闭。**

```
Client                              Server
  │                                   │
  │  读通道关闭 ←─── FIN (Client侧)   │
  │                                   │  ← Server 还能发送数据（写通道还开着）
  │                                   │    Server 处理完剩余数据后才关闭
  │                                   │
  │              FIN (Server侧) ───→  │
  │  写通道关闭                        │
```

**本质原因：**
- Client 发送 FIN 只是说"我要关闭客户端到 Server 的发送通道了"
- Server 可能还有数据要发送给 Client，所以不能立即发送 FIN
- 必须等 Server 处理完数据，调用 `close()` 后，才能发送 Server → Client 的 FIN

**如果变成三次挥手（ Server 收到 Client 的 FIN 后立即发 FIN）：**
- Server 可能还有残留数据在发送缓冲区中
- 这些数据还没发完就关掉了，会导致数据不完整

### TIME_WAIT 状态（深入追问点）

**TIME_WAIT 是主动关闭方在第四次挥手后进入的状态。**

```
Client                                          Server
  │                                               │
  │──── FIN, seq=u  ────────────────────────────→ │
  │←──── ACK, ack=u+1  ────────────────────────── │
  │  Client 进入 TIME_WAIT                         │
  │                                               │
  │  ←──── FIN, seq=w  ─────────────────────────── │
  │──── ACK, ack=w+1  ───────────────────────────→│
  │  Client 等待 2MSL                             │
  │  然后 → CLOSED                                 │
```

#### 为什么要等 2MSL？

**MSL（Maximum Segment Lifetime）**：一个 TCP 报文在网络中存在的最大时间，通常为 60 秒。

等待 2MSL 的原因：

1. **确保最后的 ACK 能到达 Server**
   - 如果第四次挥手的 ACK 丢失，Server 会重发 FIN
   - 2MSL 足够让 Server 收到重发的 FIN 并重传 ACK

2. **让旧连接的报文在网络中消失**
   - 防止相同端口的新连接被旧连接的延迟报文干扰
   - 2MSL 后，所有旧连接的报文都已从网络中消失

#### TIME_WAIT 过多怎么解决？

**问题：** 高并发短连接场景下，大量 TIME_WAIT 占满端口，导致无法创建新连接。

**解决方案：**

| 方法 | 说明 |
|------|------|
| `tcp_tw_reuse` | 允许将 TIME_WAIT 状态的 socket 重用于新连接（需客户端设置） |
| `tcp_tw_recycle` | 快速回收 TIME_WAIT（已被废弃，NAT 环境下有 bug） |
| `tcp_fin_timeout` | 减少 FIN_WAIT_2 的等待时间 |
| `SO_REUSEADDR` | 允许绑定处于 TIME_WAIT 的端口 |
| 设置 `socket` 的 `SO_LINGER` | 发送 RST 跳过 TIME_WAIT（不推荐，可能丢数据） |
| 改用长连接 | 减少连接创建/销毁频率 |
| 增加端口范围 | `net.ipv4.ip_local_port_range` |

```bash
# Linux 参数调整
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
```

#### SO_REUSEADDR 的作用

```c
int opt = 1;
setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
```

**作用场景：**
1. 允许绑定正处于 TIME_WAIT 状态的端口
2. 允许绑定到已有监听 socket 的相同端口（如重启服务器）
3. 允许多个 socket 绑定到相同的 UDP 端口（用于多播）

### CLOSE_WAIT 过多怎么排查？

**CLOSE_WAIT = Server 收到 Client 的 FIN，但 Server 的应用还没调用 close()。**

**常见原因：**
- 应用代码忘记调用 `close()` 或 `close()` 放在 finally 块中但异常被吞
- 数据库连接、HTTP 客户端等资源未正确释放
- 处理耗时操作时，长时间占用连接不释放

**排查方法：**

```bash
# 查看 CLOSE_WAIT 状态的连接数
netstat -an | grep CLOSE_WAIT | wc -l

# 查看具体是哪些进程/端口
netstat -anp | grep CLOSE_WAIT

# 查看详细的连接信息（包括进程名）
ss -tan state close-wait | head -20
```

**解决方案：**
1. 检查代码，确保所有 socket 在使用完毕后调用 `close()`
2. 使用连接池管理，及时归还连接
3. 设置合理的业务超时时间

### 常见面试题

**Q1: 四次挥手中出现了死等怎么办？**

死等可能有两种情况：
- **FIN_WAIT_1 死等**：发出 FIN 后没收到 ACK
  - 原因：网络问题或 Server 宕机
  - TCP 有超时重传机制，最终会放弃并进入 CLOSED
- **FIN_WAIT_2 死等**：收到 ACK 后等待 Server 的 FIN
  - 原因：Server 端忘记调用 close()
  - 可通过 `tcp_fin_timeout` 控制等待时间

**Q2: 如何理解 CLOSE_WAIT 和 TIME_WAIT？**

| 状态 | CLOSE_WAIT | TIME_WAIT |
|------|-----------|-----------|
| 发生位置 | 被动关闭方 | 主动关闭方 |
| 含义 | 收到对方 FIN，本应用未调用 close() | 双方都关闭了连接，本端等待 2MSL |
| 正常情况 | 短时间内存在 | 短时间内存在 |
| 异常情况 | 长时间存在 = 资源泄漏 | 大量存在 = 端口耗尽 |

---

## 五、TCP 可靠性保证（深挖点3）

### ARQ 协议与滑动窗口

#### ARQ（自动重传请求）

**停止等待 ARQ：**
```
Sender ────[Data 0]────→ Receiver
   ↑                       │
   │                       │
   │    ←──[ACK 0]──────── │
   │                       │
   │  （等收到 ACK 后才发下一个包）
```

- 每发一个包就停止等待 ACK
- 超时没收到 ACK 就重传
- **缺点：效率低，等待时间长**

#### 滑动窗口

**滑动窗口协议：**
```
发送窗口：
┌─────┬─────┬─────┬─────┬─────┐
│ 已发送│ 已发送│  可发  │ 可发  │ 不可发 │
│ 已确认│ 未确认│       │       │        │
│ [1-3]│ [4-6]│ [7-10]│[11-15]│[16+]   │
└─────┴─────┴─────┴─────┴─────┘
         ↑                ↑
      左边界             右边界
    (收到ACK后右移)    (窗口滑动)
```

**核心思想：** 不等 ACK 就能继续发送多个数据包，提高效率。

```
Timeline:
包1 ──────────────────────────────────→  ACK1 ──→
包2 ──────────────────────────────→  ACK2 ──→
包3 ──────────────────────→  ACK3 ──→
包4 ──────→ [重传] ──→  ...
```

**窗口大小决定了一次能发多少未确认的数据。**

### 超时重传与快速重传

#### 超时重传

```
发送 Data[1] → 等待 ACK[1]
  └─ 超时 ─→ 重发 Data[1] → 收到 ACK[1]
```

- RTO（Retransmission Timeout）：超时时间
- RTT（Round Trip Time）：往返时延
- RTO 动态计算：`RTO = RTT + 4 * RTTVAR`（RTTVAR 是 RTT 的方差）

#### 快速重传

**解决的问题：** 超时重传等待时间长

```
发送方：Data1 Data2 Data3 Data4 Data5 ...
                        ↓
接收方：只收到 Data1, Data2, Data4, Data5
                ↓
        发送 3 个重复 ACK（ack=3）
                ↓
发送方：立即重传 Data3（不等超时）
```

**触发条件：** 收到 3 个相同的重复 ACK

#### SACK（Selective Acknowledgment）

**背景：** 基础 ACK 只能确认连续收到的最大序列号，无法确认中间缺了什么。

**SACK 机制：**
```
接收方告诉发送方："我收到了 1-1000 和 2001-3000，但 1001-2000 没收到"
发送方只需重传 1001-2000 这段缺失的数据
```

```bash
# Linux 查看 SACK 是否启用
cat /proc/sys/net/ipv4/tcp_sack
```

### 拥塞控制（慢启动、拥塞避免、快恢复）

#### 拥塞窗口（cwnd）

- cwnd 是发送方维护的一个窗口，表示"一次能发多少数据而不至于网络拥塞"
- 与接收方窗口（rwnd）一起决定发送方的实际窗口大小
- 实际发送窗口 = min(cwnd, rwnd)

#### 慢启动（Slow Start）

```
cwnd 增长曲线：
cwnd
  │
  │              ╱╱╱ 拥塞避免（线性增长）
  │            ╱
  │          ╱  ↑ ssthresh
  │        ╱
  │      ╱~~ 慢启动（指数增长）
  │    ╱
  │  ╱
  │/
  └────────────────────→ 时间
```

- **指数增长**：每收到一个 ACK，cwnd += MSS
- **触发条件**：cwnd < ssthresh（慢启动阈值）
- **目的**：探测网络容量，避免一开始就发太多数据

#### 拥塞避免（Congestion Avoidance）

- **线性增长**：每 RTT 时间，cwnd += MSS（一个 MSS 的量）
- **触发条件**：cwnd >= ssthresh
- **目的**：缓慢逼近网络容量上限

#### 快恢复（Fast Recovery）

**触发条件：** 收到 3 个重复 ACK（说明有丢包，但网络还没完全堵死）

```
收到3个重复ACK → cwnd = ssthresh = cwnd/2 → cwnd += 3*MSS → 发送丢失数据 → 线性增长
```

#### 拥塞控制完整流程

```
状态转换：
                    │
              收到3个DUPLICATE ACK         发生超时
                    │                         │
                    ▼                         ▼
              ────────────────────────────→ 慢启动
                    ↑                         │
                    │                         │
                    │         cwnd >= ssthresh│
                    │         ─────────────────┘
                    │              │
                    │              ↓
                    │         拥塞避免
                    │              │
                    └──────────────┘
              (收到新的ACK，cwnd += MSS)
```

| 事件 | 动作 |
|------|------|
| 超时 | ssthresh = cwnd/2, cwnd = 1 MSS, 进入慢启动 |
| 收到 3 个重复 ACK | ssthresh = cwnd/2, cwnd = ssthresh + 3 MSS, 进入快恢复 |

### 常见面试题

**Q1: 如何理解 TCP 的滑动窗口？**

核心要点：
1. 滑动窗口是发送方和接收方协调的机制
2. 发送方有发送窗口，接收方有接收窗口
3. 窗口内的数据可以不必等待 ACK 直接发送
4. 收到 ACK 后，窗口滑动，释放已确认的数据
5. 接收方通过 ACK 告知发送方自己还能接收多少数据

**Q2: 超时时间设置为多少合适？**

- RTO 应该略大于 RTT，但不能太大（等太久）或太小（频繁误判）
- TCP 会动态计算 RTT 并更新 RTO：`RTO = RTT + 4 * RTTVAR`
- 如果 RTO 设置过小：网络稍慢就触发重传，加重拥塞
- 如果 RTO 过大：丢包后等太久，延迟高

---

## 六、进阶追问方向

### TCP 与 UDP 的实际应用场景举例

| 场景 | 协议 | 原因 |
|------|------|------|
| 浏览器网页访问 | TCP (HTTP/HTTPS) | 需要可靠传输，页面不能丢内容 |
| DNS 查询 | UDP | 速度快，丢了就重试，用户无感知 |
| 视频直播 | UDP | 宁可丢帧也不愿卡顿，实时性 > 可靠性 |
| 文件下载 | TCP | 文件不能缺损，必须完整 |
| 微信语音/视频 | UDP (私有协议) | 实时性要求高，自己实现可靠传输和纠错 |
| 在线游戏 | UDP/TCP 混合 | 走位等高频用 UDP，可靠操作用 TCP |

### 如何基于 UDP 实现可靠传输（QUIC 协议）

**QUIC = Quick UDP Internet Connections**

Google 开发的基于 UDP 的可靠传输协议，是 HTTP/3 的底层协议。

**QUIC 相比 TCP 的优势：**

| 特性 | TCP | QUIC |
|------|-----|------|
| 建立连接 | 1-RTT（三次握手）| 0-RTT（复用已有连接）或 1-RTT |
| 头部阻塞 | 有（HTTP/2 的问题）| 无（多 Stream 独立） |
| 切换网络 | 需重新建立连接 | 跳过 0-RTT，新连接复用 |
| 丢包影响 | 影响整个连接 | 只影响单个 Stream |
| 拥塞控制 | 内核实现 | 用户态实现，更灵活 |

**QUIC 实现可靠传输的方式：**
1. **序列号**：每个数据包都有序号
2. **ACK**：类似 TCP，确认收到的数据
3. **重传**：丢包后重传
4. **流控**：类似 TCP 的滑动窗口
5. **拥塞控制**：可插拔的拥塞控制算法

### socket 编程中 TCP/UDP 的区别

```c
// TCP 服务器
int server_sock = socket(AF_INET, SOCK_STREAM, 0);
bind(server_sock, ...);
listen(server_sock, backlog);           // 监听
int client_sock = accept(server_sock, ...);  // 提取已连接 socket
read(client_sock, buf, len);
write(client_sock, buf, len);
close(client_sock);
close(server_sock);

// UDP 服务器
int server_sock = socket(AF_INET, SOCK_DGRAM, 0);
bind(server_sock, ...);
recvfrom(server_sock, buf, len, &client_addr);  // 接收并获取客户端地址
sendto(server_sock, buf, len, client_addr);      // 发送响应
close(server_sock);
```

| 对比点 | TCP | UDP |
|--------|-----|-----|
| socket 类型 | SOCK_STREAM | SOCK_DGRAM |
| 是否 listen | 需要 | 不需要 |
| 是否 accept | 需要 | 不需要 |
| send/recv | send/write/recv/read（无目标地址）| sendto/recvfrom（需指定目标） |
| 边界 | 字节流，无消息边界 | 数据报，有消息边界 |
| 错误处理 | 返回 0 表示对端关闭 | 返回 -1 表示出错 |

---

## 七、面试技巧与总结

### 答题策略

1. **先答区别再深挖原理**
   - 先简洁明了说出 TCP vs UDP 的核心区别（连接方式、可靠性、首部开销等）
   - 面试官追问时再展开具体原理

2. **能画图就画图**
   - 三次握手时序图
   - 四次挥手状态转换图
   - TCP 首部结构图
   - 滑动窗口示意图

3. **结合实际项目经验**
   - "我们项目用到了 TCP 长连接连接池..."
   - "线上问题排查时遇到过 TIME_WAIT 过多..."
   - "某次事故是因为 UDP 丢包严重切到 TCP..."

4. **展示知识广度**
   - 提到 QUIC、WebSocket、HTTP/3 等扩展知识点
   - 可以对比不同语言/框架的实现差异

### 常见反问环节

| 面试官问 | 你的回应 |
|---------|---------|
| "还有其他想问的吗？" | "贵司网络层用的什么协议？长连接还是短连接？" |
| "你在这个领域有什么实践？" | "我曾经优化过一个 RPC 框架的连接池..." |
| "你了解 QUIC 吗？" | "了解，QUIC 是 Google 提出的基于 UDP 的可靠传输协议..." |

---

## 八、Q&A 环节

> 预留时间自由提问，讨论实际面试中遇到的问题