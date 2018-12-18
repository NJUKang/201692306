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

OLSR被设计成不从链路层强加或期望任何特定信息。如果来自描述链路中断的链路层的信息可用，节点将会用到链路层感知。

如果描述到相邻节点的连接的链路层信息可用(即，例如由于缺少链路层确认而失去连接)，则除了来自HELLO消息的信息之外，还使用该信息来维护邻居信息库和MPR选择器集。

HELLO消息产生应该考虑下面的情况：

1. 如果L_LOST_LINK_time没有过期，链路被用链接类型LOST_LINK进行通知。此外，在关联相邻元组的更新中，它不被认为是对称链接。
2. 如果和对称邻域或者不对称接口的链路中断，对应的链路元组应该设置为 L_LOST_LINK_time&&L_time  = 当前时间+ NEIGHB_HOLD_TIME。
3. 如果链路丢失，则采取8.5措施

### 13.1 互用性考虑

链路层通知为节点提供了附加准则，通过该准则，节点可以确定到相邻节点的链路是否丢失。一旦检测到链路丢失，就按照本说明书前几节中描述的规定进行公告。

## 14 链接滞后

链路感知应该面对突发损失和节点的瞬时连接具有鲁棒性（_鲁棒是Robust的音译，也就是健壮和强壮的意思。它是在异常和危险情况下系统生存的关键。比如说，计算机[软件](https://baike.baidu.com/item/%E8%BD%AF%E4%BB%B6)在输入错误、磁盘故障、网络过载或有意攻击情况下，能否不死机、不崩溃，就是该软件的鲁棒性。所谓“鲁棒性”，是指控制系统在一定（结构，大小）的参数[摄动](https://baike.baidu.com/item/%E6%91%84%E5%8A%A8/4777855)下，维持其它某些性能的特性。_）

### 14.1 本地链接集

```
 L_link_pending ：Boolean 判断链接挂起（链接未被认为建立）
 L_link_quality：描述链路质量的0到1之间的无量纲数（两个具有相同量纲的物理量的比值成为一个无量纲的量，习惯上称为无量纲量或无量纲数，如圆周率（π）、自然常数（e）、角度（rad）、黄金分割率（φ）和相对分子质量（Mr）等。）
  L_LOST_LINK_time：挂起多久后被认为丢失
```

### 14.2 HELLO消息的产生

1. 如果L_LOST_LINK_time没有过期，链路被宣布为 LOST_LINK的链路类型
2. 如果L_LOST_LINK_time过期，且 L_link_pending为真，链路根本不用被宣布
3. 如果L_LOST_LINK_time过期，且 L_link_pending为假，链路像第六节（HELLO消息格式和生成）那么被宣布。

如果在链路集里的每个元组都符合 L_LOST_LINK_time过期，L_link_pending为假，L_SYM_time没过期的节点有对称链路。对称链路用来计算邻居节点的N_status，以及在路由表计算的第一步做对称邻域，计算MPR集。

### 14.3 滞后策略

链接可能是糟糕的，即不时地让HELLO通过，只是在之后立即消失。

```
 L_link_quality被阈值 HYST_THRESHOLD_HIGH, HYST_THRESHOLD_LOW，属于（0，1）比较，HYST_THRESHOLD_HIGH >=HYST_THRESHOLD_LOW.
```

```
L_link_quality > HYST_THRESHOLD_HIGH:

               L_link_pending   = false

               L_LOST_LINK_time = current time - 1 (expired)
```

```
if L_link_quality < HYST_THRESHOLD_LOW:

               L_link_pending   = true

               L_LOST_LINK_time = min (L_time, current time +
               NEIGHB_HOLD_TIME)
```

```
 if HYST_THRESHOLD_LOW <= L_link_quality
                                           <= HYST_THRESHOLD_HIGH:

               L_link_pending and L_LOST_LINK_time remain unchanged.
```

链路质量的估计维持和存储在 L_link_quality。如果所接收的消息上的信号/噪声电平的一些测量是可用的(例如，作为链路层通知)，那么它可以在归一化之后用作估计。

如果从链路层没有这样的信号/噪声电平，用算法来填充 L_link_quality。

```
对于第一次NI被I听到， L_link_quality = HYST_SCALING（被设置为0-1之间的换算因子）
L_link_pending is set to true and L_LOST_LINK_time to current time -1
每次收到NI发出的OLSR包，稳定规则：L_link_quality = (1-HYST_SCALING)*L_link_quality
                           + HYST_SCALING.
如果NI的发出的OLSR包被I丢掉，不稳定规则： L_link_quality = (1-HYST_SCALING)*L_link_quality.
```

```
OLSR包的丢失是通过跟踪每个接口上丢失的分组序列号和来自节点的“长时间沉默”来检测的。因此，可以检测到“长时间的静默：如果在接口NI的HLLO发射间隔期间在接口I上没有从接口NI接收到OLSR分组(从NI接收的最后一个HELLO消息中的Htime字段计算)，则检测到OLSR分组的丢失。
```

### 14.3 互用性考虑

链路滞后决定了节点接受或拒绝到相邻节点的链路的标准。网络中的节点可以根据它们所通信的媒体的性质具有不同的标准。

## 15 冗余拓扑信息

为了向拓扑信息库提供冗余，节点可能广播的链路集包含到相邻节点的链路。它们不在节点的MPR选择器集中。宣布的链接集可能包含到节点的整个邻居集的链接。任何节点必须在其TC消息中通告的最小链接集是到其MPR选择器的链接。广播的链接集可以根据以下规则基于称为TC_REDUNDANCY参数的本地参数构建。

### 15.1  TC_REDUNDANCY参数

```和
TC_REDUNDANCY为0，节点的广播链路集受限于MPR选择者集
TC_REDUNDANCY为1，节点的广播链路集是MPR选择者集和MPR的合集
TC_REDUNDANCY为2，节点的广播链路集是整个邻域链路集
WILL_NEVER的节点 TC_REDUNDANCY也为0
```

### 15.2 互用性考虑

网络中的节点发出的声明链路集（advertised link set，必须包括它的MPR选择者集）的TC消息

本节中的规定指定如何声明附加信息，如通过TC_REDUNDANCY参数指定的。TC_REDUNDANCY=0表示声明的信息正好对应于MPR选择器集，与第9节相同。TC_REDUNDANCY的其他值指定要声明的附加信息，即MPR选择器集的内容总是声明的。因此，具有不同TC_REDUNDANCY值的节点可以共存于网络中：根据第3节，控制消息由所有节点承载，并且所有节点将至少接收构建路由所需的链路状态信息，如第10节所述。

## 16 MPR冗余

表明一个节点选择冗余MPRs的能力，节点应该尽可能小的选择MPR集来减少协议负载。选择MPR的标准是，所有严格的2跳节点必须通过至少一个MPR节点可到达。

虽然一般来说，最小MPR集提供的开销最小，但在某些情况下，开销可以折衷为其他好处。例如，如果节点观察到由移动性引起的其邻居信息库的许多变化，而另外保持低MPR覆盖，则节点可以决定增加其MPR覆盖。

### 16.1 MPR_COVERAGE参数

```
MPR_COVERAGE定义了MPR节点有多少任意严格2跳节点应该被覆盖
MPR_COVERAGE=1表明协议的负载被保持在最小值，导致MPR选择
MPR_COVERAGE=m保证如果可能的话节点选择它的MPR集因此对一个接口，通过那个接口上至少m个MPR节点，所有的严格2跳节点都可达。
```

为节点设置的MPR是为每个接口找到的MPR集合的并集。

```
MPR_COVERAGE可以在本地调整而不会影响协议的一致性。例如，网络中的节点可以使用不同的MPR_COVERAGE值进行操作。
```

### 16.2 MPR计算

MPR选择启发法：覆盖不良的节点是N2中被小于MPR_COVE覆盖的节点。

```
用于选择MPR的拟议启发式如下：
1. 从由N的所有成员组成的MPR集开始
          意愿等于WILL_ALWAYS
2. 对于N中的所有节点，计算D（y），其中y是N的成员。
3. 选择N中覆盖N2中覆盖较差节点的节点作为MPR。然后，节点从N2移除，用于剩余的计算。
4. N2中存在MPR集中至少MPR_COVERAGE节点未覆盖的节点：
	4.1对N中每个节点计算可达性。N2中尚未被MPR集中的至少MPR_COVERAGE节点覆盖的节点的数量，并且这些节点可以通过这个1跳邻居到达
	4.2在具有非零可达性的N个节点中选择具有最高意愿的节点作为MPR。在多选的情况下，选择为N2中最大数量的节点提供可达性的节点。在多个节点提供相同数量的可达性的情况下，选择D(y)较大的节点作为MPR。从N2中删除MPR集中的MPR_COVERAGE节点现在覆盖的节点。
5. 节点的MPR集是从每个接口的MPR集的并集中生成的。作为优化，按照N_willingness递增的顺序处理MPR集中的每个节点y。如果N2中的所有节点仍然被MPR集中的至少MPR_COVERAGE节点覆盖（不包括节点y），并且如果节点y的N_willingness小于WILL_ALWAYS，则节点y可能从MPR集中删除。
```

### 16.3 互用性考虑

根据第8.3节，节点的MPR集必须由节点以这样的方式计算，即通过MPR集中的邻居，它能够到达所有对称的严格的2跳邻居。对于MPR_COVERAGE>0的所有值，这是通过本节中的试探法实现的。MPR_COVERAGE是每个节点的本地参数。设置此参数仅影响部分网络中的冗余量。

```
有不同MPR_COVERAGE的节点可以共存于同一个网络
```

## 17 IPv6考虑

协议同样适用于IPv6，只需要把最小包大小和消息长度相应调整。

## 18 常量的建议值

