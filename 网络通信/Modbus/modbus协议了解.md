### **4种数据类型** 

Modbus协议规定，进行读写操作的数据类型，按照读写属性和类型可分为以下4种：

- 离散量输入（Discretes Input  ）：1位，只读
- 线圈（Coils）：1位，读写
- 输入寄存器（Input Registers  ）：16位，只读
- 保持寄存器（Holding Registers）：16位，读写



Modbus地址模型的编号从1开始。由于每一种数据都最大支持65536个元素，具体如下：

* 线圈型数据，地址范围(从1开始)：000001~065536；
* 离散量输入，地址范围(从1开始)：100001~165536；
* 输入寄存器，地址范围(从1开始)：300001~365536；
* 保持寄存器，地址范围(从1开始)：400001~465536；

由于上述地址都是比较大的数值，实际应用一般采用区域代码+地址的方式，或者直接采用16进制表示地址，具体如下：

* 线圈型数据，区域代码+地址(从0开始)：DQ0~DQ65535；
* 离散量输入，区域代码+地址(从0开始)：DI0~DI65535；
* 输入寄存器，区域代码+地址(从0开始)：IR0~IR65535；
* 保持寄存器，区域代码+地址(从0开始)：HR0~HR65535，16进制(从0开始)：0000H~FFFFH；

Modbus地址模型对于Modbus-RTU/ASCII和Modbus-TCP协议都是适用的。





### **3种传输模式** 

1979年，Modicon 首先推出了串行Modbus标准，后来由于网络的普及，需要更高的传输速度，1997年制定了基于TCP网络的Modbus标准。

所以总的可分为两个传输模式：基于**串行链路**的和基于**以太网TCP/IP**的。但是我个人还是习惯分为3种传输模式：

- **基于串口的Modbus-RTU**数据按照[标准串口协议](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzNzk2NTMxMw%3D%3D%26mid%3D2247484166%26idx%3D1%26sn%3Def4b89ed24eeeb847c91093cfe318b70%26scene%3D21%23wechat_redirect)进行编码，是使用最广泛的一种Modbus协议，采用[CRC-16_Modbus校验算法](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzUzNzk2NTMxMw%3D%3D%26mid%3D2247485023%26idx%3D1%26sn%3D1c0972062c0393fe385cd1777ebf6dab%26scene%3D21%23wechat_redirect)。
- **基于串口的Modbus-ASCII**所有数据都是ASCII格式，一个字节的原始数据需要两个字符来表示，效率低，采用LRC校验算法。
- **基于网口的Modbus-TCP**Modbus-TCP基于TCP/IP协议，占用**502端口**，数据帧主要包括两部分：MBAP（报文头）+PDU（帧结构），数据块与串行链路是一致的。





### 汇川 MB、MX、MW、MD的含义及长度

M表示内部存储区。MB表示长度为字节的操作数在内部存储区，MW表示长度为字的操作数在内部存储区，MD表示长度为双字的操作数在内部存储区。

操作数包含两个要素：标识符Q和标识参数。标识符用来表示操作数存放区域及操作位数；标识参数用来表示操作数在该存储区域内的具体位置。

存储区域包括有：输入映像区(I)，输出映像区(Q),内部存储区(M),物理输入区(P),物理输出区(PQ),数据块(DB),数据块(DI),临时堆栈Q(L)



MB：1个字节，8为

MX：MB的位，MX0.1表示MB的第二位

MW：2个字节，16位

MD：4个字节，32位



MD和MB都要转为MW，MB的需要除以2，MD需要乘以2；