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

## 18常量的建议值

### 18.1 设定发射间隔和保持时间

```
常数C应该设置为建议值。为了实现互操作性，C必须在所有节点上都相同。
C = 1/16秒（等于0.0625秒）

   C是“有效时间”计算的缩放因子（“Vtime”）和邮件标题中的“Htime”字段
```

### 18.2 发射间隔

```
 HELLO_INTERVAL = 2秒

          REFRESH_INTERVAL = 2秒

          TC_INTERVAL = 5秒

          MID_INTERVAL = TC_INTERVAL

          HNA_INTERVAL = TC_INTERVAL
```

### 18.3 保持时间

```
 NEIGHB_HOLD_TIME = 3 x REFRESH_INTERVAL

 TOP_HOLD_TIME = 3 x TC_INTERVAL

 DUP_HOLD_TIME = 30秒

 MID_HOLD_TIME = 3 x MID_INTERVAL

 HNA_HOLD_TIME = 3 x HNA_INTERVAL
```

```
  value = C *（1 + a / 16）* 2 ^ b [以秒为单位]
鉴于上述持有时间之一，计算方法
   数字T（秒）的尾数/指数表示是
   以下：

     - 找到最大的整数'b'，使得：T / C> = 2 ^ b

     - 计算表达式16 *（T /（C *（2 ^ b）） -  1），其可能不是a
          整数，并将其四舍五入。这导致'a'的值

     - 如果'a'等于16：将'b'加1，并将'a'设置为0

     - 现在，'a'和'b'应该是0到15之间的整数，并且
          字段将是一个保持值a * 16 + b的字节

   例如，对于2秒，6秒，15秒和30的值
   分别为秒，a和b为：（a = 0，b = 5），（a = 8，b = 6），
   （a = 14，b = 7）和（a = 14，b = 8）。
```

### 18.4 消息类型

```
          HELLO_MESSAGE = 1

          TC_MESSAGE = 2

          MID_MESSAGE = 3

          HNA_MESSAGE = 4
```

### 18.5 链接类型

```
          UNSPEC_LINK = 0

          ASYM_LINK = 1

          SYM_LINK = 2

          LOST_LINK = 3
```

### 18.6 邻居类型

```
          NOT_NEIGH = 0

          SYM_NEIGH = 1

          MPR_NEIGH = 2
```

### 18.7 链接滞后

```
          HYST_THRESHOLD_HIGH = 0.8

          HYST_THRESHOLD_LOW = 0.3

          HYST_SCALING = 0.5
```

### 18.8 意愿

```
          WILL_NEVER = 0

          WILL_LOW = 1

          WILL_DEFAULT = 3

          WILL_HIGH = 6

          WILL_ALWAYS = 7
```

### 18.9 常量

```
          TC_REDUNDANCY = 0


          MPR COVERAGE = 1


          MAXJITTER = HELLO_INTERVAL / 4
```

## 19 序号

丢弃无序接收的消息

为了避免环绕（序列号从最大值开始递增），引入

```
MAXVALUE指定最大可能序号的值。
序列号S1被称为“大于”序列
   数字S2如果：

          S1> S2和S1  -  S2 <= MAXVALUE / 2 OR

          S2> S1和S2  -  S1> MAXVALUE / 2

```

## 20 安全考虑因素

OLSR没有指定任何特殊的安全措施

### 20.1 保密

作为一种主动协议，OLSR周期性地传播拓扑信息。因此，如果在无保护无线网络中使用，则网络拓扑将向收听OLSR控制消息的任何人展示。

在网络拓扑的机密性很重要的情况下，可以应用常规加密技术，例如交换由PGP加密或由某些共享密钥加密的OLSR控制通信量消息，以确保控制通信量只能被授权这样做的人读取和解释。

### 20.2 健全性

在OLSR中，每个节点通过传输HELLO消息和TC消息向网络中注入拓扑信息。如果某些节点由于某种原因，恶意或故障，注入无效的控制流量，可能会危及网络的完整性。因此，建议进行消息认证。

可能发生的恶意行为

1. 节点产生TC或者HNA消息向非邻域节点广播链路
2. 节点产生TC或者HNA消息假装是另一个节点
3. 节点产生HELLO消息向非邻域节点广播
4. 节点产生HELLO消息假装是另一个节点
5. 节点转发改变的控制消息
6. 节点不广播控制消息
7. 节点没有正确选择多点中继
8. 节点转发广播控制消息不变，但不转发单播数据通信量
9. 节点“重放”先前记录的来自另一节点的控制通信量

对于控制消息(对于情况2、4和5)和对于控制消息(对于情况1和3)中宣布的各个链路(对于情况1、3)的发起者节点的认证可以用作对策。然而，为了防止节点重复旧的(和正确认证的)信息(情况9)，需要时间信息，允许节点积极地识别这种延迟的消息。

一般来说，数字签名和其他需要的安全信息可以作为单独的OLSR消息类型来传输，从而允许“安全”和“不安全”节点可以在相同的网络中共存（如果需要的话）。

具体而言，整个OLSR控制消息的真实性可以通过采用IPsec认证头来建立，而各个链路(情况1和3)的真实性需要分发额外的安全信息。

一个重要的考虑是所有OLSR的控制消息被转发给邻域内的所有节点（HELLO），或者广播给网络里的所有节点（TC）

### 20.3 与外部路由域的交互

通过HNA消息与外部路由域交互

```
路由信息从OLSR的拓扑表或路由表中提取 并且，如果路由，可能注入外部域管理该域允许的协议。
```

### 20.4 节点标识

```
OLSR不对节点地址做任何假设，除了假设每个节点具有唯一的IP地址。
```

## 21 流量和拥塞控制

```
由于其主动性，OLSR协议对其控制流量的流具有自然的控制。节点以预定刷新间隔固定的预定速率发送控制信息。此外，MPR优化极大地节省了控制开销，这是从两方面进行的。首先，由于可以只通告MPR选择器，所以通告拓扑的分组要短得多。其次，由于只有MPR节点转发广播分组，所以洪泛该信息的成本大大降低。在密集网络中，与使用经典洪泛（如OSPF）的路由协议相比，控制流量的减少可以达到几个数量级[10]。这个特性自然为有用的数据流量提供更大的带宽，并进一步推动了拥塞的前沿。由于控制业务是连续的和周期性的，因此它保持了路由中使用的链路的质量更稳定，其中具有用于路由发现和修复的突发洪流的反应性协议可能通过在这些链路上造成大量冲突而在短时间内损坏链路质量，可能引发路由修复级联。然而，在某些OLSR选项中，一些控制消息可能在它们的最后期限（TC或Hello消息）之前被有意地发送，以便增强协议对拓扑改变的反应性。这可能导致控制流量的小的、临时的和本地的增加。
```

## 22 IANA注意事项

OLSR为控制消息定义了Message Type

```
 		消息类型			  值
      -------------------- -----
       HELLO_MESSAGE		 1
       TC_MESSAGE			 2
       MID_MESSAGE			 3
       HNA_MESSAGE 			 4
```