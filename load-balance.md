### Load Balance

load balance基于TCP／IP 或者OSI网络模型的各个层来做均衡转发

1. 软件类型（dns、nginx、LVS即linux virtual server等\)，通常能达到10w QPS的级别

2. 硬件类型（F5等），可以达到百万QPS的级别

Nginx主要是在应用层进行均衡服务，作为proxy代理流入的user请求，然后向后台server请求数据信息，然后返回给user；硬件类型均衡路由器它基于路由器的功能在IP层将流入的用户请求转发至后台server；DNS均衡器是采用用户请求域名解析服务的方式来实现均衡，但是因为域名解析缓存的问题更新生效比较慢。

