## 11 Node Configuration

### 11.1 地址分配

MANET中的节点应该被分配地址序列

在每个相关网络中的节点都应被分配不同的地址

### 11.2 路由配置 

MANET网络中的具有关联网络或者主机的节点应该被配置，因此它有设在具有关联网络或者主机的接口的路由

### 11.3 数据包转发

OLSR本身不转发包。在底层操作系统中他维持着一个路由表，按照RFC1812中的转发包

## 12  Non OLSR Interfaces

一个节点可能有多个接口，其中一些不参与OLSR MANET。这些非OLSR接口可能是点对点向其他单主机，或者连向独立的网络。

为了提供向OLSR MANET加入外部路由信息的稳定性，有非MANET接口的节点应定时发送Host and Network Association(HNA)消息【该消息包含了接收者构造合适的路由表的足够信息】

### 12.1 HNA消息格式

消息发送时，Message Type 设为HNA_MESSAGE，TTL设为255，Vtime根据HNA_HOLD_TIME设置

```
       0                   1                   2                   3
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                         Network Address                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                             Netmask                           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                         Network Address                       |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                             Netmask                           |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |                              ...                              |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Network Address:关联网络的网络地址

Netmask:对应于紧接在上面的网络地址的网关

### 12.2 主机网络连接信息库

每个节点都维持着关于节点充当网关来连接主机和网络的信息，这个信息以连接元组的形式记录。

```连接元组
 "association tuples"（ A_gateway_addr, A_network_addr, A_netmask, A_time）
```

**HNA-message应该被认为是TC消息的通用版本。HNA和TC消息的发起者都向其他主机宣布“可达性”。**

TC消息和HNA消息的一个不同点在于，TC消息可能有取消之前消息的影响（如果ANSN是增加的），而HNA消息只有当过期时才会被移除。

### 12.3 HNA消息的产生

HNA消息应该在每个HNA_INTERVAL被定期转发

没有关联主机或者网络的节点不应该产生HNA消息

### 12.4 HNA消息的转发

当收到一个HNA消息的时候按照3.4的规则转发

### 12.5 HNA消息的处理

关联基础应该被更新

1. 如果发送者接口（并非最初发送者）不在对称一跳邻域内，消息应该被丢弃

2. 否则对消息内每一对元组

   2.1 如果在联系集中有 A_gateway_addr == originator address           

   ​				      A_network_addr == network address                 

   ​				      A_netmask      == netmask

   的实体存在，元组的保持时间应该设置为 A_time         =  current time + validity time

   2.2 否则生成新的元组A_gateway_addr =  originator address                  

   ​				     A_network_addr =  network address                  

    				     A_netmask      =  netmask                

   ​				     A_time         =  current time + validity time

   ### 12.6 路由表计算

   对联系集中的每个元组

   1.  如果路由表中没有 R_dest_addr     == A_network_addr/A_netmask的实体，那么新建一个实体

   2. 如果有 R_dest_addr     == A_network_addr/A_netmask               

      ​	     R_dist          >  dist to A_gateway_addr of  current association set tuple

      或者新建一个一个元组

      那么更新元组为 R_dest_addr     =  A_network_addr/A_netmask

      ​			     R_next_addr     =  the next hop on the path from the node to 

      ​									      A_gateway_addr               

      ​			    R_dist          =  dist to A_gateway_addr             

      ​			    R_next_addr and R_iface_addr MUST be set to the same   values as the tuple from the routing set with R_dest_addr               == A_gateway_addr

      

### 12.7 互用性考虑

没有实现对非OLSR接口支持的节点可以和一个实现了对非OLSR接口支持的节点共存于同一个网络。所有节点都可以转发HNA消息，但是只有实现了非OLSR接口支持的节点可以处理HNA消息。

## 13 链路层通知



