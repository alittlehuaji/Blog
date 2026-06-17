在 BitTorrent 客户端中，经常可以在“用户 / Peers”列表里看到一串简短的字母标记，例如：

```text
D U H E P
```

这些字母就是 Peer Flags。它们用来快速描述当前客户端与某个 peer 之间的连接状态、上传下载关系、来源、加密方式以及传输协议。这些 Flag 不是互斥的，一个 peer 可以同时拥有多个 Flag

## 速查表

| Flag | 含义 | 简单解释 |
|---|---|---|
| `d` | interested local + choked peer | 我想下载，但对方不让我下 |
| `D` | interested local + unchoked peer | 我正在或可以从对方下载 |
| `u` | interested peer + choked local | 对方想下载，但我不让它下 |
| `U` | interested peer + unchoked local | 我正在或可以向对方上传 |
| `O` | optimistic unchoke | 乐观解锁 |
| `S` | snubbed | peer 长时间不响应或不给有效数据 |
| `I` | incoming connection | 对方主动连进来 |
| `K` | not interested local + unchoked peer | 对方允许我下，但我不需要 |
| `?` | not interested peer + unchoked local | 我允许对方下，但对方不需要 |
| `X` | PEX | 通过 Peer Exchange 获得 |
| `H` | DHT | 通过 DHT 获得 |
| `L` | LSD | 通过局域网发现 |
| `h` | hole punching | 与 NAT 打洞连接有关 |
| `E` | encrypted traffic | 流量加密 |
| `e` | encrypted handshake | 握手加密 |
| `P` | uTP | 使用 uTP 协议 |

## 核心概念

在解释具体 Flag 之前，需要先理解 BitTorrent 中两个非常重要的状态：

### 1. Interested：感兴趣

`interested` 表示一方想要另一方拥有的数据块

例如：

- 你的客户端发现某个 peer 有你缺少的 piece，就会对它表示 `interested`
- 某个 peer 发现你有它缺少的数据，也会对你表示 `interested`

### 2. Choked：被阻塞

`choked` 表示一方暂时不允许另一方下载

即使你对某个 peer 感兴趣，也不代表马上能下载。对方必须 `unchoke` 你，你才能真正从它那里接收数据

简单说：

- `choked`：不允许下载
- `unchoked`：允许下载

所以 BitTorrent 里的下载/上传关系，通常由两组状态共同决定：

- 我是否对 peer 感兴趣
- peer 是否允许我下载
- peer 是否对我感兴趣
- 我是否允许 peer 下载

## Flags

### 上传与下载相关

#### `d`：本地想下载，但 peer 不允许

含义：

```text
本地 interested + peer choked
```

你的客户端想从这个 peer 下载数据，但对方暂时不允许你下载

#### `D`：正在下载 / 可以下载

含义：

```text
本地 interested + peer unchoked
```

你的客户端想从这个 peer 下载，并且对方允许你下载

#### `u`：peer 想下载，但本地不允许

含义：

```text
peer interested + local choked
```

对方想从你的客户端下载数据，但你的客户端暂时不允许它下载

#### `U`：正在上传 / 可以上传

含义：

```text
peer interested + local unchoked
```

对方想从你的客户端下载数据，并且你的客户端允许它下载

### 特殊连接状态

#### `O`：Optimistic Unchoke，乐观解锁

`O` 表示 optimistic unchoke

BitTorrent 客户端不会永远只服务当前表现最好的 peer。它会定期随机或半随机地给某些 peer 一个上传机会，这叫“乐观解锁”

作用：

- 发现潜在的高速 peer
- 避免新 peer 永远得不到机会
- 改善 swarm 中的数据流动

简单点说：“我临时给这个 peer 一个上传机会，康康它表现如何”

#### `S`：Snubbed，被冷落 / 长时间无响应

`S` 表示 peer 被标记为 snubbed

通常意味着这个 peer 长时间没有向你发送有效数据，或者响应很慢。客户端可能会认为它暂时对下载没有帮助

简单点说：“这个 peer 可能不太响应，或者长时间不给我数据”

#### `I`：Incoming Connection，传入连接

`I` 表示此连接是对方主动连到你的客户端

如果你能收到传入连接，往往说明你的监听端口、NAT 或防火墙配置较好，其他 peer 可以主动连接到你。

#### `K`：peer 允许你下载，但你不感兴趣

含义：

```text
本地 not interested + peer unchoked
```

简单点说：“对方允许我下载，但我当前不需要它的数据”

常见原因包括：

- 它拥有的数据你已经有了
- 当前选择的文件或 piece 不需要从它那里获取
- 下载进度或优先级导致暂时不需要它

#### `?`：你允许 peer 下载，但 peer 不感兴趣

含义：

```text
peer not interested + local unchoked
```

简单点说：“我允许对方下载，但对方当前不需要我的数据”

这不一定是异常，只表示对方暂时没有从你这里下载的需求

### 来源相关

#### `X`：来自 PEX

`X` 表示这个 peer 是通过 PEX 获得的

PEX 是 Peer Exchange 的缩写，意思是“用户交换”。当你连接到一些 peer 后，它们可以告诉你它们还知道哪些 peer

简单点说：“这个 peer 是别人介绍给我的”

PEX 的好处是可以在 tracker 和 DHT 之外，进一步扩展 peer 来源

#### `H`：来自 DHT

`H` 表示这个 peer 是通过 DHT 找到的。

DHT 是 Distributed Hash Table，分布式哈希表。它允许 BitTorrent 客户端在没有 tracker 的情况下寻找同一个 torrent 的 peer

简单点说：“这个 peer 是我通过 DHT 网络找到的”

#### `L`：来自 LSD，局域网发现

`L` 表示这个 peer 来自 LSD。

LSD 是 Local Service Discovery，本地服务发现。它用于发现同一局域网内的 BitTorrent 客户端

形象点说：“这个 peer 是在局域网里发现的”

如果同一个局域网内有其他人在下载或做种，LSD 可以让客户端更快找到对方，而且局域网传输速度通常更高

#### `h`：Hole Punching，打洞连接

`h` 通常表示这个 peer 与 NAT hole punching 有关

NAT hole punching 是一种穿透 NAT 的技术，用于帮助两个都在 NAT 或防火墙后面的客户端建立直接连接

简单点说：“这个 peer 是通过打洞方式连接或发现的”

需要注意大小写：

- `H`：来自 DHT
- `h`：hole punching

这两个含义完全不同

### 加密与协议相关

#### `E`：Encrypted Traffic，流量加密

`E` 表示连接使用了协议加密，通常指**全部通信流量**被加密

简单点说：“这个 peer 的通信流量是加密的”

这类加密主要是 BitTorrent 协议加密，不等同于 HTTPS 那种端到端安全模型，但可以减少明文协议特征

#### `e`：Encrypted Handshake，握手加密

`e` 表示连接**握手阶段**使用了加密

简单点说：“这个 peer 的握手过程是加密的”

它和 `E` 的区别是：

- `E` 更偏向**整体流量**加密
- `e` 更偏向**握手阶段**加密

#### `P`：uTP 连接

`P` 表示这个 peer 使用 uTP 协议。

uTP 是 Micro Transport Protocol，是 BitTorrent 常用的一种基于 UDP 的传输协议。它和传统 TCP 连接不同，更强调拥塞控制和延迟控制

简单点说：“这个 peer 连接使用 uTP”

uTP 的设计目标是：

- 减少对网页浏览、视频通话等其他网络活动的影响
- 在网络拥塞时主动降低传输强度
- 更适合大量 peer 并发连接

## 快速判断 peer 的状态

看到一串 Flags 时，可以按这个顺序理解：

1. 先看有没有 `D` 或 `d`

   判断你能不能从它那里下载。

   - 有 `D`：它正在帮你下载
   - 有 `d`：你想下，但它暂时不给你

2. 再看有没有 `U` 或 `u`

   判断你有没有在给它上传。

   - 有 `U`：你正在给它上传
   - 有 `u`：它想下，但你暂时不给它

3. 来源

   - `H`：DHT
   - `X`：PEX
   - `L`：局域网
   - `h`：打洞相关

4. 连接能力

   - `I`：对方能主动连进来
   - `P`：uTP
   - `E` ：流量加密
   - `e`：握手加密

5. 特殊状态

   - `S`：表现差或长时间无响应
   - `O`：乐观解锁

----

参考链接：

<https://github.com/qbittorrent/qBittorrent/blob/master/src/lang/qbittorrent_zh_CN.ts#L8150>

<https://github.com/qbittorrent/qBittorrent/blob/master/src/base/bittorrent/peerinfo.cpp>

<https://github.com/qbittorrent/qBittorrent/wiki/Frequently-Asked-Questions#what-do-all-those-flags-in-the-flags-column-mean>
