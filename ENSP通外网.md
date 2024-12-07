# ENSP模拟器通外网

# 一、添加Windows 的 KM-TEST 环回网卡
1、win+r键 运行cmd：输入hdwwiz

![image-20241206190406950](ENSP通外网.assets/image-20241206190406950.png)

2、在添加硬件界面，单击下一页

![image-20241206191051602](ENSP通外网.assets/image-20241206191051602.png)

3、选择安装我手动从列表选择的硬件（高级）

![image-20241206191108828](ENSP通外网.assets/image-20241206191108828.png)

4、往下找“网络适配器”

![image-20241206191350213](ENSP通外网.assets/image-20241206191350213.png)

5、在厂商类别中找到Microsoft，型号选择：Microssoft KM-TEST适配器，单击 下一页

![image-20241206191416591](ENSP通外网.assets/image-20241206191416591.png)

6、等待安装结束，win+r键 运行cmd：输入ncpa.cpl。

![image-20241206191911904](ENSP通外网.assets/image-20241206191911904.png)

7、查看当前的网卡是否创建完成

![image-20241206191556647](ENSP通外网.assets/image-20241206191556647.png)

# 二、配置共享本地网卡

1、找到当前与现网通信的网卡，比如我现在用的是WLAN上网，那我当前使用的就是WLAN网卡，如果你是插的网线，那么你就在通过以太网网卡上网。

![image-20241206192253713](ENSP通外网.assets/image-20241206192253713.png)

2、在当前上网的网络适配器设置中，右键，--点击属性

![image-20241206192329971](ENSP通外网.assets/image-20241206192329971.png)

3、勾上允许其他网络用户通过计算机的Internet连接来连接，选择刚才创建的KM-TEST环回网卡，我的是以太网2，点确定。

![image-20241206192441230](ENSP通外网.assets/image-20241206192441230.png)

3、提示地址会被设置为192.168.137.1，点击是

![image-20241206192557498](ENSP通外网.assets/image-20241206192557498.png)

4、完成网络共享到环回网卡



然后重启计算机，不然ENSP无法识别到新创建的网卡

# 三、ENSP与公网通信

![image-20241206192936831](ENSP通外网.assets/image-20241206192936831.png)

配置AR1

```
[Huawei]int g0/0/0
# 接口配置 192.168.137.0/24这个网段的地址，别是192.168.137.1，这个地址是你电脑的KM-TEST网卡已经设置了
[Huawei-GigabitEthernet0/0/0]ip address 192.168.137.123 24	
# 配置一条静态的缺省路由，让AR路由器可以发到192.168.137.1，也就是KM-TEST网卡，然后因为是网络共享，KM-TEST网卡会通过WLAN网卡上网
[Huawei]ip route-static 0.0.0.0 0.0.0.0 192.168.137.1

```

测试ensp中路由连接外网

```
[Huawei]ping 8.8.8.8
```

# 四、ENSP与公网通过DNS通信

![image-20241206193910665](ENSP通外网.assets/image-20241206193910665.png)

```
[Huawei]int g0/0/1
[Huawei-GigabitEthernet0/0/0]ip address 192.168.2.2 24	
```

PC配置IP、网关、DNS

![image-20241206194822058](ENSP通外网.assets/image-20241206194822058.png)

AR路由器是支持作为DNS客户端或DNS Proxy/Relay，可以通过静态和动态方式配置DNS解析，不支持自身作为DNS服务器。

1、静态方式：通过手动建立域名和IP地址之间的对应关系表来实现静态域名解析，在设备上建立静态域名解析表，当客户端需要域名所对应的IP地址时，在静态域名解析表中查找指定的域名，从而获得所对应的IP地址，实现域名解析。 

比如现在windows上用“nslookup”名查一下 域名baidu.com对应的，（每个地区不一样，以你的真实反馈为准）

![image-20241206194230988](ENSP通外网.assets/image-20241206194230988.png)

例如：

```
[Huawei]ip host baidu.com 39.153.66.10  //配置静态DNS表项  
[Huawei] dns proxy enable   //配置DNS代理功能
```

DNS代理接收到DNS客户端的DNS请求报文后会先查找本地域名解析表，然后我们用PC测试访问baidu.com

```
PC>ping baidu.com
```

baidu.com，可以通；其他的域名都不能通，因为你只写了这一个静态映射，其他的域名都没有进行映射。

2、动态方式：

- DNS代理接收到DNS客户端的DNS请求报文后会先查找本地域名解析表，如果未查询到对应的解析表项，才将DNS请求报文转发给DNS服务器。
- DNS中继接收到DNS客户端的DNS请求报文后不会查询本地域名解析表，而是直接将其转发给DNS服务器进行解析。

动态域名解析需要域名服务器的配合。
例如：

```
[Huawei] dns relay enable   // 配置DNS中继（可选）
[Huawei] dns resolve // 使能动态域名解析功能 

```

查看你的本地域名解析服务器地址

![image-20241206200056423](ENSP通外网.assets/image-20241206200056423.png)

```
[Huawei] dns server 223.5.5.5 //配置DNS服务器的IP地址 
```

现在动态域名配置完，PC就可以随便ping 域名了，只要是你Windows能ping 通的，PC都可以ping通。



3、DHCP方式：AR作为DHCP服务器给下面PC进行DHCP动态分配地址的时候，也可以分配DNS地址，这个是在DHCP的功能中配置。

```
[Huawei] dhcp enable  //开启DHCP功能
[Huawei] ip pool pool1   //配置全局地址池
[Huawei-ip-pool-pool1] network 192.168.2.0 mask 255.255.255.0  //配置可动态分配的IP地址范围
[Huawei-ip-pool-pool1] gateway-list 192.168.2.1 //配置网关地址
[Huawei-ip-pool-pool1] dns-list 223.5.5.5 //配置DNS地址
[Huawei-ip-pool-pool1] quit
[Huawei] interface GigabitEthernet 0/0/1  //DHCP服务器连接客户端的接口
[Huawei-GigabitEthernet0/0/1] ip address 192.168.2.1 255.255.255.0 //须与地址池在同一网段
[Huawei-GigabitEthernet0/0/1] dhcp select global //使能接口采用全局地址池的DHCP服务器功能
```

然后将PC改成DHCP获取

![image-20241206204325008](ENSP通外网.assets/image-20241206204325008.png)

```
//查看PC的DHCP获取
PC>ipconfig			
```

这样PC在通过域名访问的时候，就会直接找域名服务器