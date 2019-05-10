### Load Balance

load balance基于TCP／IP 或者OSI网络模型的各个层来做均衡转发

1. 软件类型（dns、nginx、LVS即linux virtual server等\)，通常能达到10w QPS的级别

2. 硬件类型（F5等），可以达到百万QPS的级别

Nginx主要是在应用层进行均衡服务，作为proxy代理流入的user请求，然后向后台server请求数据信息，然后返回给user；DNS均衡器是采用用户请求域名解析服务的方式来实现均衡，但是因为域名解析缓存的问题更新生效比较慢；LVS是通过将linux主机配置实现路由器功能来路由流量，主要针对链路层、网络-IP层来实现路由均衡功能，主要有LVS-NAT（网络-IP层）、LVS-DR（链路层）、LVS-TUN（链路层）三种。

硬件类型均衡路由器它基于路由器的功能在IP层将流入的用户请求转发至后台server。

#### LVS负载均衡

LVS是 Linux Virtual Server 的简称，也就是Linux虚拟服务器。这是一个由章文嵩博士发起的一个开源项目，它的官方网是[http://www.linuxvirtualserver.org](http://www.linuxvirtualserver.org/)现在 LVS 已经是 Linux 内核标准的一部分。使用 LVS 可以达到的技术目标是：通过 LVS 达到的负载均衡技术和 Linux 操作系统实现一个高性能高可用的 Linux 服务器集群，它具有良好的可靠性、可扩展性和可操作性。从而以低廉的成本实现最优的性能。LVS 是一个实现负载均衡集群的开源软件项目，LVS架构从逻辑上可分为调度层、Server集群层和共享存储。

* **LVS术语**

  DS：Director Server。指的是前端负载均衡器节点。  
  RS：Real Server。后端真实的工作服务器。  
  VIP：向外部直接面向用户请求，作为用户请求的目标的IP地址。  
  DIP：Director Server IP，主要用于和内部主机通讯的IP地址。  
  RIP：Real Server IP，后端服务器的IP地址。  
  CIP：Client IP，访问客户端的IP地址。

* **LVS-NAT**![](/assets/nat.png)

请求处理流程如下

```
(a) 当用户请求到达Director Server，此时请求的数据报文会先到内核空间的PREROUTING链。 此时报文的源IP为CIP，目标IP为VIP
(b) PREROUTING检查发现数据包的目标IP是本机，将数据包送至INPUT链
(c) IPVS比对数据包请求的服务是否为集群服务，若是，修改数据包的目标IP地址为后端服务器IP，然后将数据包发至POSTROUTING链。 此时报文的源IP为CIP，目标IP为RIP
(d) POSTROUTING链通过选路，将数据包发送给Real Server
(e) Real Server比对发现目标为自己的IP，开始构建响应报文发回给Director Server。 此时报文的源IP为RIP，目标IP为CIP
(f) Director Server在响应客户端前，此时会将源IP地址修改为自己的VIP地址，然后响应给客户端。 此时报文的源IP为VIP，目标IP为CIP
```

该模型的特点是

```
1. RS应该使用私有地址，RS的网关必须指向DIP
2. DIP和RIP必须在同一个网段内
3. 请求和响应报文都需要经过Director Server，高负载场景中，Director Server易成为性能瓶颈
4. 支持端口映射
5. RS可以使用任意操作系统
缺陷：对Director Server压力会比较大，请求和响应都需经过director server
```

* **LVS-DR**![](/assets/dr.png)

请求处理流程

```
(a) 当用户请求到达Director Server，此时请求的数据报文会先到内核空间的PREROUTING链。 此时报文的源IP为CIP，目标IP为VIP
(b) PREROUTING检查发现数据包的目标IP是本机，将数据包送至INPUT链
(c) IPVS比对数据包请求的服务是否为集群服务，若是，将请求报文中的源MAC地址修改为DIP的MAC地址，将目标MAC地址修改RIP的MAC地址，然后将数据包发至POSTROUTING链。 此时的源IP和目的IP均未修改，仅修改了源MAC地址为DIP的MAC地址，目标MAC地址为RIP的MAC地址 
(d) 由于DS和RS在同一个网络中，所以是通过二层来传输。POSTROUTING链检查目标MAC地址为RIP的MAC地址，那么此时数据包将会发至Real Server。
(e) RS发现请求报文的MAC地址是自己的MAC地址，就接收此报文。处理完成之后，将响应报文通过lo接口传送给eth0网卡然后向外发出。 此时的源IP地址为VIP，目标IP为CIP 
(f) 响应报文最终送达至客户端
```

模型特点

```
1. 保证前端路由将目标地址为VIP报文统统发给Director Server，而不是RS
2. RS可以使用私有地址；也可以是公网地址，如果使用公网地址，此时可以通过互联网对RIP进行直接访问
3. RS跟Director Server必须在同一个物理网络中
4. 所有的请求报文经由Director Server，但响应报文必须不能进过Director Server
5. 不支持地址转换，也不支持端口映射
6. RS可以是大多数常见的操作系统
7. RS的网关绝不允许指向DIP(因为我们不允许他经过director)
8. RS上的lo接口配置VIP的IP地址
缺陷：RS和DS必须在同一机房中
```

> * 设置RIP之一的网络接口地址为VIP的原因是RS的网络数据处理模型中在IP层解析获取的数据时会判断目标IP是否为本机IP，如果不是将会丢弃包
> * RS与DS必须在一个物理网络中是因为链路层在传递包时是依据MAC地址，而依据MAC地址传递数据包只能在局域网内传递，不能跨网传递；如果要传递跨网的数据包，需要先根据IP地址路由至下一个局域网依次传递至目的地址
> * TODO 针对模型特点第7点原因是？

为了保证模型特点1需要做以下处理

```
1. 在前端路由器做静态地址路由绑定，将对于VIP的地址仅路由到Director Server
2. 存在问题：用户未必有路由操作权限，因为有可能是运营商提供的，所以这个方法未必实用
3. arptables：在arp的层次上实现在ARP解析时做防火墙规则，过滤RS响应ARP请求。这是由iptables提供的
4. 修改RS上内核参数（arp_ignore和arp_announce）将RS上的VIP配置在lo接口的别名上，并限制其不能响应对VIP地址解析请求。
```



* **LVS-TUN**



##### 参考资料

使用LVS实现负载均衡原理及安装配置详解：[https://www.cnblogs.com/liwei0526vip/p/6370103.html](https://www.cnblogs.com/liwei0526vip/p/6370103.html)

