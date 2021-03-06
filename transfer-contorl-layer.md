
---

#### 运行在该层的协议

```
TCP UDP
```

---

#### 协议格式

* TCP

* UDP

```
注意：UDP允许系统运行多个程序侦听同一个端口，但是需要使用端口的程序设置socket REUSEADDR选项和系统支持多播
```

---

#### TCP流量控制

* TCP流量控制的方法流程为  慢启动 --&gt; 拥塞避免

* 正常情况下拥塞避免算法为（以下2个要求必须同时满足）

```
  1. 在收到1个ack时，cwnd = cwnd + 1 ／ cwnd
  2. 在1个RTT内，cwnd最多增加1
```

* 发生RTO超时，重传超时数据，采用以下拥塞避免算法

```
1. ssthresh = cwnd ／ 2
2. cwnd = 1
3. 进入慢启动流程
```

* 发生重复ACK时，重传重复ack数据，采用以下拥塞避免算法

```
1. ssthresh = cwnd / 2
2. cwnd = ssthresh + 3
3. 凡是收到一个非更新过的ACK，cwnd = cwnd + 1
4. 收到更新过的ACK，cwnd = ssthresh，进入拥塞避免算法
```

* 拥塞避免算法优化方案

（1）SACK

在发生重复ACK时，接收方并不知道哪些包丢失或者迟到，利用TCP OPTION字段可以发送SACK信息来告诉发送方已经接收到的那些片段，然后发送方可以根据这些信息来确定哪些片段丢失![](/assets/tcp_sack_example-1024x577.jpg)

基于SACK有一个D-SACK概念，依据该概念可以告诉发送方有哪些数据被重复收到了

```
D-SACK使用了SACK的第一个段来做标志，

1. 如果SACK的第一个段的范围被ACK所覆盖，那么就是D-SACK
2. 如果SACK的第一个段的范围被SACK的第二个段覆盖，那么就是D-SACK
```

#### 其它拥塞避免算法

Reno

New Reno

#### RTT、RTO计算算法

karn

