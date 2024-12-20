# 浅谈华为防火墙

**编辑：大树老师丨HCIE安全专家级讲师，更多网络安全干货，请关注公众号：网络工程师训练营**

防火墙是我们日常生活中经常使用的一种网络设备，为什么会被经常使用呢？

因为我们要保护我们的网络免受来自网络的攻击和入侵。

怎么做到防御的呢？

验证数据包的来源、目的地和内容，根据预先设定的安全策略，决定是否允许数据包通过或丢弃，根据部署位置的不同，侧重点也不同，部署在网络边界上主要就是防御来自外网的攻击，不是在服务器集群之前主要就是防御针对服务器的攻击，部署在内网不同区域之间主要就是控制内网之间的访问，也就是防火墙的安全保护有很多种玩法，我们今天就来看看华为防火墙的基础知识.

**今日文章阅读福利：《华为防火墙配置指南》**

**私信发送暗号“华为防火墙”，即可获取此份优质指南，提升你的技术水平。**

### 产品系列

- **USG5000 系列**：这个是传统防火墙，配置复杂，而且不支持多通道协议，比如FTP、SIP这种协议，所以现在我们使用这种防火墙极少，就不再重点介绍。
- **USG6000 系列**：下一代防火墙产品，NGFW，Next Generation Firewall，适用于大中型企业及数据中心等网络环境，比之前的5000系列的防火墙配置简单、性能高，而且支持多通道协议，如何支持的呢？这就是我们后续要学的ASPF技术，6000系列应用很广泛，包括华为的认证也使用的是6000系列，可用于边界防护、互联网出口防护等部署位置。
- **USG9500 系列**：该系列主要用在大型数据中心、大型企业园区网络等，访问控制更加精准，最领先的 “NP + 多核 + 分布式” 构架及最丰富的虚拟化，主要用在大型数据中心边界防护、广电和二级运营商网络出口安全防护、教育网出口安全防护等网络场景。

### 工作模式

![img](https://picx.zhimg.com/80/v2-28dd84e3e991910727a12f27c2bcf74f_720w.png?source=d16d100b)



添加图片注释，不超过 140 字（可选）

- **路由模式**：防火墙默认情况下网络的接口是三层接口，也就是接口需要配置 IP 地址，这种就是路由模式下，此时防火墙和路由器一样都要进行路由转发，但是防火墙在路由的基础上，还有安全防护的功能，大多数情况下，防火墙都是路由模式。
- **透明模式**：透明模式，就是接口是交换模式，不配置 IP ，反而是跟交换机一样，配置access、trunk、hyprid类型，可将其当成交换机使用，但是数据包还是会被防火墙进行检查，只是不会进行路由转发，而是二层转发。
- **混合模式**：华为防火墙可以同时存在路由模式的接口和透明模式的接口，工作在混合模式下，就是有的接口是三层可以配置IP地址，有的接口是二层接口，二层转发，主要体现在双机热备的透明模式中。

### 安全区域划分

![img](https://pic1.zhimg.com/80/v2-5c8191dcdb25874912c1e03d6fc3cbec_720w.png?source=d16d100b)



防火墙安全区域

- **Trust 区域**：通常用来定义内部用户所在的网络，优先级为 85，优先级数字越大，代表该区域内的网路越可信.
- **Untrust 区域**：通常连接外部网络，优先级为 5，安全级别很低，一般把连接Internet的接口 划入此区域.
- **DMZ 区域**：非军事化区域，通常用来定义内部服务器所在的网络，其安全性介于 Trust 和 Untrust 区域之间，优先级为 50.
- **Local 区域**：这个区域代表防火墙本身，优先级为 100，防火墙除了转发区域之间的报文之外，还需要自身接受或发送流量，凡是由防火墙主动发出的报文均可认为是从Local区域中发出，凡是需要防火墙响应并处理（而不是转发）的报文均可认为是由Local区域接收。
- **自定义区域**：除此之外，我们还可自定义区域，自定义的安全区域，需要自己手动指定优先级。

### 报文流动方向

![img](https://picx.zhimg.com/80/v2-549bac9e580dc40ab28dd0e3e160d098_720w.png?source=d16d100b)





报文流动方向

- **Inbound 和 Outbound**：当数据流在安全区域之间流动时，才会触发防火墙的安全策略的检查。Inbound 方向是指数据由低级别的安全区域向高级别的安全区域传输的方向，如 Untrust 区域（5）到 Trust（85） 区域的流量；Outbound 方向是指数据由高级别的安全区域向低级别的安全区域传输的方向，如 Trust 区域（85）到 Untrust （5）区域的流量。如今的6000系列已经不需要这种流量方向的标识，而5000系列需要在写安全策略时指明流量方向。
- **状态化信息**：防火墙通过五元组（即源 IP、目标 IP、协议、源端口号、目标端口）来唯一区分一个数据流。防火墙的状态化检测机制会重点处理首个报文，安全策略一旦允许首个报文通过，将会形成一个会话表，后续报文和返回的报文如果匹配到会话表将会直接放行，从而提高防火墙的转发效率。这是6000系列这种NGFW下一代防火墙的特点，可以创建会话表，根据会话表进行转发。

### 配置与管理方式

- **界面访问**：华为防火墙USG6000系列开始提供了 Web 界面的管理方式，方便配置和日常管理，默认的G0/0/0口是管理口，配置有默认的IP地址192.168.0.1/24位，并且允许了https的链接。

也就是，我们拿到一个USG6000的防火墙，使用自身电脑设置和192.168.0.0/24同网段的一个地址，然后https://192.168.0.1:8443，就可以登录到防火墙的web管理页面进行管理。

编辑：大树老师丨HCIE安全专家级讲师，更多网络安全干货，请关注公众号：网络工程师训练营
