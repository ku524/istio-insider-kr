# 源码设计



## 设计模式与行话

时常有人问，如何快速系统地学习一个领域的知识。现代社会，大部分知识是可以公开访问到的。但让人烦恼的是，文档都有了，源码也有了，文档中的每一个单词都懂，源码中的每一小段好像也懂。但要整体把握一个系统的机理，还是相当困难。哪怕把范畴缩小到学习一个软件系统，或者直接说，是 Envoy。

Envoy 是开源软件，文档也写得相当细致，这在开源项目中是少有的细致。C++ 14 的写法比起 C 和 C++ 1998 已经很平易近人了。可读性已经比较接近 Java，并且不像看 Linux 内核一样那么多缩写术语。但和任何其它软件源码一样，Envoy 有自己的设计模式和行话术语，要读懂其中的行话，就会事半功倍。



### Callback回调设计模式

Gang of Four (GoF) 的 《*Design Patterns*》 中，并没有一个设计模式叫 Callback 的。`Callback 设计模式`其实是 [`Observer pattern(观察者模式)`](https://en.wikipedia.org/wiki/Observer_pattern) 的一个变体或应用。



> 观察者模式解决了以下问题：
>
> - 解藕对象之间的一对多依赖关系，避免对象紧密耦合。
> - 当一个对象更改状态时，自动更新不限数量的依赖对象。
> - 一个对象可以通知多个其他对象。

以上是  [Wkipedia](https://en.wikipedia.org/wiki/Observer_pattern) 关于`Observer pattern(观察者模式)`  的解释。我觉得本质上，观察者模式 更多是一种运用 `agnosticism(不可知论) `让对象依赖反转的设计模式。

在 Envoy 中，它的主要作用是让各子系统在设计时可以互相独立。减少直接（起码是编译期）的依赖。Envoy 大量使用了这种 `Callback 设计模式`，并有自己的命名规范。这是要读懂 Envoy 代码要过的门槛。



![*OOP 子系统回调设计模式*](./oop-subsystem-callback.drawio.svg)



上面有两个子系统，`Network::TransportSocket`  与  `Network::Connection` 。你可以这么想，如果 每个使用 `Network::TransportSocket`  的子系统都要 `Network::TransportSocket` 去依赖通知，结果是一个循环依赖。而 Callback 的设计模式，避免了这个问题。



## 子系统

Envoy 为了模块化，设计出很多在编译期相互独立的子系统。而这些子系统通过 显式依赖和 Callback 胶水等的依赖反转方法，实现互动。一般，每个子系统都有自己的 C++ namespace(名几个相关的子系统共用一个 namespace)。以下列出主要的子系统：



- `Buffer`  - 缓存块
- `Api` - 操作系统调用
- `Config` - XDS 等配置
- `Event` - 事件驱动
- `Http` - HTTP 相关
  - `Http::ConnectionPool` - HTTP 连接池相关
  - `Http1` - HTTP/1.1 相关
  - `Http2` - HTTP/2 相关
- `Network` - IP/TCP/DNS/Socket  层，即 OSI 的 L3/L4 层相关。包括 `Envoy Network Filter` 、`Listener`
  - `Network::Address` - IP 地址相关

- `Server` - Envoy 作为服务 Daemon  lifecycle 相关的实现
- `Stats` - 监控指标相关
- `Tcp` - TCP 与连接池相关
- `Upstream` - Upstream 相关的负载均衡、健康检查等等
- ``



其中，大部分子系统均有自己的功能 `C++ Class/Interface`，如 `Envoy Network Filter`  的 `ReadFilter`，以及其配套的 Callback 接口定义，如 `ReadFilterCallbacks` 。





