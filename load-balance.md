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

* **LVS-NAT**

* **LVS-DR**

* **LVS-TUN**

##### 参考资料

使用LVS实现负载均衡原理及安装配置详解：[https://www.cnblogs.com/liwei0526vip/p/6370103.html](https://www.cnblogs.com/liwei0526vip/p/6370103.html)

