# 网站加载性能术语表

## 指标

- **DNS 时间**：将域名解析为 IP 的耗时。
- **TCP 连接时间**：建立传输层连接的耗时。
- **TLS 时间**：HTTPS 握手耗时。
- **TTFB（Time to First Byte）**：从发起请求到收到第一个响应字节的时间。它混合了网络往返、代理和服务端处理时间。
- **FCP（First Contentful Paint）**：浏览器首次绘制文本、图片等内容的时间。
- **LCP（Largest Contentful Paint）**：视口内最大内容元素完成绘制的时间，用于描述用户感知到的主要内容何时出现。
- **`load` 事件**：页面 `load` 事件触发时间。它不等于页面可交互，也不一定等于 LCP。
- **CLS（Cumulative Layout Shift）**：页面生命周期内非预期布局偏移的累计分数。
- **Long Task**：主线程上持续时间较长、可能阻塞输入和绘制的任务。

## 网络与缓存

- **Decoded Body Size / Resource Size**：移除 `Content-Encoding` 后的响应体大小，即资源解压后的逻辑大小，不包含响应头。
- **Encoded Body Size**：移除 `Content-Encoding` 之前的响应体大小；对 gzip 响应而言通常就是压缩后的正文大小，不包含响应头。
- **Transfer Size**：浏览器 Resource Timing 口径下的传输大小，通常包含响应头和 encoded body，并会反映缓存命中。它不等于 TLS 帧、TCP/IP 头和网络包层面的精确字节数。
- **冷加载**：清空或绕过浏览器缓存后的首次加载。
- **热加载**：浏览器已有可复用缓存时的再次加载。
- **内容指纹**：将内容哈希或版本写入文件名/URL，例如 `app.a1b2c3.js` 或 `client.js?rev=a1b2c3`。
- **Cache Busting**：内容变化时改变 URL，使新资源与旧缓存自然分离。
- **Private Cache**：单个用户代理（通常是浏览器）的缓存。
- **Shared Cache**：多个用户之间共享的缓存，例如代理或 CDN。
- **`no-cache`**：允许保存响应，但每次复用前必须向源站重新验证；不等于“不保存”。
- **`no-store`**：要求缓存不要保存响应。
- **`immutable`**：表示响应在新鲜期内不会变化，适合带内容指纹的静态资源。

## 代理与应用

- **gzip**：HTTP 内容编码之一，适合压缩 JavaScript、CSS、JSON、SVG 等文本资源。
- **Proxy Buffering**：反向代理先尽快读取上游响应并保存在内存或临时文件，再向客户端发送的机制。
- **SSE（Server-Sent Events）**：服务端通过一个长连接持续向浏览器推送文本事件的协议模式。
- **WebSocket**：通过 HTTP Upgrade 建立的双向长连接协议。
- **Source Map**：将压缩/编译后的 JavaScript 映射回源码的调试文件，通常体积较大。
- **关键路径**：首屏完成前必须依次完成的资源、计算与请求链路。
- **瀑布（Waterfall）**：请求之间因依赖关系形成的串行等待。
- **增量聚合**：只处理上次计算以后新增的数据，而不是每次重新扫描全部历史记录。
