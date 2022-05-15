### 一、实验要求

利用仿真器Packet Tracer，实现如下网络

![image-20220512093928125](https://s2.loli.net/2022/05/12/qw7lfSkTZBGsbA4.png)

<center style="color:#808080;text-decoration;font-size:14px">实验要求网络拓扑图</center>

#### 具体实现技术如下

1. **二层交换机的每个端口在不同Vlan。**
2. **利用静态路由来配置路由。**
3. **要求用到DHCP技术来分配IP地址。**
4. **要求用到NAT技术完成地址变换。**
5. **要求用到单臂路由来实现互联。**
6. **要求用到GRE隧道技术。**
7. **要求用到访问控制列表技术。**
8. **任意两个节点之间能在规则下互相访问。**

### 二、实验具体实现及相关命令

#### 2.1 二层交换机的每个端口在不同的VLAN

> ​		VLAN（虚拟局域网）是对连接到的第二层交换机端口的网络用户的逻辑分段，不受网络用户的物理位置限制而根据用户需求进行网络分段。一个VLAN可以在一个交换机或者跨交换机实现。VLAN可以根据网络用户的位置、作用、部门或者根据网络用户所使用的应用程序和协议来进行分组。基于交换机的虚拟局域网能够为局域网解决冲突域、广播域、带宽问题。

以下图为例

![img](https://s2.loli.net/2022/05/12/eCmEHt4P9yGbgvI.png)

<!--相关交换机配置命令-->

```bash
# 添加vlan 3,8 (对应学号19030500103、19030500308)
Switch(config)#vlan 2
Switch(config-vlan)#exit
Switch(config-vlan)#vlan 8
Switch(config)#exit
# 进入端口f0/3, f0/4
Switch(config)#interface fa 0/3
Switch(config-if)#switch access vlan 8
Switch(config-if)#exit 
Switch(config)#interface fa 0/4
Switch(config-if)#switch access vlan 3
Switch(config-if)#exit
Switch#show vlan
```

#### 2.2 利用静态路由来配置路由。

> ​		静态路由（Static routing）是一种路由的方式，路由项（routing entry）由手动配置，而非动态决定。与动态路由不同，静态路由是固定的，不会改变，即使网络状况已经改变或是重新被组态。一般来说，静态路由是由网络管理员逐项加入路由表。
>
> ​		优点：使用静态路由的另一个好处是网络安全保密性高。动态路由因为需要路由器之间频繁地交换各自的路由表，而对路由表的分析可以揭示网络的拓扑结构和网络地址等信息。因此，网络出于安全方面的考虑也可以采用静态路由。不占用网络带宽，因为静态路由不会产生更新流量。静态路由适用于中小型网络。
>
> ​		缺点：大型和复杂的网络环境通常不宜采用静态路由。一方面，网络管理员难以全面地了解整个网络的拓扑结构；另一方面，当网络的拓扑结构和链路状态发生变化时，路由器中的静态路由信息需要大范围地调整，这一工作的难度和复杂程度非常高。当网络发生变化或网络发生故障时，不能重选路由，很可能使路由失败。

<!--相关路由器配置命令-->

```bash
# 配置路由命令
Router(config)#ip route [Destination_network] [Subnet_mask] [Next_hop]
# 显示当前路由器下所有route规则
Router#show ip route
```

#### 2.3 要求用到DHCP技术来分配IP地址

> ​		DHCP（动态主机配置协议）是一个局域网的网络协议。指的是由服务器控制一段IP地址范围，客户机登录服务器时就可以自动获得服务器分配的IP地址和子网掩码。默认情况下，DHCP作为Windows Server的一个服务组件不会被系统自动安装，还需要管理员手动安装并进行必要的配置。

以下图网络为例

![img](https://s2.loli.net/2022/05/12/2TLo7z8Wl3jOJCf.png)

<!--相关路由器配置命令-->

```bash
Router(config)#interface FastEthernet 0/0 # 进入到接口0/0 
Router(config-if)#no shutdown # 启动接口，路由器的接口状态默认是shutdown 
Router(config-if)#exit # 退出该接口 
# 进入到F0/0.1接口 
Router(config)#interface fastEthernet0/0.1 
# 配置接口的ip地址
Router(config-subif)#ip address 192.168.38.1 255.255.255.240 
# 退出F0/0.1接口 
Router(config-subif)#exit 
# 进入到F0/0.2接口 
Router(config)#interface fastEthernet 0/0.2
 # 配置ip地址
Router(config-subif)#ip address 192.168.8.1 255.255.255.240 
# 建立dhcp地址池
Router(config)#ip dhcp pool weijunwei
# 为dhcp地址池配置子网
Router(config)#network 192.168.103.1 255.255.255.240 
# 配置网关
Router(config)#default-router 192.168.103.1
# 配置dns-server
Router(config)#dns-server 192.168.103.100
# 简历dhcp地址池
Router(config)#ip dhcp pool wuyue
Router(config)#network 192.168.8.1 255.255.255.240 
Router(config)#default-router 192.168.8.1
Router(config)#dns-server 192.168.8.100
```

#### 2.4 NAT技术完成地址变换

> ​		NAT（Network Address Translation），是指网络地址转换，1994年提出的。
> ​		当在专用网内部的一些主机本来已经分配到了本地IP地址（即仅在本专用网内使用的专用地址），但又想和因特网上的主机通信（并不需要加密）时，可使用NAT方法。
> ​		这种方法需要在专用网（私网IP）连接到因特网（公网IP）的路由器上安装NAT软件。装有NAT软件的路由器叫做NAT路由器，它至少有一个有效的外部全球IP地址（公网IP地址）。这样，所有使用本地地址（私网IP地址）的主机在和外界通信时，都要在NAT路由器上将其本地地址转换成全球IP地址，才能和因特网连接。
> ​		另外，这种通过使用少量的全球IP地址（公网IP地址）代表较多的私有IP地址的方式，将有助于减缓可用的IP地址空间的枯竭。

以下图网络为例

![地址变化](https://s2.loli.net/2022/05/12/MuxeNl6QPRBwZ3o.png)

<!--相关路由器配置命令-->

```bash
# 该端口在配置单臂路由时已经指定ip地址，此处不再指定，直接进行配置
Router(config)#int f0/0.1
# 指定内部接口
Router(config-subif)#ip nat inside
Router(config-subif)#ip nat outside
Router(config-subif)#exit
# 建立两个ip地址之间的静态映射
Router(config)#ip nat inside source static 192.168.103.34 38.1.1.1 
Router(config)#exit
```

#### 2.5 单臂路由来实现互联

> ​		单臂路由（router-on-a-stick）是指在路由器的一个接口上通过配置子接口（或“逻辑接口”，并不存在真正物理接口）的方式，实现原来相互隔离的不同VLAN（虚拟局域网）之间的互联互通。

以下图网络为例，该网络只需要配置路由器某个端口下有两个子端口

![单臂](https://s2.loli.net/2022/05/12/eCmEHt4P9yGbgvI.png)

<!--相关交换机配置命令-->

```bash
# 单臂路由需要设置trunk接口
Switch(config)#interface fastEthernet 0/1 
Switch(config-if)#switchport mode trunk 
# 配置turnk口允许vlan2 和vlan3通过，除了 vlan2和vlan3，其他所有的vlan默认拒绝所有 
Switch(config-if)#switchport trunk allowed vlan 2,3，8 
```

<!--相关路由器配置命令-->

```bash
# 进入到F0/0.1接口 
Router(config)#interface fastEthernet0/0.1 
# 将vlan2封装在F0/0.1接口 
Router(config-subif)#encapsulation dot1Q 3 
# 配置接口的ip地址
Router(config-subif)#ip address 192.168.103.1 255.255.255.240  
# 进入到F0/0.2接口 
Router(config-subif)#interface fastEthernet 0/0.2 
# 将vlan3封装在F0/0.2接口 
Router(config-subif)#encapsulation dot1Q 3 
# 配置ip地址，该ip地址作为vlan3内的电脑的网关
Router(config-subif)#ip address 192.168.103.16 255.255.255.240
```

#### 2.6 GRE隧道

> ​		GRE VPN（Generic Routing Encapsulation）即通用路由封装协议，是对某些网络层协议（如IP和IPX）的数据报进行封装，使这些被封装的数据报能够在另一个网络层协议（如IP）中传输。
> ​		GRE是VPN（Virtual Private Network）的第三层隧道协议，即在协议层之间采用了一种被称之为Tunnel（隧道）的技术。

以下图网络为例，使用GRE隧道技术，使得从193.168.103.16/28网段到192.168.8.80/28网段的流量走GRE隧道。如下图所示

![GRE](https://s2.loli.net/2022/05/12/pvZBszlQXyboSWC.png)

<!--相关路由器配置命令-->

```bash
# 配置GRE隧道模式
interface Tunnel 0     
# 配置GRE IP地址
ip address 172.16.1.1 255.255.255.0
# 配置GRE IP源地址
tunnel source FastEthernet0/0
# 配置GRE IP目的地址
tunnel destination 38.1.1.2
# 配置静态路由
ip route 192.168.8.98 255.255.255.240 172.16.1.2   //引流
# 同理可配置GRE隧道另一端的路由
```

#### 2.7 访问控制列表

> ​		访问控制列表(ACL)是一种基于包过滤的访问控制技术，它可以根据设定的条件对接口上的数据包进行过滤，允许其通过或丢弃。访问控制列表被广泛地应用于路由器和三层交换机，借助于访问控制列表，可以有效地控制用户对网络的访问，从而最大程度地保障网络安全。

以下图网络为例

![访问控制](https://s2.loli.net/2022/05/12/Re5rx19bQFBJpGV.png)

<!--相关路由器配置命令-->

```bash
# 禁止所有来自于ip为192.168.103.50主机的流量通过该路由器
# 配置access-list
Router(config)#access-list 1 deny icmp 192.168.103.50 0.0.0.0
# 启用ACL 
Router(config)#access-list 1 permit ip any any 
Router(config)#int fa0/0
Router(config-if)#ip access-group 1 in
```

#### 2.8 两个节点之间能在规则下互相访问

***该部分内容将在实验结果展示（使用ping命令测试网络的连通性）***

### 三、实验结果

<img src="https://s2.loli.net/2022/05/12/ZYC3OU4hy5RePrp.jpg" alt="img" style="zoom:80%;" />

PS：配置相关ip地址参数为103/8/38均与学号相关联

#### 3.1 划分VLAN

以下以交换机switch1为例，划分了3个VLAN分别是VLAN 2, VLAN 3, VLAN 8。（其中3为103尾号，8为308尾号）

<img src="https://s2.loli.net/2022/05/12/kNXlvS5hQnri6zg.jpg" alt="img" style="zoom: 80%;" /> 

#### 3.2  配置静态路由

下面以Router2为例，配置了该路由器到子网192.168.38.0/28，192.168.8.0/28，192.168.8.80/28，192.168.8.96/28的相关路由信息。

<img src="https://s2.loli.net/2022/05/12/9vfnqrmbwjs1Lgz.jpg" alt="img" style="zoom:80%;" /> 

#### 3.3 单臂路由配置

以Router2为例，Router2 intf0/1为Trunk口能通过vlan 2，3，8的流量，为连接在switch1的主机划分vlan 2，3 ，8。配置路由其相关子接口。实现vlan2，3，8之间互联。

<img src="https://s2.loli.net/2022/05/12/7ckUjalHN52fECq.jpg" alt="img" style="zoom:80%;" /> 

**Ping测试结果如下**

<img src="https://s2.loli.net/2022/05/12/DBwQ7Gb2F1MOEmr.jpg" alt="img" style="zoom:80%;" /> 

 

#### 3.4 配置DHCP

使用DHCP分配的IP地址范围分别为

PC0:192.168.103.2~192.168.103.15

PC1:192.168.8.2~192.168.8.15

详情如下图：

<img src="https://s2.loli.net/2022/05/12/BbukjW75tfTQZU4.jpg" alt="img" style="zoom:80%;" /> 

还需要为Router1配置默认路由，和其它主机实现互通。

<img src="https://s2.loli.net/2022/05/12/lTN5zSp8kmGvA7J.jpg" alt="img" style="zoom:80%;" /> 

#### 3.5 NAT配置

本实验使用的是静态NAT配置

Inside 192.168.103.34 Outside 38.1.1.1

<img src="https://s2.loli.net/2022/05/12/BCY58aTRcjXsiLD.jpg" alt="img" style="zoom:80%;" /> 

**Ping测试结果如下**

外网Ping 192.168.103.34

<img src="../AppData/Local/Temp/ksohtml2856/wps9.jpg" alt="img" style="zoom:80%;" /> 

内网

<img src="https://s2.loli.net/2022/05/12/aRgX69S1iGIvthJ.jpg" alt="img" style="zoom: 67%;" /> 

#### 3.6 ACL 访问控制列表

在路由器Route7上设置了访问控制列表，限制所有的192.168.103.50的流量通过路由器。

<img src="https://s2.loli.net/2022/05/12/MOKRv3iAWoYuykE.jpg" alt="img" style="zoom:80%;" /> 

![img](https://s2.loli.net/2022/05/12/qXQ82LCzxy7PTKd.jpg) 

**Ping测试结果如下**

<img src="https://s2.loli.net/2022/05/12/uo3dw5TIUkhaBY6.jpg" alt="img" style="zoom: 67%;" /> 

#### 3.7 配置GRE通道

配置了GRE通道，使得192.168.103.16/28网段到192.168.8.80/28网段走Tunnel0通道。

<img src="https://s2.loli.net/2022/05/12/DcHwWZIjAKaf6CQ.jpg" alt="img" style="zoom:80%;" /> 

 

<img src="../AppData/Local/Temp/ksohtml2856/wps15.jpg" alt="img" style="zoom:80%;" /> 

**Ping测试**

<img src="https://s2.loli.net/2022/05/12/UZLPVlfwkQGpjMv.jpg" alt="img" style="zoom: 67%;" /> 

<img src="https://s2.loli.net/2022/05/12/1uYjqXDVdbQ5JnB.jpg" alt="img" style="zoom: 67%;" /> 

由此可见网段192.168.8.80/28到目标网段路由跳数为2，而下图跳数为3没有走Tunnel。

#### 3.8 配置静态路由以及单臂路由使得所有的主机之间可以进行通信

配置静态路由以及单臂路由使得所有的主机之间可以进行通信

以下展示部分PC之间ping

**PC 13 ping PC1**

<img src="https://s2.loli.net/2022/05/12/uyPT4SlLkINiYb6.jpg" alt="img" style="zoom: 67%;" /> 

**PC 8 ping PC 13**

<img src="https://s2.loli.net/2022/05/12/8p39OvsMN7rCGFK.jpg" alt="img" style="zoom: 67%;" /> 

### 四、实验总结

#### 4.1 实验中主要遇到的困难

- 对仿真器Packet Tracer不是特别熟悉，熟悉环境以及熟悉相关配置路由、交换机、隧道，访问控制列表的命令花费了比较多的时间。
- 对于不太熟悉这个网络拓扑的我们，在这个网络中进行子网划分存在一定的疑惑，以及困难。

#### 4.2 实验中遇到困难时的一些解决方案

- 通过百度等搜索引擎查找相关命令的用法。
- 翻阅书籍，查看交换机以及路由器的相关概念以及路由规则等。
- 回忆上个学期做过的计算机网络实验，该实验和那次实验有相似之处。

#### 4.3 实验心得

- 通过此次实验，让我们组复习了计算机网络相关的知识。
- 实验的动手的方式使得我们对这些基础知识有了更加深刻的理解与认识。
- 了解了在网络出现问题的时候，如何进行排查问题、调试问题。
