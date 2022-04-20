# Sockert SO_REUSEPORT相关问题

> 1. SO_REUSEPORT选项如何实现端口复用？
> 2. 在端口复用下，负载均衡策略是什么？
> 3. SO_REUSEPORT与SO_REUSEADDR的区别？
> 4. 各个操作系统对SO_REUSEPORT的实现有何不同？是否所有系统都支持？

## 1. SO_REUSEPORT选项如何实现端口复用？

首先，UDP socket在`bind()`后（TCP socket的`listen()`也一样）会把该socket加入reuseport group中。操作系统的BPF（Berkeley Packet Filter）程序通过返回一个index来指定reuseport group中的某个socket接收数据包。

## 2. 在端口复用下，负载均衡策略是什么？

tcp连接或udp数据报到达服务端socket后，使用hash策略（基于四元组，即双方的ip和端口）进行分配指定的socket。

## 3. SO_REUSEPORT与SO_REUSEADDR的区别？

- TCP socket在`close()`后会有一段`TIME_WAIT`时间（为了等待网络中的数据传输完毕），此时如果要绑定当前socket会失败，加了SO_REUSEADDR选项后，就可以绑定成功。一般用于防止服务器重启时之前的端口还未释放。
- SO_REUSEPORT可以让多个socket绑定一个端口。

## 4. 各个操作系统对SO_REUSEPORT的实现有何不同？是否所有系统都支持？

- macOS和其他的BSDs会返回最新的一个socket，不会进行load balance
- 最近的Linux实现会在内核进行load balance