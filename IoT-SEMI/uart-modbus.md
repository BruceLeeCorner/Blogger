[TOC]

# 异步串行通信协议详解

## 串行通信和并行通信

## 同步串行通信和异步串行通信的区别

串行通信又可以分成同步方式和异步方式。

同步通信和异步通信最大的差别是，同步通信必须有时钟信号线，而异步通信不需要。同步通信不仅要接数据线，还要接一根时钟线！

![image](https://img2023.cnblogs.com/blog/1037641/202311/1037641-20231111204422600-1016527777.png)

同步串行通信，不会拆分有效数据，它在一大坨有效数据的两端加上开始和结束bit以及CRC校验的bit，一次性传送出去。

![image](https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210903225635294-289353670.png)

发送方的时钟控制时钟信号线上的电平，每当被发送的bit维持到一半时间时，立刻取反时钟线上的电平以产生1个上升沿或下降沿；而接收方的时钟也必须实时监控时钟线上的电平，每当监测到一个上升沿或下降沿立刻去采样，将此时采样的电平作为一个有效bit。

异步串行通信，会把有效数据拆分成若干个8个bit的片段，每个片段首位加上开始和结束bit以及校验bit组成数据帧，然后依次传送每个数据帧。优点是数据帧很短，对通信双方的时钟要求没那么高，通信双方使用两个不同的时钟，只要两个时钟的误差足够小即可。

**同步串行通信和异步串行通信最大的不同之处**

1. 同步对时钟的精度要求较高

   异步串行通信，双方的时钟对RX和TX数据线进行监测，只要在对串口帧的数据位的8次采样后累计的时钟偏差不超出合理的采样范围，那么时钟的精度就够用了。

   同步串行通信，双方的时钟对CLK时钟控制线进行监测，数据帧有多长，时钟控制线就要维持多久，双方的时钟要保证在这一个串口帧发送结束时，累计偏差不能构成采样偏差！由于同步串行通信的串口帧很长，所以累计偏差很多次，所以精度要足够高。

2. 异步串行通信的速率较快。因为异步串行通信的一个串口帧包含的数据位很长，但是同步串行通信一个串口帧只有8个数据位，前者传输的有效数据位占比高！

3. 异步串行通信的每个字节之间都会至少间隔1个停止位，或间隔1或1.5或2个停止位 + 任意长度的空闲位，但同步串行通信的每个字节之间是没有时间间隔的。

## 信道种类

**单工通信**

单工就是指任何时刻只能往某一个固定方向传输数据，通信是单向的，就像灯塔之于航船，收音机之于耳朵，遥控器之于电视机。

**半双工通信**

半双工就是指通信的一方不可同时既收又发数据，可分时收发数据。分时使用信道来发送数据！就像我们在影视作品中看到的对讲机一样：
007：呼叫总部，请求支援，OVER
总部：收到，增援人员将在5分钟内赶到，OVER
007：要5分钟这么久？！要快呀！OVER
总部：……
　　　　　　　GAME OVER
在这里，每方说完一句话后都要说个OVER，然后切换到接收状态，同时也告之对方——你可以发言了。如果双方同时处于收状态，或同时处于发状态，便不能正常通信了。

**全双工通信**

全双工比半双工又进了一步。在A给B发信号的同时，B也可以给A发信号。典型的例子就是打电话。
A：我跟你说呀……
B：你先听我说，情况是这样的……
A和B在说的同时也能听到对方说的内容，这就是全双工。

## UART帧结构示意图

<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210903204646420-2110699962.png" width="50%" height="50%" title=""/></div>

- 每帧由 1起始位 + (5,6,7,8)数据位 + 1校验位(可选) + (1,1.5,2)停止位组成。
- 最大帧长12bit(1起始位 + 8数据位 + 1校验位 + 2停止位)，最小帧长7bit(1起始位 + 5数据位 + 0校验位 + 1停止位)
- 发送数据位时，低位在前，高位在后，即先发字节的低权值bit，后发高权值bit，如 字母A（0100 0001），发送顺序是1000 0010.

## UART参数释义

**波特率**

每秒钟发送的bit数量，bits per second, bps。常见值1200、2400、4800、9600、19200、38400、115200；记作115200bps或115200b/s, b/s表示bits/每秒，B/s 表示bytes/每秒。

波特率越大，通信速率越高。但波特率越高，逻辑电平的(维持时间)周期越短，接收方定时采样时对时钟精度的累计偏差容忍度越低，所以，如果对串口通信速度有要求的话，通信双方的时钟精度要高,串口驱动程序的中断响应能力和数据处理效率要求也越高，否则误码率会高。
如果通信误码率频出，数据校验失败频发，可以尝试降低波特率，减小数据位，增大停止位。
常用的波特率是9600bps,如果采用8-None-1，每秒钟可以传输9600 / (1 + 8 + 1) = 960个字节，约1ms发送1个字节。



**数据位**
常见值 5，6，7，8。选择7的理由：ASCII码有效位是7bit，一帧包含1个完整的ASCII字符是很合适的。选择8的理由：一帧传输1个完整的字节也是很合适的。选择5,6的理由：电子设备5(0-31共计32个整数)或6(0-63共计64个整数)个bit就可以包含一个完整的控制指令，而传输1个完整的字节有点浪费。

`当今，几乎99%的串口设备数据位都是8。`

不选择5，6，7的原因：高级语言编写的接收程序，捕获到一个串口帧后会补全空余的bit(如数据位是6，会补2个0)凑成1个字节作为API的返回值，应用程序拿到字节后可以截取低5bit处理，虽然相比8数据位处理起来没啥太大区别，但是这样写程序还是有点累，其次就是现在存储成本和处理器性能日新月异，不会在意这2，3个bit的效率问题，而且8bit比5bit能表达更多的信息量，没必要省那一两个bit。

不选择16，24，32更大的数据位的原因：协议包都是1个字节(8bit)的整数倍，并不一定是16，24，32等的整数倍，只会让串口驱动程序变复杂，同时单帧数据位越长，时钟累计误差越大，误码的可能性越大，如果假设数据位无限长就变成同步通信了！



**校验位**

无校验None，奇校验Odd，偶校验Even，0校验Space，1校验Mark；

8个数据位 + 1个校验位共计9个电平，1的个数是奇数称为奇校验，1的个数是偶数称为偶校验，校验位始终放置0，称为SPACE校验，校验位始终放置1，称为Mark校验。

举例：串口帧组成：起始位(低电平) + 8个数据位 + 1个校验位 + 停止位。 当8个数据位1的个数是偶数时，采用偶校验的话，停止位置0，采用奇校验的话，停止位置1。

添加校验位在一定程度上能够检查出接收到的一个串口帧的数据位数据的正确性，但此校验方法不是100%可靠的。比如采用偶校验，8个数据位其中2个1变异成0，但是1的个数还是偶数个，所以奇偶校验查不出来这种错误。
`随着串行通信硬件技术的抗干扰能力越来越强，在实际工程项目中，几乎都不使用奇偶校验这种鸡肋手段，而是在应用层协议添加CRC16校验，异或校验，累计和校验等更靠谱的校验手段保证通信的可靠性，同时由于不采用校验，串口帧少1个bit，通信效率也变高了。`



**停止位**

常见值 1，1.5，2；

**停止位有3个作用**

**作用1：标志着一帧的结束，可以借用停止位检查时钟有无偏差。**

因为停止位是高电平，接收方采样完数据位后，在下1个采样点采到的必定是高电平，否则这一帧肯定是错误数据，会触发接收帧错误回调函数。
1 Stop bit: UART transmits one Stop bit; the receiver verifies the first Stop bit received
1.5 Stop bits: UART transmits 1.5 Stop bits; the receiver verifies the first Stop bit only
2 Stop bits: UART transmits two Stop bits; the receiver verifies the first Stop bit only
2 Stop bits: UART transmits two Stop bits; the receiver verifies both

**作用2：停止位提供下降沿，以触发帧接收中断**

总线空闲时一直维持高电平，串口驱动程序一直监控下降沿。当一帧数据抵达时，起始位是低电平，下降沿会触发数据接收中断处理函数，接收方按照波特率定时采样。因为必须有下降沿才能触发接收，所以必须保证每一帧的尾巴必须有高电平，所以就有了停止位，以保证监测到下一帧的起始位下降沿。

**纠正时钟累计偏差**

`帧结束下降沿，都会开启1个新的定时器采样，所以，发送方和接收方的时钟累计误差不会影响到下一帧，只要保证累计误差在单帧中的10(11,11.5,12)次采样正常就可以了。`

为什么停止位有1.5和2的选择？
串口停止程序采完完整一帧后，肯定要处理一些额外的任务，停止位变长，可以让串口驱动程序有喘息的机会以更好的响应下一个下降沿。


Extra stop bits can be a useful way to add a little extra receive processing time, especially at high baud rates and/or using soft UART, where time is required to process the received byte.

`时钟累计误差越来越大，可能造成采样位置偏差到相邻电平，造成解码错误。`

接收方停止位设置成1，发送方的停止位设置成2，接收方可以正常接收到消息。
接收方停止位设置成2，发送方的停止位设置成1，接收方如果只检查第1个停止位，则接收正常，如果2个停止位都检查，则接收异常。
接收方停止位设置成2，发送方的停止位设置成3，4甚至更大，接收也能正常。
只要接收和发送方约定好串口参数，发送方的帧间距是随意的，因为接收方只要下降沿。但一般串口程序帧间距是0bit或1bit，最多2-3个bit。肯定不能太多，太多通信速率极低，而且一些协议如Modbus，用帧间隔时长作为一帧完整协议解析。

## 示波器解读UART协议

发送的配置：8个数据位、无校验位、1位停止位

发送的数据：0x55 = 0101 0101

<img src="https://img2023.cnblogs.com/blog/1037641/202309/1037641-20230910020959125-1339874032.png" alt="image" style="zoom:67%;" />

![image](https://img2023.cnblogs.com/blog/1037641/202309/1037641-20230910021010816-1721517416.png)

通过波形图可以得知一下结论： 

1. 0号位置和11号位置为空闲电平（高电平）
2. 1号位置为起始位（一个数据大小的低电平）
3. 2、3、4、5、6、7、8、9组成8位数据，为10101010，但是数据是低位在前，所以真是的数据位01010101
4. 10为停止位，高电平，可以选择一个1、1.5、2个数据大小的时间
5. 串口TX或RX数据线上没有传输任何数据时，则该线处于为空闲状态。空闲是TX和RX都是处于高电平。



# RS232 vs RS485 vs RS422

组网用RS485，一对一用RS232或RS485或RS422.




RS485-2Wire(二线制485)有3根引脚：
- RxD/TxD(+)
- RxD/TxD(-)
- GND

RS485-2Wire的引脚只分正负，接线时，正接正，负接负即可。
两个引脚的电压差作为0或1信号。
两个引脚是复用的，既收又发，是半双工通信，所以同一时间只能有一个设备发送数据。

RS485-2Wire支持一对多的组网，拓扑图如下：

![image](https://img2024.cnblogs.com/blog/1037641/202403/1037641-20240328093421996-1411404691.png)

一根线串起所有设备的RxD/TxD(+)引脚，另一根线串起来所有设备的RxD/TxD(-)，线的最两端的设备的RxD/TxD(+)和RxD/TxD(-)之间再加一个100Ω的电阻。

R485-2Wire总线并没有仲裁机制，所以，此组网一般是一主多从式应答机制，只允许一个设备主动向网络中发送请求，请求消息中包含目标从机的地址，所有从机都能收到请求，但是只有地址匹配的从机才会响应，主机的请求未收到响应，不能发送下一个请求，避免冲突。

RS422有5根引脚
RxD+
RxD-
TxD+
TxD-
GND



![image](https://img2024.cnblogs.com/blog/1037641/202403/1037641-20240328103134372-653362443.png)

![image](https://img2024.cnblogs.com/blog/1037641/202403/1037641-20240328103041687-257328242.png)

## RS232和RS485区别

RS485诞生于RS232之后，是针对RS232的缺点进行了改良后的串口实现。
## 通信距离
RS232的最大通信距离是15m,实际工程现场，建议不超过10m。RS485最大通信距离1km(180kbits/s)，实际工程现场，建议不超过500m。串口的通讯距离与通信速率是成反比的，RS485在1km的距离上传输数据时，为了保证数据的准确性，传输速率应控制在100-180kbps以下,100m时RS485的传输速率可达12Mbit/s。RS485可以使用中继器延长通信距离和增强通信可靠性。
## 通信速率
RS232的最大传输速率是20kbps，RS485的最大传输速率是10Mbps。
## 抗干扰性

RS485硬件设计的优越性，使其通信的抗干扰性比RS232强的多。

## 电气特性
RS232接口的信号电平值较高(±3-±15V)，易损坏接口电路的芯片，又因为与TTL电平不兼容故需使用电平转换电路（MAX232）方能与TTL电路连接。
RS-485接口的信号电平值较高(±2-±6V)。接口信号电平比RS-232-C降低了，就不易损坏接口电路的芯片，且该电平与TTL电平兼容，可方便与TTL电路直接连接。

## 通信模式
RS232是一对一的全双工通信。它仅仅支持两个设备一对一的通信模式，常见的应用场景是超市收银台的手持扫码枪。
工控机的串口一般是RS232，外设的串口如果也是RS232，可以直连。如果外设的串口是RS485，需要RS232转RS485模块。如果外设的串口是信号电平是TTL的串口(单片机)，需要一个MAX232转换电平。

![image](https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210903145429433-822644717.jpg)

![image](https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210901153216442-1802619139.jpg)

<img src="https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210903171956008-709520665.png" alt="image" style="zoom: 50%;" />

RS485是一对多的半双工通信。多个具有RS485接口的设备，可以以`手牵手`的方式串联到一起组网。由于RS485是半双工通信，总线上也没有仲裁机制，所以同一时间只能有一个设备在总线上发送数据，而网络上的其他设备都能同时接收到它发送的数据。因此典型的RS485组网通信模式是master-slaves一主多从模式，网络上的任意设备都可以作为主设备，但主设备通常是PC上的应用程序。RS485网络只负责将一个RS485设备的消息等价的传送到网络上其他RS485设备，这些RS485收到消息后按照一定的规则解析消息，根据消息中的地址辨别出是不是送给自己的消息而决定是否响应回复。这个规则是烧写进RS485的固件程序（如ModbusRTU）决定的，一般会为设备赋予一个地址，根据消息中的地址信息判定消息是不是给自己的，以及如何响应消息。RS485组网最常见的使用场景是：主设备轮询从设备，获取从设备的状态，即主设备主动发送请求，从设备答复，但从设备不会主动发送数据。

补充：RS485串联时采用`手牵手`方式，这样线路的干扰最低，杜绝使用星型，分叉等连接方式。

![image](https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210903171834245-1336715968.jpg)
![image](https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210903171844903-2107035515.jpg)
![image](https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210903171852951-1586771111.jpg)
RS485与RS232仅仅是通讯的物理协议（即接口标准）有区别，RS485是差分传输方式，RS232是单 端传输方式，但通讯程序没有太多的差别。PC机上已经配备有RS232，直接使用就行了，若使用 RS485通讯，只要在RS232端口上配接一个RS232转RS485的转换头就可以了，我们也可以购买PCI多串口卡，直接从工控机的PCI引出RS485接口，不需要修改程序。
<img src="https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210903171956008-709520665.png" alt="image" style="zoom: 67%;" />

# 串口的应用场景
## PC的RS232串口与单片机的串口通信
前提说明：PC的串口称为RS232，因为其电平特性是±3 - ±15V，符合RS232标准。单片机的串口虽然也是串口，但是不能称之为RS232,因为其电平特性是TTL ±3 - ±5.即，RS232，RS485，单片机TTL串口等只是按照串口协议的某种实现。

单片机有一个全双工的串行通讯口，所以单片机和电脑(ibm-pc机)之间可以方便地进行串口通讯。由于ibm-pc机串行口输出的是rs232电平，而单片机串口输出的是ttl电平，两者之间应有一个电平转换电路。本系统采用了专用芯片max232进行转换，如下图所示。

![image](https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210901153216442-1802619139.jpg)

图中，imp-pc机的下传命令和数据由ibm-pc机的txd端(rs-232电平)发送，经max232转换为ttl电平被at89c51串行口所接收。同样，at89c51的上传代码(ttl电平)由max232转换rs-232电平加以发送，经过ibm-pc机的rxd端，并由其内部的串行口变换为ttl电平。



# 选择交叉线还是直连线

## DB9
<div align="center">
<image src="https://img2024.cnblogs.com/blog/1037641/202402/1037641-20240225204342588-1645634031.png" height="30%" width="30%"></image>
</div>

<div align="center">DB-9公头串口卡</div>

---

<div align="center">
<image src="https://img2024.cnblogs.com/blog/1037641/202402/1037641-20240225205556739-1080841618.png" height="30%" width="30%"></image>
</div>

<div align="center">DB-9母头串口</div>

---

<div align="center">
<image src="https://img2024.cnblogs.com/blog/1037641/202402/1037641-20240225220925335-2006948463.png" height="30%" width="30%"></image>
</div>

<div align="center">公母头近景图</div>

---

<div align="center">
<image src="https://img2024.cnblogs.com/blog/1037641/202402/1037641-20240225221619667-794564866.png" height="35%" width="35%"></image>
</div>

<div align="center">公母头引脚序号及功能定义</div>

---

<div align="center">
<image src="https://img2023.cnblogs.com/blog/1037641/202309/1037641-20230910022622959-1819020060.png" height="30%" width="30%"></image>
</div>

---

<div align="center">
<image src="https://img2024.cnblogs.com/blog/1037641/202402/1037641-20240225072110724-2106801690.png" height="30%" width="30%"></image>
</div>

---

在实际工程应用中，DB9的9个引脚只用到5个，① RX 收，② TX 发，③ GND 地，④ RTS 告诉对方已方可接收数据(DO)，⑤ CTS 探知对方是否可接收数据(DI)。通信的双方必须彼此的RX<=>TX，CTS<=>RTS,GND<=>GND.

## 选择直连线还是交叉线？

选择串口线时，需要确定2个参数：①公母头或公公头或母母头 ②交叉或直连。目的是保证2台设备的串口RX、TX引脚交叉相连，即设备的收发器发送引脚与另一台设备的收发器接收引脚相连，GND相连。

确定步骤：
1. 确定长在外设身上的DB9是公头还是母头。
   假设一方是母头另一方是公头，那么需要公-母线。假设两方都是母头，那么需要公-公线。假设两方都是公头，那么需要母-母线。

2. 一方的2是RX,一方的2是TX，选直连线。一方的2是RX，另一方的2也是RX，选交叉线。一方的2是TX,另一方的2也是TX,选交叉线。一方是公头标准引脚定义，另一方也是公头标准引脚定义，选交叉线。一方是母头标准引脚定义，另一方也是母头标准引脚定义，选交叉线。一方是公头标准引脚定义，另一方是母头标准引脚定义，选直连线。一方是公头非标准引脚定义(2TX3RX)，另一方是母头标准引脚定义，选交叉线。一方是公头非标准引脚定义(2TX3RX)，另一方是公头标准引脚定义，选直连线。
`公公标准，母母标准选交叉；公母标准选直连。`

经过上面2个步骤，我们就可以确定需要的是哪种串口线了。

---

## 什么是交叉和直连线。

交叉和直连线两端可以同是公头或同是母头或公母头。直连的含义是一端的2接另一端的2，交叉的含义是一端的2接另一端的3。注意序号，公头和母头序号相反不一样，制作串口连接线时要认对两端的序号，别接错了。

`直连串口线和交叉串口线接线方式示意图`
<div align="center">
<image src="https://img2024.cnblogs.com/blog/1037641/202402/1037641-20240225083409573-264385586.jpg" height="27%" width="27%"></image>
</div>

---

<div align="center">
<image src="https://img2024.cnblogs.com/blog/1037641/202402/1037641-20240225161928910-1644030836.png" height="30%" width="30%"></image>
</div>

---

<div align="center">
<image src="https://img2024.cnblogs.com/blog/1037641/202402/1037641-20240225072017878-1532315964.png" height="27%" width="27%"></image>
</div>

如果是直连线，A端的2和B端的2功能相同，A端的2和设备的3相接，所以B端的2和设备的3功能相同，而B端2和另一设备的3相接，所以如果采用直连线，双方设备的2和2接，3和3接，那么双方设备的2必须一个是RX另一个是TX。同理，双方设备的2引脚功能定义相同，必须采用交叉线，比如双方设备都是标准母头时，必须采用交叉线。

`如果串口线本来就能正常使用，只是想再买根延长线接到原来的串口线上，那必然要买直连线。

引脚的编号次序是行规，方便工程师之间无障碍无歧义的沟通，公头的编号确定好，母头的编号也随着确定了。`编号的本质表达的是，公母头对插后，各自相同编号的引脚相接。假设公头的5脚后面是GND，母头的5脚后面是TXD，那么公母头对插后，通信双方的串行收发器实际上的连接方式是TXD和GND相接。

---

## RS232-DB9引脚分类

- 数据： 2、3

* 硬流控：1、4、6、7、8

- 地线：5

- 其他：9

## RS485和RS422无交叉直连概念

DB9的RS485定义。RS485常用的的半双工两线制的接口定义为，1-DATA-, 2-DATA+, 5-GND。因RS485的DATA+与DATA+对应，DATA-与DATA-一一对应，所以RS485的公母头不存在信号不一致的情况。

4线制的RS485有几种不同的命名方法：英式标识为　　TDA(-) 　TDB(+) 　RDA(-) 　RDB(+) 　GND ；美式标识为Y　　Z　　A　　B　　GND；



# 排查串口硬件故障参考手册

## 如何查看PC上有无串口

<table><td bgcolor=gray>方法一： 查看PC外观，是否有DB9接口</td></table>

------------

<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210903101423949-1612309564.png" width="30%" height="30%" title=""/></div>

------------

<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210903101503528-1310875677.png" width="30%" height="30%" title=""/></div>

------------

<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210830144944362-2072577104.png" width="30%" height="30%" title=""/></div>

------------


说明：
- 工控机上的串口一般都是RS232，很少有RS485。如果我们需要RS485接口，可以在订购工控机的时候，向供应商说明，供应商可以定制提供有RS485的工控机。当然我们也可以购买USB转RS485模块或者RS232转RS485模块扩展出RS485接口。
- 串口接口一般是DB9，很古老的设备上才会有DB25，DB25几乎已被淘汰。
- 针状称为公头，孔状称为母头。




<table><td bgcolor=gray>方法二： 设备管理器 → 端口(COM和LPT)</td></table>



<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210830145224644-1276165560.png" width="30%" height="30%" title=""/></div>

说明：不显示虚拟串口；如果PC无物理串口，则找不到端口(COM和LPT)这个节点。



<table><td bgcolor=gray>方法三：使用串口调试软件，查看软件列出的串口</td></table>

<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210830152342122-293168599.png" width="20%" height="20%"/></div>

说明：会列出虚拟串口和物理串口


<table><td bgcolor=gray>方法四：利用虚拟串口软件Virtual Serial Port Driver</td></table>

<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210903234039416-1849428433.png" width="30%" height="30%" title=""/></div>

说明：既能列出所有的虚拟串口和物理串口，又可以识别出串口是虚拟的，还是物理的。


<table><td bgcolor=gray>方法五：C# API</td></table>


```csharp
public static string[] GetPortNames();
```

## 如何识别主机上物理串口的COM号

<table><td bgcolor=gray>问题</td></table>

> 含有多个物理串口的PC，打开其设备管理器，能看到很多串口号，但是我们并不能知晓哪个物理串口对应哪个COM号。
>
> ![image](https://img2023.cnblogs.com/blog/1037641/202309/1037641-20230903000031153-308553336.png)

---

<table><td bgcolor=gray>方法1：短接RXD和TXD</td></table>

选择某个物理串口，使用导线短接其2脚和3脚(2是RXD,3是TXD)，然后使用串口调试软件依次打开每个COM号，发送数据。如果能在软件的收消息栏看到收到消息，则该COM号就是此物理串口。

---

<table><td bgcolor=gray>方法2：使用电压表测量TXD的电压变化</td></table>

使用万用表，拨到直流电压档，一端接TXD(3脚)，一端接地(5脚),然后使用串口调试软件依次打开每个COM号，发送数据。如果万用表的数值发生明显跳变，则该COM号就是此物理串口。
原理就是发送数据时，发送引脚的电压会不断变化。

1. 万用表接的是发送引脚TXD，不是接收引脚RXD.

2. RS232 逻辑0电平范围 [-15V,-3V]，逻辑1电平范围[+3V, +15V]。 RS485逻辑0电平范围[-6V , -2V],逻辑1电平范围[+2V , +6V]。 在测量时，根据DB9的类型选择最小的量程，量程越小，数值变化越明显，容易观察到变化。

3. UART协议规定，当总线处于空闲状态时信号线的状态为‘1’即高电平，表示当前线路上没有数据传输。为了让低电平持续时间长点，建议以最小的波特率(2400bps)发送数据，无停止位(大部分串口调试软件的停止位选项无None选项，所以会被迫选择1，其实这样做并不好，因为停止位是高电平逻辑1)，无校验或恒0校验(SPACE)，且以16进制形式发送多个数字0。
假设波特率是2400，恒0校验，停止位是1，发送200个0，则电平图如下：

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202309/1037641-20230904225639641-1194419864.png" width="30%" height="30%" title=""/></div>

电平0维持时长：1000 / 2400 * 200 * 10 = 832ms
电平1维持时长：1000 / 2400 * 200 * 1 = 83.2ms

未发送数据时，电压表显示15V，发送数据中显示-15V，时长约0.8秒，数据发送完毕后又变回15V。

## 如何诊断工控机的物理串口是否已经损坏
如果工控机上的串口已经损坏，我们上位机程序写的再怎么6，也不可能正常控制串口外设的。

诊断方法：使用导线短接串口的RXD和TXD，通过串口调试助手打开此串口，若能自发自收，则串口正常，否则已损坏。

<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210908194713957-1407493489.png" width="30%" height="30%" title=""/></div>

`禁止带电插拔串口，这样很容易损坏串口。请至少关闭通信的其中一方电源后再插拔。`

## 如何判断串口连接线是直连还是交叉的以及是否损坏
<table><td bgcolor=gray>判断交叉还是直连</td></table>

万用表拨到通断挡位，红表笔接串口线一端DB9的2号针脚，黑表笔接触另外一端的2号针脚，如果蜂鸣器响，红灯亮，则表示2与2通，是直连线，否则是交叉线。
当然，也可以使用3号引脚进行判断。

`串口线分为公-公，母-母，公-母；使用万用表测试时，注意辨别公母头的2号针脚，别把表笔怼错位置了。公头自左至右第2个，母头自右至左第2个。`

![image](https://img2023.cnblogs.com/blog/1037641/202309/1037641-20230903012302764-162578836.png)

`在判断串口线是直连还是交叉时，无需考虑DB9的第2针脚是RX还是TX，直接找第2个针脚就行了！`

<table><td bgcolor=gray>诊断串口线是否正常</td></table>

万用表拨到通断挡位，测试连接线两端的DB9：

2和2通，3和3通，2和3不通，5和5通，正常的直连线。

2和3通，3和2通，2和2不通，3和3不通，5和5通，正常的交叉线。

其他情况串口线已损坏。

**注意：公头和母头的2和3针脚顺序不同，公头左2是2，母头右2是2，不要测错针脚。**



# USB转串口和虚拟串口VSPD

RS232转RS485并不能提高通信速率，因为通信链路的最大速度取决于最慢的一环。

九针串口体积大，导致个人笔记本无法做的轻薄，另外，绝大多数人并用不到串口，所以，现代笔记本几乎都不带串口。

<div align="center"><img src="https://img2023.cnblogs.com/blog/1037641/202309/1037641-20230910005038182-94370941.png" width="20%" height="20%" title=""/></div>

没有物理串口怎么调试我们写好的串口程序呢？我们可以利用USB转串口(RS232或RS485)模块扩展出物理串口或者利用VSPD虚拟出模拟串口调试串口程序。
1. USB转串口

<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210830152959837-2098858051.png" width="25%" height="25%" title=""/></div>

<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210830153753005-761096193.png" width="25%" height="25%" title=""/></div>

[USB转串口驱动下载地址](https://www.lulian.cn/download/list-108-cn.html)
我们同时需要安装USB转串口驱动程序，如果未安装或安装不合适的驱动，USB转串口即使插上PC也无法被识别和使用。我们在哪个商家那里购买的USB转串口模块，就像哪个商家索要相应的驱动程序，一般会给个包含驱动程序的小光盘。

<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210830153123865-1343701530.png" width="25%" height="25%" title=""/></div>

2. 虚拟串口
利用虚拟串口软件Virtual Serial Port Driver,为PC增加虚拟的串口。这种方式是我们调试串口程序最常用的方法。

<div align="center"><img src="https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210830153631755-1823236299.png" width="25%" height="25%" title=""/></div>



# 硬件流控和软件流控

## 流控的概念和目的

接收方接收线程监测数据线，把发送方发来的数据先放入到自己的接收缓存以待稍后处理。

如果接收方处理不过来，来不及从缓存中取出数据，而发送方持续发来数据，接收方只能强行接收数据放入已满的缓存覆盖掉之前接收但还未处理的数据，这就造成了消息丢失。

如果接收方处理不过来，接收缓存不够用时，反馈到发送方，发送方便暂停发送数据，待接收方能够处理新数据时再向发送方反馈您可以继续向我发送数据了。这样就不会导致消息丢失了，上述协调发送方的发送速度和接收方的接收速度机制就叫流控。

1. 流控是控制发送，与接收无关。
2. 实施流控，接收方需有反馈自身闲忙的功能，发送方在发送前需要检查接收方的闲忙反馈信号。
3. 不实施流控，可能导致接收方缓存被覆盖而导致数据丢失。

异步串行通信的流控方式分为硬件流控和软件流控。`RS232支持硬件流控，也支持软件流控；RS485和RS422的接线方式无控制线，仅有数据线，所以只支持软件流控，不支持硬件流控`。

RS232只接RX，TX，GND，则无硬件流控；RX，TX，GND，CTS，RTS则可以硬件流控。如下图，CTS和RTS交叉连接实现RS232的硬件流控。RTS是输出，CTS是输入，发送前检查CTS，若有效则发送，否则不发送。当自身接收缓存已满后，置位RTS无效，缓存空出后，置位RTS有效，即表示我可以接收数据了，你可以发送了。

![image](https://img2023.cnblogs.com/blog/1037641/202311/1037641-20231113000947428-1756496285.png)


## 软件流控（XON、XOFF）

软件流控是指在消息帧中使用特殊字符来实现流通量控制，不需要接控制线，发送方只需要在硬件组态中设置 XON/XOFF 字符；而接收方通过发送 XON/XOFF 字符给发送方即可实现数据传输控制。

- 表示可以继续传送的 ASCII 字符称为 **XON**
- 表示传送必须停止的 ASCII 字符称为 **XOFF**

![image](https://img2023.cnblogs.com/blog/1037641/202311/1037641-20231113001001092-1598379964.png)


当通信一方不能继续接收数据时，会发送一个 XOFF 字符到通信的另一方，告诉另一方停止发送数据；当通信可以恢复时，该方会再发送一个 XON 字符到另一方，告诉对方继续发送数据。如果 CM1241 模块在指定的时间内没有等待到 XON 字符，则终止数据传输并返回错误到用户程序。

**![img](https://www.ad.siemens.com.cn/productportal/Prods/S7-1200_PLC_EASY_PLUS/11-Comm/03-Serial/01-PTP/images/4.gif)注意：**在通信数据中，不得包含 XON 和 XOFF 字符



# SerialPort



**属性默认值**

BaudRate : 9600
BreakState : False
BytesToWrite : 0
BytesToRead : 0
CDHolding : False
CtsHolding : False
DataBits : 8
DiscardNull : False
DsrHolding : False
DtrEnable : False
Encoding : System.Text.ASCIIEncoding+ASCIIEncodingSealed
Handshake : None
IsOpen : True
NewLine :

Parity : None
ParityReplace : 63
PortName : COM1
ReadBufferSize : 4096
ReadTimeout : -1
ReceivedBytesThreshold : 1
RtsEnable : False
StopBits : One
WriteBufferSize : 2048
WriteTimeout : -1
Site :
Container :




## 获取PC上所有的串口名

```csharp
public static string[] GetPortNames();
```

## 构造函数（设置串口参数）

```csharp
public SerialPort(string portName, int baudRate, Parity parity, int dataBits, StopBits stopBits);
```

**BaudRate**

默认值是9600.

The baud rate must be supported by the user's serial driver. The default value is 9600 bits per second (bps).
波特率并不是随便任意设置成1个正整数都可以，而是必须通信双方的串口驱动程序都支持的整数，常用的有9600，19200，38400，115200等。


**Parity**

None, Even, Odd, Mark, Space.

默认值是None。

开启校验后，如果检测到串口帧发生校验错误，这个串口帧的数据位会被替换成字节`ParityReplace`，ParityReplace的默认值是63(ASCII字符是 `?` )，如果设置SerialPort.ParityReplcae = 0x00，则不会替换，仍旧原样提取错误帧中的数据位数据。

因为串口通信的奇偶校验本身并不可靠，对于数据准确性至关重要的场景下，通常会设置None，不开启奇偶校验，这样属性ParityReplace也变得无用了。

**DataBits**

默认值是8.
Range: from 5 through 8 (5,6,7,8)

假设数据位设置成是5。发送方想发送二进制串 11111111111(B)，步骤：

1. 先切割，5bit一份，结果是  11111  11111  1，需发送3个串口帧。
2. 构建发送字节数组，SerialPort.Write(new byte[] { 0bxxx11111, obxxx11111, obxxxxxxx1  })
3. 接收方接到3个帧，高权位补0


**StopBits**

One,OnePointFive,Two.

Caution：枚举值虽然有None选项，但是不可以将参数设置成None，它不受支持,执行serialPort.StopBits = StopBits.None会抛出如下异常：
System.ArgumentOutOfRangeException: '枚举值超出合法范围。
Parameter name: StopBits'

```txt
You use this enumeration when setting the value of the StopBits property on the SerialPort class. Stop bits separate each unit of data on an asynchronous serial connection. They are also sent continuously when no data is available for transmission.

The SerialPort class throws an ArgumentOutOfRangeException exception when you set the StopBits property to None.
```

## 判断串口打开/关闭

```csharp
public bool IsOpen { get; }
```

## 打开串口

```csharp
public void Open();
```
微软官方建议，Close()后，最好延时若干时间后再打开此串口。

## 关闭串口

```csharp
public void Close();
```
1. Close()内部也会调用Dispose()方法，所以SerialPort.Close()后不能再次Open()，只能重新New一个SerialPort实例来操纵串口。
2. Close()内部会调用DiscardInBuffer和DiscardOutBuffer

## 发送数据

```csharp
public void Write(byte[] buffer, int offset, int count);
public void Write(char[] buffer, int offset, int count);
public void Write(string text);
public void WriteLine(string text);
```
WriteString和WriteChar本质是先把字符使用`Encoding`字符集编码转换成字节数组然后通过WriteByte发送出去。
WriteLine会在参数text后自动追加`NewLine`一并发送出去。
`Encoding`默认值是ASCII，

## 读取数据

```csharp
public int Read(byte[] buffer, int offset, int count);
public int Read(char[] buffer, int offset, int count);
```

将接收缓存中的字节读出来，放到数组buffer中。
若接收缓存中的字节数量不小于count，则读取count个，若小于count，则有多少读取多少，实际读到的数量通过返回值知晓，返回值表示本次Read读取到的实际字节数量。读取到的字节从buff索引offset处开始放。
本API是阻塞型API，如果count等于0，则直接返回，不可能发生阻塞；如果count大于0，如果缓存中有字节，即使是1个或其他任何小于count的正整数，Read也立刻读取返回，如果缓存中一直没有任何字节，则Read会阻塞`ReadTimeout`毫秒后抛出异常TimeoutException，但是如果ReadTimeout是默认值-1表示永不超时，则Read会一直阻塞下去。

```csharp
public string ReadTo (string value);
```
从缓存中读取字节，一边读取一边解码，直到读到字符串value后方法返回。如果一直读不到value，则ReadTo一直被阻塞，直到抛出TimeoutException.
示例：接收方，string str = ReadTo("F"); 发送方发送"ABCDEFGHIJK"; 返回值str是ABCDE,接收缓存剩余GHIJK. 注意，返回值和缓存剩余都不包含F!

## 发送数据

```csharp
public void Write (string text);
public void Write (byte[] buffer, int offset, int count); // 从buffer的offset位置开始截取count个字节进行发送。
public void Write (char[] buffer, int offset, int count);
```


## 编码方式
```csharp
public Encoding Encoding { get; set; }
```
此属性，决定ReadChar,ReadTo(string value),Write(string text),Write(char[] buffer, int offset, int count)采用的编码方式。

## 发送缓存和接收缓存
ReadBufferSize，接收缓存的大小，单位字节，默认值是4096,如果应用程序为该属性指定小于4096的值会被忽略，也就是说ReadBufferSize的最小值是4096.
WriteBufferSize，发送缓存的大小，单位字节，默认值是2048，如果应用程序为该属性指定小于2048的值会被忽略，也就是说WriteBufferSize的最小值是2048.

### 超时设置

```csharp
public int ReadTimeout { get; set; }
public int WriteTimeout { get; set; }
```
当我们调用读或写方法，如果在指定的超时时间内没有从接收缓存读到数据或者将数据发送到发送缓存，那么读写方法就会抛出超时异常。


### 串口收到数据触发的事件

```csharp
public event SerialDataReceivedEventHandler DataReceived;
```

### 接收到多少字节触发一次DataReceived事件

```csharp
public int ReceivedBytesThreshold { get; set; }
```

## BytesToRead
SerialPort还未读取到内存但是已经接收到的字节数量。
The receive buffer includes the serial driver's receive buffer as well as internal buffering in the [SerialPort](https://learn.microsoft.com/en-us/dotnet/api/system.io.ports.serialport?view=dotnet-plat-ext-7.0) object itself.

### 清空接收和发送缓存

```csharp
public void DiscardInBuffer(); // 清空接收缓存（Received Buffer）
public void DiscardOutBuffer(); // 清空发送缓存（Transmit Buffer）
```

| Open()后Close()前才能调用的API | Open()前调用Open()后就不要调用的API |
| ------------------------------ | ----------------------------------- |
| DiscardInBuffer                |                                     |
| DiscardOutBuffer               |                                     |

****



****



## 流控相关属性(😄)
```csharp
public bool DtrEnable { get; set; }
public bool RtsEnable { get; set; }
public bool CtsHolding { get; }
public bool DsrHolding { get; }
public Handshake Handshake { get; set; }
```

这5个属性在串口编程中几乎永远用不到，但我们解释下这5个属性，可以选择不看。
上世纪串口实现使用了RxD，TxD两根数据收发线，一根地线，若干根控制线。随着IT技术的日新月异，现在几乎不再使用控制线。而上述的5个属性正是对控制线的抽象，所以这些属性也用不到了，这些属性的默认值一般都是false，None,即禁用，不使用的含义，所以在串口编程中不理这些属性就OK了。
当然，你可能会问什么时候使用这些属性？
串口驱动程序支持控制线的话，我们连接两个串口时必须也要把控制线接好。如果串口驱动程序不支持控制线的话，那么我们只需要接好RxD,TxD,Ground三根线即可,控制线可以不接。
我们根据被控制的串口设备所使用的驱动程序是否支持控制线，来设置DtrEnable，RtsEnable，Handshake的值。但一般都未使用控制线。

## 什么是流控
Handshake指流控，也称为握手。通信双方发送数据要考虑对方的处理数据能力，接收方处理不过来发送方就要先暂停发送一段时间，这就是流控。流控可以通过控制线实现，称为硬流控；流控通过软件实现，称为软流控。现代串口通信一般也不使用流控机制。



# SerialDeviceBase

**适用场景**

1. Request - Reply  请求-响应模型。 发送一条请求，必须收到一条且仅有一条响应。

**使用限制**

1. 上一条请求未收到响应，不会发送下一条

**使用方法**

继承SerialDeviceBase实现其抽象方法`protected abstract bool IsExpectedReply(byte[] bytesHasRead, byte[] requestBytes, int bytesLengthToRead, int loopTimes)`即可。

---

<details>
  <summary>SerialDeviceBase 源码</summary>


```csharp
public abstract class SerialDeviceBase : IDisposable
{
    #region Fileds

    private static readonly byte[] EmptyByteArray = Array.Empty<byte>();

    // ReSharper disable once IdentifierTypo
    private readonly object _transmitlocker = new();

    private readonly object _openPortLocker = new();

    #region Reset These Variables Before Transmition

    private readonly AutoResetEvent _waitReplyEvent;
    private readonly Stopwatch _stopWatch;
    private int _waitReplyTimeoutMilliseconds;
    private DateTime? _lastSendTime;
    private string _errorContent;
    private byte[] _receivedData;
    private byte[] _requestBytes;

    #endregion

    private readonly SerialPort _port;

    #endregion

    #region Properties

    public string PortName { get; }
    public int BaudRate { get; }
    public Parity Parity { get; }
    public StopBits StopBits { get; }
    public int DataBits { get; }
    public int SingleSerialFrameTransmissionMilliseconds => (int)Math.Ceiling(9600d / BaudRate * 2);
    public int IntervalBetweenSendingRequest { get; set; }

    #endregion

    // 单链接，长链接
    protected SerialDeviceBase(string portName, int baudRate, Parity parity, StopBits stopBits, int dataBits = 8)
    {
        PortName = portName;
        BaudRate = baudRate;
        Parity = parity;
        StopBits = stopBits;
        DataBits = dataBits;
        _port = new SerialPort();
        _port.PortName = portName;
        _port.BaudRate = baudRate;
        _port.DataBits = dataBits;
        _port.Parity = parity;
        _port.StopBits = stopBits;
        _port.Handshake = Handshake.None;
        _port.RtsEnable = false;
        _port.DtrEnable = false;
        _port.DataReceived += _port_DataReceived;
        _errorContent = string.Empty;
        _receivedData = EmptyByteArray;
        _receivedData = EmptyByteArray;
        _requestBytes = EmptyByteArray;
        _lastSendTime = null;
        _stopWatch = new Stopwatch();
        _waitReplyEvent = new AutoResetEvent(false);
    }

    protected abstract bool IsExpectedReply(byte[] bytesHasRead, byte[] requestBytes, int bytesLengthToRead, int loopTimes);

    protected (byte[] reply, string errorContent) Send(byte[] bytes, int waitCompleteReplyTimeoutMilliseconds = 500)
    {
        try
        {
            Open();
            lock (_transmitlocker)
            {
                if (_lastSendTime != null)
                {
                    if (IntervalBetweenSendingRequest > 0)
                    {
                        double interval = (DateTime.Now - _lastSendTime.Value).TotalMilliseconds;
                        if (interval < IntervalBetweenSendingRequest)
                        {
                            Thread.Sleep((int)Math.Ceiling(IntervalBetweenSendingRequest - interval));
                        }
                    }
                }

                _errorContent = string.Empty;
                _receivedData = EmptyByteArray;
                _waitReplyTimeoutMilliseconds = waitCompleteReplyTimeoutMilliseconds;
                _waitReplyEvent.Reset();
                _requestBytes = bytes.ToArray();
                _port.Write(bytes, 0, bytes.Length);
                _lastSendTime = DateTime.Now;
                _stopWatch.Restart();
                bool timeout = !_waitReplyEvent.WaitOne(_waitReplyTimeoutMilliseconds);
                byte[] returnReceivedData;
                string returnErrorContent;
                if (timeout == false)
                {
                    returnErrorContent = _errorContent;
                    returnReceivedData = _receivedData.ToArray();
                }
                else if (timeout && _receivedData.Length <= 0)
                {
                    returnErrorContent = "Timed out because no reply data was received";
                    returnReceivedData = EmptyByteArray;
                }
                else
                {
                    _waitReplyEvent.WaitOne(_waitReplyTimeoutMilliseconds);
                    returnErrorContent = _errorContent;
                    returnReceivedData = _receivedData.ToArray();
                }
                return (returnReceivedData, returnErrorContent);
            }
        }
        catch (Exception e)
        {
            return (_receivedData.ToArray(), e.ToString());
        }
    }

    private void _port_DataReceived(object sender, SerialDataReceivedEventArgs e)
    {
        // 必须保证此方法在有限的时间内能返回，也就是必须要有超时机制，否则Close不掉串口。

        var port = (SerialPort)sender;
        try
        {
            // 这个判断很有必要， 因为可能多个线程池线程在同时执行DataReceived事件， 但是只有1个线程拿到锁，其他的线程在排队。 拿到锁的线程把Buffer中的数据读取完毕结束后， 唤醒1个等待线程，等待线程第1步先判断有无可读数据，无直接返回， 这样效率比较高。
            if (port.BytesToRead > 0)
            {
                var buff = new List<byte>();
                int i = 0;
                bool expected;
                do
                {
                    var temp = new byte[port.BytesToRead];
                    // 注意Read不能死等 不要在其他线程轻易调用DiscardIn/OutBuff,会造成Read死等
                    int count = port.Read(temp, 0, temp.Length);
                    // Read阻塞型API. 直到缓冲区至少有1个字节可读便返回，如果长时间缓冲区为0，则一直阻塞，直到超时抛出TimeoutException,如果Timeout值是-1，则一直阻塞。 temp.Length - offset 如果是0，表示不读取，任何情况下Read都会立刻返回，不存在阻塞情况。
                    if (count > 0)
                    {
                        buff.AddRange(temp.Take(count));
                        _receivedData = buff.ToArray();
                    }

                    expected = IsExpectedReply(buff.ToArray(), _requestBytes.ToArray(), port.BytesToRead, i++);
                    if (expected)
                    {
                        break;
                    }

                    if (_waitReplyTimeoutMilliseconds > 0 && _stopWatch.ElapsedMilliseconds > _waitReplyTimeoutMilliseconds)
                    {
                        _errorContent = "Timeout! Although some data has been received, it cannot be resolved into a completed and valid reply frame.";
                        break;
                    }
                } while (expected == false);
            }
        }
        catch (Exception exception)
        {
            _errorContent = exception.ToString();
        }
        finally
        {
            _stopWatch.Stop();
            _waitReplyEvent.Set();
        }
    }

    private void Open()
    {
        // ReSharper disable once InconsistentlySynchronizedField
        if (!_port.IsOpen)
        {
            lock (_openPortLocker)
            {
                if (!_port.IsOpen)
                {
                    _port.Open();
                }
            }
        }
    }

    #region Disposable

    private void ReleaseUnmanagedResources()
    {
        // TODO release unmanaged resources here
    }

    protected virtual void Dispose(bool disposing)
    {
        ReleaseUnmanagedResources();
        if (disposing)
        {
            _port.DataReceived -= _port_DataReceived;
            _waitReplyEvent.Dispose();
            _port.Dispose();
        }
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    ~SerialDeviceBase()
    {
        Dispose(false);
    }

    #endregion
}
```

</details>



# Winform关闭串口界面卡死原因分析(转载)

![image](https://img2023.cnblogs.com/blog/1037641/202311/1037641-20231109231938779-78688515.png)


[原文地址](https://www.cnblogs.com/timefiles/p/CsharpSerialPortDeadlockOnClose.html "原文地址")
<div id="cnblogs_post_body" class="blogpost-body cnblogs-markdown">
<p></p><div class="toc"><div class="toc-container-header">目录</div><ul><li><a href="#问题描述">问题描述</a></li><li><a href="#查找原因">查找原因</a><ul><li><a href="#serialport类open方法">SerialPort类Open()方法</a></li><li><a href="#serialport类close方法">SerialPort类Close()方法</a></li></ul></li><li><a href="#死锁原因">死锁原因</a></li><li><a href="#解决死锁">解决死锁</a></li><li><a href="#总结">总结</a></li></ul></div><p></p>
<h1 id="问题描述">问题描述<a href="#问题描述" class="esa-anchor" style="opacity: 0;">#</a></h1>
<p>前几天用SerialPort类写一个串口的测试程序，关闭串口的时候会让界面卡死。<br>
参考博客<a href="https://blog.csdn.net/xufei4987/article/details/81174963" target="_blank">windows程序界面卡死的原因</a>，得出界面卡死原因：<strong>主线程和其他的线程由于资源或者锁争夺，出现了死锁。</strong></p>
<p>参考知乎文章<a href="https://www.zhihu.com/question/274949644" target="_blank">WinForm界面假死，如何判断其卡在代码中的哪一步？</a>，通过<strong>点击调试暂停，查看ui线程函数栈，直接定位阻塞代码的行数</strong>，确定问题出现在SerialPort类的Close()方法。</p>
<p>参考文章<a href="https://blog.csdn.net/wuyazhe/article/details/5606276" target="_blank">C# 串口操作系列(2) -- 入门篇，为什么我的串口程序在关闭串口时候会死锁 ？</a>文章的解决方法和网上的大部分解决方法类似：<strong>定义2个bool类型的标记Listening和Closing，关闭串口和接受数据前先判断一下</strong>。我个人并不太接受这种方法，感觉还有更好的方式，而且文章讲述的也并不太清楚。</p>
<h1 id="查找原因">查找原因<a href="#查找原因" class="esa-anchor" style="opacity: 0;">#</a></h1>
<p>基于刨根问底的原则，我继续查找问题发生的原因。<br>
先看看导致界面卡死的代码：</p>
<pre><code class="language-csharp hljs hljsln"><span class="ln-bg"></span><span class="ln-num" data-num="1"></span><span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">comm_DataReceived</span>(<span class="hljs-params"><span class="hljs-built_in">object</span> sender, SerialDataReceivedEventArgs e</span>)</span>   
<span class="ln-num" data-num="2"></span>{   
<span class="ln-num" data-num="3"></span>    <span class="hljs-comment">//获取串口读取的字节数</span>
<span class="ln-num" data-num="4"></span>    <span class="hljs-built_in">int</span> n = comm.BytesToRead;    
<span class="ln-num" data-num="5"></span>    <span class="hljs-comment">//读取缓冲数据  </span>
<span class="ln-num" data-num="6"></span>    comm.Read(buf, <span class="hljs-number">0</span>, n);       
<span class="ln-num" data-num="7"></span>    <span class="hljs-comment">//因为要访问ui资源，所以需要使用invoke方式同步ui。   </span>
<span class="ln-num" data-num="8"></span>    <span class="hljs-keyword">this</span>.Invoke(<span class="hljs-keyword">new</span> Action(() =&gt;{...界面更新，略})); 
<span class="ln-num" data-num="9"></span>}   
<span class="ln-num" data-num="10"></span>  
<span class="ln-num" data-num="11"></span><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">buttonOpenClose_Click</span>(<span class="hljs-params"><span class="hljs-built_in">object</span> sender, EventArgs e</span>)</span>   
<span class="ln-num" data-num="12"></span>{   
<span class="ln-num" data-num="13"></span>    <span class="hljs-comment">//根据当前串口对象，来判断操作   </span>
<span class="ln-num" data-num="14"></span>    <span class="hljs-keyword">if</span> (comm.IsOpen)   
<span class="ln-num" data-num="15"></span>    {   
<span class="ln-num" data-num="16"></span>        <span class="hljs-comment">//打开时点击，则关闭串口   </span>
<span class="ln-num" data-num="17"></span>        comm.Close();<span class="hljs-comment">//界面卡死的原因</span>
<span class="ln-num" data-num="18"></span>    }   
<span class="ln-num" data-num="19"></span>    <span class="hljs-keyword">else</span>  
<span class="ln-num" data-num="20"></span>    {...}  
<span class="ln-num" data-num="21"></span>}
<span class="ln-eof"></span></code></pre>
<p>问题就出现在上面的代码中，原理目前还不明确，我只能参考.NET源码来查找问题。</p>
<h2 id="serialport类open方法">SerialPort类Open()方法<a href="#serialport类open方法" class="esa-anchor" style="opacity: 0;">#</a></h2>
<p>SerialPort类Close()方法的源码如下：</p>
<pre><code class="language-csharp hljs hljsln"><span class="ln-bg"></span><span class="ln-num" data-num="1"></span>		<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">Open</span>(<span class="hljs-params"></span>)</span>
<span class="ln-num" data-num="2"></span>        {
<span class="ln-num" data-num="3"></span>           <span class="hljs-comment">//省略部分代码...</span>
<span class="ln-num" data-num="4"></span>            internalSerialStream = <span class="hljs-keyword">new</span> SerialStream(portName, baudRate, parity, dataBits, stopBits, readTimeout,
<span class="ln-num" data-num="5"></span>                writeTimeout, handshake, dtrEnable, rtsEnable, discardNull, parityReplace);
<span class="ln-num" data-num="6"></span> 
<span class="ln-num" data-num="7"></span>            internalSerialStream.SetBufferSizes(readBufferSize, writeBufferSize); 
<span class="ln-num" data-num="8"></span>            internalSerialStream.ErrorReceived += <span class="hljs-keyword">new</span> SerialErrorReceivedEventHandler(CatchErrorEvents);
<span class="ln-num" data-num="9"></span>            internalSerialStream.PinChanged += <span class="hljs-keyword">new</span> SerialPinChangedEventHandler(CatchPinChangedEvents);
<span class="ln-num" data-num="10"></span>            internalSerialStream.DataReceived += <span class="hljs-keyword">new</span> SerialDataReceivedEventHandler(CatchReceivedEvents);
<span class="ln-num" data-num="11"></span>        } 
<span class="ln-eof"></span></code></pre>
<p>每次执行SerialPort类Open()方法都会出现实例化一个SerialStream类型的对象，并将CatchReceivedEvents事件处理程序绑定到SerialStream实例的DataReceived事件。</p>
<p>SerialStream类CatchReceivedEvents方法的源码如下：</p>
<pre><code class="language-csharp hljs hljsln"><span class="ln-bg"></span><span class="ln-num" data-num="1"></span>        <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">CatchReceivedEvents</span>(<span class="hljs-params"><span class="hljs-built_in">object</span> src, SerialDataReceivedEventArgs e</span>)</span>
<span class="ln-num" data-num="2"></span>        {
<span class="ln-num" data-num="3"></span>            SerialDataReceivedEventHandler eventHandler = DataReceived;
<span class="ln-num" data-num="4"></span>            SerialStream stream = internalSerialStream;
<span class="ln-num" data-num="5"></span> 
<span class="ln-num" data-num="6"></span>            <span class="hljs-keyword">if</span> ((eventHandler != <span class="hljs-literal">null</span>) &amp;&amp; (stream != <span class="hljs-literal">null</span>)){
<span class="ln-num" data-num="7"></span>                <span class="hljs-keyword">lock</span> (stream) {
<span class="ln-num" data-num="8"></span>                    <span class="hljs-built_in">bool</span> raiseEvent = <span class="hljs-literal">false</span>;
<span class="ln-num" data-num="9"></span>                    <span class="hljs-keyword">try</span> {
<span class="ln-num" data-num="10"></span>                        raiseEvent = stream.IsOpen &amp;&amp; (SerialData.Eof == e.EventType || BytesToRead &gt;= receivedBytesThreshold);    
<span class="ln-num" data-num="11"></span>                    }
<span class="ln-num" data-num="12"></span>                    catch {
<span class="ln-num" data-num="13"></span>                        <span class="hljs-comment">// Ignore and continue. SerialPort might have been closed already! </span>
<span class="ln-num" data-num="14"></span>                    }
<span class="ln-num" data-num="15"></span>                    <span class="hljs-keyword">finally</span> {
<span class="ln-num" data-num="16"></span>                        <span class="hljs-keyword">if</span> (raiseEvent)
<span class="ln-num" data-num="17"></span>                            eventHandler(<span class="hljs-keyword">this</span>, e);  <span class="hljs-comment">// here, do your reading, etc. </span>
<span class="ln-num" data-num="18"></span>                    }
<span class="ln-num" data-num="19"></span>                }
<span class="ln-num" data-num="20"></span>            }
<span class="ln-num" data-num="21"></span>        }
<span class="ln-num" data-num="22"></span> 
<span class="ln-eof"></span></code></pre>
<p>可以看到SerialStream类CatchReceivedEvents方法触发自身的DataReceived事件，这个DataReceived事件就是我们处理串口接收数据的用到的事件。</p>
<p><strong>DataReceived事件处理程序是在lock (stream) {...}块中执行的，ErrorReceived 、PinChanged 也类似。</strong></p>
<h2 id="serialport类close方法">SerialPort类Close()方法<a href="#serialport类close方法" class="esa-anchor">#</a></h2>
<p>SerialPort类Close()方法的源码如下：</p>
<pre><code class="language-csharp hljs hljsln"><span class="ln-bg"></span><span class="ln-num" data-num="1"></span>		<span class="hljs-comment">// Calls internal Serial Stream's Close() method on the internal Serial Stream.</span>
<span class="ln-num" data-num="2"></span>        <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">Close</span>(<span class="hljs-params"></span>)</span>
<span class="ln-num" data-num="3"></span>        {
<span class="ln-num" data-num="4"></span>            Dispose();
<span class="ln-num" data-num="5"></span>        }
<span class="ln-num" data-num="6"></span>        
<span class="ln-num" data-num="7"></span>        <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">Dispose</span>(<span class="hljs-params"></span>)</span> {
<span class="ln-num" data-num="8"></span>            Dispose(<span class="hljs-literal">true</span>);
<span class="ln-num" data-num="9"></span>            GC.SuppressFinalize(<span class="hljs-keyword">this</span>);
<span class="ln-num" data-num="10"></span>        }
<span class="ln-num" data-num="11"></span>        <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">override</span> <span class="hljs-keyword">void</span> <span class="hljs-title">Dispose</span>(<span class="hljs-params"> <span class="hljs-built_in">bool</span> disposing </span>)</span>
<span class="ln-num" data-num="12"></span>        {
<span class="ln-num" data-num="13"></span>            <span class="hljs-keyword">if</span>( disposing ) {
<span class="ln-num" data-num="14"></span>                <span class="hljs-keyword">if</span> (IsOpen) {
<span class="ln-num" data-num="15"></span>                    internalSerialStream.Flush();
<span class="ln-num" data-num="16"></span>                    internalSerialStream.Close();
<span class="ln-num" data-num="17"></span>                    internalSerialStream = <span class="hljs-literal">null</span>;
<span class="ln-num" data-num="18"></span>                }
<span class="ln-num" data-num="19"></span>            }
<span class="ln-num" data-num="20"></span>            <span class="hljs-keyword">base</span>.Dispose( disposing );
<span class="ln-num" data-num="21"></span>        }        
<span class="ln-eof"></span></code></pre>
<p>可以看到，执行Close()方法最终会调用Dispose( bool disposing )方法。<br>
微软SerialPort类对父类的Dispose( bool disposing )方法进行了重写，在执行base.Dispose( disposing )前会执行internalSerialStream.Close()方法，也就是说<strong>SerialPort实例执行Close()方法时会先关闭SerialPort实例内部的SerialStream实例，再执行父类的Close()操作</strong>。</p>
<p>base.Dispose( disposing )方法不作为重点，我们再看internalSerialStream.Close()方法。</p>
<p>SerialStream类源码没有找到Close()方法，说明没有重写父类的Close方法，直接看父类的Close()方法，源码如下：</p>
<pre><code class="language-csharp hljs hljsln"><span class="ln-bg"></span><span class="ln-num" data-num="1"></span>		<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">virtual</span> <span class="hljs-keyword">void</span> <span class="hljs-title">Close</span>(<span class="hljs-params"></span>)</span>
<span class="ln-num" data-num="2"></span>        {
<span class="ln-num" data-num="3"></span>            Dispose(<span class="hljs-literal">true</span>);
<span class="ln-num" data-num="4"></span>            GC.SuppressFinalize(<span class="hljs-keyword">this</span>);
<span class="ln-num" data-num="5"></span>        }        
<span class="ln-eof"></span></code></pre>
<p>SerialStream父类的Close方法调用了Dispose(true)，不过SerialStream类重写了父类的Dispose(bool disposing)方法，源码如下：</p>
<pre><code class="language-csharp hljs hljsln"><span class="ln-bg"></span><span class="ln-num" data-num="1"></span>		<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">override</span> <span class="hljs-keyword">void</span> <span class="hljs-title">Dispose</span>(<span class="hljs-params"><span class="hljs-built_in">bool</span> disposing</span>)</span>
<span class="ln-num" data-num="2"></span>        {
<span class="ln-num" data-num="3"></span>            <span class="hljs-keyword">if</span> (_handle != <span class="hljs-literal">null</span> &amp;&amp; !_handle.IsInvalid) {
<span class="ln-num" data-num="4"></span>                <span class="hljs-keyword">try</span> {
<span class="ln-num" data-num="5"></span>                <span class="hljs-comment">//省略一部分代码</span>
<span class="ln-num" data-num="6"></span>                }
<span class="ln-num" data-num="7"></span>                <span class="hljs-keyword">finally</span> {
<span class="ln-num" data-num="8"></span>                    <span class="hljs-comment">// If we are disposing synchronize closing with raising SerialPort events</span>
<span class="ln-num" data-num="9"></span>                    <span class="hljs-keyword">if</span> (disposing) {
<span class="ln-num" data-num="10"></span>                        <span class="hljs-keyword">lock</span> (<span class="hljs-keyword">this</span>) {
<span class="ln-num" data-num="11"></span>                            _handle.Close();
<span class="ln-num" data-num="12"></span>                            _handle = <span class="hljs-literal">null</span>;
<span class="ln-num" data-num="13"></span>                        }
<span class="ln-num" data-num="14"></span>                    }
<span class="ln-num" data-num="15"></span>                    <span class="hljs-keyword">else</span> {
<span class="ln-num" data-num="16"></span>                        _handle.Close();
<span class="ln-num" data-num="17"></span>                        _handle = <span class="hljs-literal">null</span>;
<span class="ln-num" data-num="18"></span>                    }
<span class="ln-num" data-num="19"></span>                    <span class="hljs-keyword">base</span>.Dispose(disposing);
<span class="ln-num" data-num="20"></span>                }
<span class="ln-num" data-num="21"></span>            }
<span class="ln-num" data-num="22"></span>        }
<span class="ln-eof"></span></code></pre>
<p>SerialStream父类的Close方法调用了Dispose(true)，上面的代码一定会执行到lock (this) 语句，也就是说<strong>SerialStream实例执行Close()方法时会lock自身</strong>。</p>
<h1 id="死锁原因">死锁原因<a href="#死锁原因" class="esa-anchor" style="opacity: 0;">#</a></h1>
<p>把我们前面源码分析的结果总结一下：</p>
<ul>
<li><strong>DataReceived事件处理程序是在lock (stream) {...}块中执行的</strong></li>
<li><strong>SerialPort实例执行Close()方法时会先关闭SerialPort实例内部的SerialStream实例</strong></li>
<li><strong>SerialStream实例执行Close()方法时会lock实例自身</strong></li>
</ul>
<p>当辅助线程调用DataReceived事件处理程序处理串口数据但还未更新界面时，点击界面“关闭”按钮调用SerialPort实例的Close()方法，UI线程会在lock(stream)处一直等待辅助线程释放stream的线程锁。<br>
当辅助线程处理完数据准备更新界面时问题来了，DataReceived事件处理程序中的this.Invoke()一直会等待UI线程来执行委托，但此时UI线程还停在SerialPort实例的Close()方法处等待DataReceived事件处理程序执行完成。<br>
此时，线程死锁发生，两边都执行不下去了。</p>
<h1 id="解决死锁">解决死锁<a href="#解决死锁" class="esa-anchor" style="opacity: 0;">#</a></h1>
<p>网上大多数方法都是定义2个bool类型的标记Listening和Closing，关闭串口和接受数据前先判断一下。<br>
我的方法是<strong>DataReceived事件处理程序用this.BeginInvoke()更新界面，不等待UI线程执行完委托就返回</strong>，stream的线程锁会很快释放，SerialPort实例的Close()方法也无需等待。</p>
<h1 id="总结">总结<a href="#总结" class="esa-anchor" style="opacity: 0;">#</a></h1>
<p>问题最终的答案其实很简单，但我在查阅.NET源码查找问题源头的过程中收获了很多。这是我第一次这么深入的查看.NET源码，发现这种解决问题的方法还是很有用处的。<strong>结果不重要，解决问题的方法是最重要的。</strong></p>
</div>







# 网络通信编程的计算机组成原理基础

## 计算机如何存储浮点数

先转换成基为2的科学计数法形式 <font>±1.M × 2^E^</font>,`M`称作尾数，`E`称作阶码。`1.M`必须小数点左边有且仅有一个1(当然，表示0、+∞、-∞、NaN时除外)。

Sample: (408.6875)~10~==转二进制===> (110011000.1011)~2~ ==尾数变成1.M形式==> 2^8^ × (1.100110001011)~2~

最终浮点数在计算机中的存储形式是：`1bit表示正负，若干bit表示尾数M，若干bit表示阶码E`。 

| 符号位 | 尾数 | 阶码         | 表示              |
| ------ | ---- | ------------ | ----------------- |
| 0或1   | 0    | 0            | 0                 |
| 1      | 0    | 所有bit都是1 | -∞                |
| 0      | 0    | 所有bit都是1 | +∞                |
| 0或1   | !=0  | 所有bit都是1 | NaN(Not a Number) |

NaN可以表示 ① 0/0 ② 负数的平方根 ③ 无穷/无穷 ④ 0 × 无穷等运算结果没有意义的场景。部分NaN运算可能导致异常。

因为浮点数可以表示无穷，整数不能，所以整数运算不能除0，浮点数可以，程序不会抛出异常！而是得到无穷大或无穷小。

## 浮点数的特点

- 阶码的位数决定了表示范围，尾数的位数决定了表示精度。存储阶码和尾数的bit长度是有限的。
- 阶码上溢：当浮点数的阶码大于最大阶码时，称为上溢，此时机器停止运算，浮点运算器件会显示溢出标志。
- 阶码下溢：当浮点数的阶码小于最小阶码时，称为下溢，虽然此时数据不能被精确表示，但由于发生下溢时数据的绝对值很小，通常将尾数各位强置为0，按机器0处理，此时机器可以继续运行。
- 尾数溢出：精度损失，但不会导致程序异常，只是参与运算的操作数是原值的一个近似值。
- 尾数溢出，原数和实数的误差数量级可控，阶码溢出，则会差之千里。

## 计算机竟然无法精确存储(0.1)~10~

`十进制的有理数转换成二进制，得到的结果可能是无限循环或无理数！换句话说，任意M进制数并不一定有完全等价的N进制数与之对应(M!=N)。`

(0.1)~10~ = (0.0001100110011001100110011001100110011001100110011001101...)~2~

float和double的尾数bit长度分别是23和52，若数的尾数超出此长度，后面的01串会被截掉，发生精度损失。所以，0.1存储的01串和真实01串不相同，自然会出现`float f = 0.1; but 0.1 != f。`

0.1 + 0.1 + 0.1 - 0.3 != 0

0.7 - 0.2 != 0.5

3.3 ÷ 1.1 ！= 3

> 程序员编程时一定要非常小心，不能对两个浮点数直接进行判等比较操作(a == b),而是 (Math.Abs(a - b) < errorBand)。

## float和double的精度

因为阶码像指数那样，微小的增幅对一个数确实翻倍的影响，所以float和double的表示范围都很大，在实际开发中，基本不会出现溢出异常。`但是float和double并不能表示最大值和最小值之间的所有数字。`如果某个数的二进制01串比计算机存储尾数的bit数目还长，则会被截断，只有前面的01才会被存储，截掉的部分会被全部认作是0。举例：假设存储尾数的bit是14，现在有数(1.010_1001_1011_1111 × 2^15^)~2~，则实际存储到计算机的是(1.010_1001_1011_11 × 2^15^)~2~，该数是1010_1001_1011_1100。



float的精度是6，double的精度是15。换句话说，float能精确存储6个有效数字，double能精确存储15个有效数字。



有效数字的定义：从第一个非零的数字算起的所有数字，因此，1.24和0.00124的有效数字都有3位。

对数字强制保留n个有效数字时，一般会遵循四舍五入的进位规则。例如取1.23456789为三位有效数字后的数值将会是1.23，而取四位有效数字后的数值将会是1.235。



下面以float为例。

`float f = 123456789f;`     f是123456790。 近似存储

`float f = 0.0000123456f`    f是0.0000123456。  精确存储

`float f = 789.123456f;`    f是789.1235。  近似存储

`float f = 123.45678999f;`  f是123.45679000。 近似存储

## 计算机如何存储整数

n个二进制权位表示整数，[0,2^n-1^-1]表示正数，[2^n-1^,2^n^-1]表示负数，表示范围是[-2^n-1^,2^n-1^-1]。若`X`在[0,2^n-1^-1]，则`X`表示其本身，即`X`。若`X`在[2^n-1^,2^n^-1]，则`X`表示-(2^n^-X)。2^n^称作该权位个数下的mod.

> 计算机为什么不选择使用真值二进制方式存储负数，而是选择这样不直观故意复杂化的方式存储负数呢？因为，虽然对人类不友好，但是对机器友好。减法可以表示成加上一个负数，计算机运算时，不用考虑加法还是减法，只要将操作数的二进制存储形式对应的bit位置的数无脑相加，得到的结果就是正确的结果。

可以把表示范围想象成一个圆圈，有助于理解。

<img src="https://gitee.com/li6v/img/raw/master/202406302000835.png" alt="image-20240630200039670" style="zoom:67%;" />

- n是4，mod是2^4^=16,[0000,0111]\(0-7)表示0和正数，[1000,1111]\(8-15)表示负数，范围是[-(16-8),-(16-15)],[-8,-1].
- 有个规律：7是最大值，是分界线。7继续往上加，会跳变到-8，-7，-6.......，-8是最小值，继续往下减，会跳变到最大值7，6，5......

## 如何确定一个十进制数`X`的二进制存储形式

1. 先确定要存储X的变量的数据类型
   - 如果`X`不在该数据类型的表示范围内，则无法存储`X`，强制赋值会出现类型转换，导致失真。
2. 如果`X`是正数，写出二进制真值，权位不够补0。如果`X`是负数，先计算出正数`mod-abs(X)`，然后此正数的存储形式就是`X`的存储形式。

> 如果不声明存储十进制整数的数据类型，则无法确定二进制存储形式的。比如-128在sbyte和short两种情况下的二进制存储形式完全不同。

## 如何确定二进制存储形式表示的十进制数

1. 如果最高权位是0，直接二进制转十进制，转换后的结果就是最终结果。当然一定是0或正数。
2. 如果最高权位是1，先直接二进制转十进制，得到正数Y。然后确定二进制有多少个权位(假设是n)得到mod(2^n^)。最终得到十进制结果数`X=-(mod-Y)`.当然一定是负数.

## 溢出判断

无法把类型范围外的整数赋给该类型变量。

```c#
short a = 32767; // ok
short b = -32768; // ok
short c = 32768; // error
short d = -32769; // error
```

**操作数和运算结果**

正负之和不会溢出。因为和一定在[-(N+1),2N+1]内。

```c#
sbyte a = 127;
sbyte b = -1;
```

两正之和不在[0,N]，则溢出。

```c#
sbyte a = 100;
sbyte b = 50;
int result = a + b; // 结果不正确，因为a+b的结果类型是sbyte，已经溢出，转换来的int也必然不正确了。
```



两负之和[-(N+1),-1],则溢出。

## C#数字类型间相互转换规则

- 计算机中的数据以二进制的形式存储在寄存器或存储器中。

- 汇编语言中的数据类型取决于指令操作码。

  - 存储在寄存器、存储器中的操作数本身没有数据类型，对该数进行何种数据类型的操作完全取决于指令。

  - 同一个操作数，既可以当作有符号数，也可以当作无符号数；既可以是定点数，也可以是浮点数。

## 整型之间的类型转换

#### 1.相同字长之间

【二进制存储形式始终不变。】

```c#
sbyte a = 126;
byte b = (byte)126;
b = ?
```

126的存储形式是 0111_1110,解释成byte仍旧是126,所以b=126.

```c#
sbyte c = -127;
byte d = (byte)c;
d = ?
```

-127的存储形式是 1000_0001,解释成byte是2^7^+2^0^=129，所以d=129.

> 相同字长的类型之间的相互转换，可能失真。

#### 2.窄字长 to 宽字长

【原数据为无符号类型，进行0扩展；原数据为有符号类型，进行符号扩展。】



> 原数据是无符号类型，不会失真。
>
> 原数据是有符号类型，目标类型也是有符号类型，不会失真；目标类型是无符号类型，则可能失真。

#### 3.宽字长 to 窄字长

直接截除二进制存储形式的高权位bit，保留下来的低权位bit不变。

```c#
int i = -40832;
short s = (short)i;
```

求 s = ?

(-40832)~10~ = (1111_1111_1111_1111_0110_0000_1000_0000)~2~

截除左边16个高权位bit，剩下(0110_0000_1000_0000)~2~ = (24704)~10~

> 宽字长转成窄字长，可能会导致失真。

## 整型和浮点数之间的类型转换

以 int float double为例。

| float-->double                                               | double->float                                  | float/double-->int                                           | int-->float                                                  | int-->double                                                 |
| ------------------------------------------------------------ | ---------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 值完全相等                                                   | 大数可能溢出，高精度发生舍入                   | 小数部分舍弃，大数可能溢出                                   | 比较大的数无法精确表示                                       | 值相同                                                       |
| float f = 0.1f;<br> double d = f1;<br> d == f  √             |                                                | double d = 111111111111111; <br/>int i = (int)d;<br>i == -2147483648 √ | int i = 2147478889;<br>float f = (float)i;<br>f == 2147478900 √ |                                                              |
| double的阶码bits尾数bits更宽，用0填充新增bits,所以`d==f (√)`. | double的阶码bits尾数bits更宽，超出部分会被截断 | float/double的表示范围远远大于int，会导致溢出                | float的精度是6，但是int最大最小值有15个有效数字，所以精度会损失，但是不会溢出。 | int范围：-2147483648～2147483647。<br>最大最小值只有10个有效数字，double可表示15个有效数字，且表示范围数量级约10^308^，所以，double可以精确表示任何int整数。 |

假设变量i、f、d的类型分别是int、float和double，请判断下列C#语言关系表达式是否恒为真？

| 表达式                 | 真值     |
| ---------------------- | -------- |
| i == (int)(float)i;    | 不恒为真 |
| f == (float)(int)f;    | 不恒为真 |
| i == (int)(double)i;   | 恒为真   |
| f == (float)(double)f; | 恒为真   |
| d == (float)d;         | 不恒为真 |
| f == -(-f);            | 恒为真   |





























---









|      |      |      |
| ---- | ---- | ---- |
|      |      |      |
|      |      |      |
|      |      |      |

停止位非常重要，所以有些串口设备的串口驱动程序，每发完一帧，强行插入1个高电平，以保证每个帧解析前都能监控1个下降沿。

# uart device list

|     device      | Manufacturer         | baudrate | databits | parity | stopbits | flow control | check |  protocol  |
| :-------------: | -------------------- | :------: | :------: | ------ | :------: | :----------: | ----- | :--------: |
| butterfly valve | hightlight tech crop |  19.2k   |    8     | none   |    1     |     none     | crc16 | modbus-rtu |
|                 |                      |          |          |        |          |              |       |            |
|                 |                      |          |          |        |          |              |       |            |



# Modbus

[Modbus一站式学习点](https://www.modbus.cn/ "Modbus一站式学习点")
https://www.modbus.cn/



## 简介

1979年，莫迪康公司设计Modbus协议，无版权，免费使用。常作为电子设备，工控PLC的标准通信接口。

## 分类

| 基于串口    | 基于以太网                              |
| ----------- | --------------------------------------- |
| ModbusRtu   | ModbusTcp                               |
| ModbusAscii | ModbusUdp(实际工程中几乎不会使用此方式) |

ModbusRtu/Ascii被唤作Master/Slave模式。ModbusTcp被唤作Client/Server模式。

## 存储区和地址(√)

| 寄存器的种类 | 英文名称          | 区号 | 访问类型 | 数据类型               | 存储地址范围  | 报文协议地址范围 | 功能码     |
| ------------ | ----------------- | ---- | -------- | ---------------------- | ------------- | ---------------- | ---------- |
| 输出线圈     | coils             | 0    | 读写     | 位 1bit                | 000001-065536 | 0000-65535       | 01  05  0F |
| 离散输入     | Discrete Inputs   | 1    | 只读     | 位 1bit                | 100001-165536 | 0000-65535       | 02         |
| 保持寄存器   | Holding Registers | 4    | 读写     | 字 1word=2byte = 16bit | 400001-465536 | 0000-65535       | 03  06  10 |
| 输入寄存器   | Input Registers   | 3    | 只读     | 字 1word=2byte = 16bit | 300001-365536 | 0000-65535       | 04         |

Modbus存储区的地址范围从1开始编号，每个区长度65536, 第一个数字仅表示区号，编写文档与人沟通可以使用存储地址表示法。如300010,表示3区第10个地址；065536，表示01区第65536个地址。
存储地址范围65k有余，这个范围很大，实际项目中用不了这么多的地址。所以开发人员有时会省略一两个0，如0001-0999，1001-1999，3001-3999，4001-4999表达存储地址，应当注意理解这一点。如1005，表示1区第5个地址。
Modbus协议的报文地址都是`省略区号`的相对地址且从0开始计数，所以，协议报文地址比存储地址`小1`，范围是0x0000-0xFFFF。
所有存储区的地址个数相同，都是65536，输出线圈和离散输入的`一个地址表示1个bit`，对其一个地址读或写，需要一个bit信息；输入寄存器和保持寄存器的`一个地址表示1个word`，即表示2个字节，对其一个地址读或写，需要2个byte信息。

## Modbus协议特点

从站地址范围0-255。0广播地址，1-247单播地址，248-255保留地址，用于扩展，比如，假设用248表示[1-10]地址区间的广播地址，1-10地址的从机接收到地址是248的请求消息会执行Open动作，其他地址的从机收到消息直接忽略不执行任何动作。

地址只与从站有关，每个从站的地址必须是唯一的，主站是没有地址的。

只能主站主动先向从站发送请求消息，然后从站回复。从站不能主动向主站发消息，从站之间也不能相互发送消息。

主站发送的请求消息，所有从站都能收到，但是消息帧的地址与自身不匹配则丢弃消息不响应，匹配才响应。

单播模式下，从站必须回复，主站发送请求消息收到从站的响应消息后，才能继续向该从站发送下一条请求消息。

广播模式下，所有从站都能接收到主站的请求，但都不用回复消息。

## 常用功能码

| 功能码 | 描述                 |
| ------ | -------------------- |
| 0X01   | 读单个或多个线圈     |
| 0X02   | 读单个或多个离散输入 |
| 0X03   | 读保持寄存器         |
| 0X04   | 读输入寄存器         |
| 0X05   | 写单个线圈           |
| 0X06   | 写单个保持寄存器     |
| 0X10   | 写多个保持寄存器     |
| 0X17   | 读/写多个保持寄存器  |
| 0X0F   | 写多个线圈寄存器     |

`功能码占1个字节。正常的功能码的范围是1-127（0x01-0x7F），异常功能码是正常功能码+128(0x80)，范围是129-255(0x81-0xFF).所以，理论上，正常请求报文、正常响应报文、异常响应报文都不会出现0和128这两个功能码。`

## ModbusRTU

### 报文结构

| slave unit address | function code | data                   | CRC                 |
| ------------------ | ------------- | ---------------------- | ------------------- |
| 1个字节            | 1个字节       | 若干字节，取决于功能码 | 2个字节(低字节在前) |

### CRC校验

Modbus协议帧以字节流形式涌入串口，串口分割Modbus协议帧，每8个bit为一个串口帧，若干个串口帧发送完毕一个Modbus协议帧。接收方接收后，存在两处校验：第一处，接收方收到串口帧，进行奇偶校验，但不够严谨，这层校验是串口底层驱动做的。第二处，串口驱动程序将串口帧的起始位校验位停止位等去除，提取出8位有效数据放到接收缓存中，应用层从串口接收缓存读到的有效数据组成一个Modbus帧后，对Modbus帧进行CRC校验，这是个可靠的校验方法，这层校验是应用层开发者自己校验的。

`RTU对整个报文(unit address,function code, Data)的字节进行校验。CRC16-Modbus检验结果共2个字节，在组成Modbus帧时，低字节在前，高字节在后。如CRC16结果是0xCA31，实际消息帧发送的顺序是0x31,0xCA.`


**使用工具网站在线生成CRC16-Modbus:** [在线生成CRC16-Modbus](https://www.lddgo.net/encrypt/crc)

**使用串口调试软件生成CRC16-Modbus：**

<br>
<div align="center"><img src="https://img2022.cnblogs.com/blog/1037641/202209/1037641-20220929035514335-24704080.png" width="50%" height="50%" title="ComMonitor.exe"/></div>
<br>

### 0区功能码

#### 0x01 ReadMultiCoils 读取1个或多个线圈

线圈是0区，存储绝对地址范围是000001- 065536，即相对地址编号范围是1-65536。协议报文地址从0开始编号，始终比真实地址小1.

需求：读取线圈20-56值。

分析：

起始地址  20 = 0x14，减1是13H;

寄存器数量 56-20 + 1 = 37 = 25H;

结束地址 56 = 37H;

**请求报文**

| 地址 | 功能码 | 起始地址高位 | 起始地址低位 | 寄存器数量高位 | 寄存器数量低位 | CRC校验低位 | CRC校验高位 |
| ---- | ------ | ------------ | ------------ | -------------- | -------------- | ----------- | ----------- |
| 0x03 | 0x01   | 0x00         | 0x13         | 0x00           | 0x25           | 0x0D        | 0xF6        |

起始地址由两个字节构成，取值范围是0x0000 – 0xFFFF  (0-65535)，可表示65536个地址。

寄存器数量由两个字节构成，取值范围是0x0000-0x07D0 (1-2000)，最多一次性读取2000个。

**响应报文**

| 地址 | 功能码 | 数据域字节数 | 数据1 | 数据2 | 数据3 | 数据4 | 数据5 | CRC校验低位 | CRC校验高位 |
| ---- | ------ | ------------ | ----- | ----- | ----- | ----- | ----- | ----------- | ----------- |
| 0x03 | 0x01   | 0x05         | 0x53  | 0x6B  | 0x01  | 0xF4  | 0x1A  |             |             |

一个地址是1bit，37个地址是37bit，需要5个字节，多余的bit填0.

数据1表示00020 – 00027，01010011表示00027，00026，00025，00024，00023，00022，00021，00020的状态，低bit位低modbus地址。

数据2表示00028 – 00035

数据3表示00036 – 00043

数据4表示00044 – 00051

数据5表示00052 – 00056，00011010,前3bit无效，00052，00053，00054，00055，00056的状态依次是0，1，0，1，1.



```csharp
bool[] ReadCoils(byte slaveAddress, ushort startAddress, ushort numberOfPoints);
bool[] values = ReadCoils(3,19,37);
```
### 1区功能码

### 4区功能码(RW|保持寄存器|Holding Register|ushort)

#### 0X03 ReadMultiRegisters 读单个或多个寄存器

**请求报文**

| 域名                           | 值   |
| ------------------------------ | ---- |
| slave address                  | 0x01 |
| function code                  | 0x03 |
| initial address of read (high) | 0x00 |
| initial address of read (low)  | 0x01 |
| quantity of read (high)        | 0x00 |
| quantity of read (low)         | 0x03 |
| crc (low)                      | 0x54 |
| crc (high)                     | 0x0B |

**响应报文**

| 域名               | 值   |
| ------------------ | ---- |
| slave address      | 0x01 |
| function code      | 0x03 |
| byte count of data | 0x06 |
| data1 (high)       | 0x04 |
| data1 (low)        | 0x2B |
| data2 (high)       | 0x03 |
| data2 (low)        | 0x41 |
| data3 (high)       | 0x02 |
| data3 (low)        | 0x10 |
| crc (low)          | 0x54 |
| crc (high)         | 0x1F |



#### 0X10 WriteMultiRegisters 写多个寄存器

按下面表格所示，在地址是5的从站的寄存器中写入相应的值。

| 寄存器地址 | 设定值 |
| ---------- | ------ |
| 40020      | 0x0155 |
| 40021      | 0x0156 |
| 40022      | 0x0157 |

**请求报文**

*字节数量是要写入的地址数量的2倍，不包括2个校验字节。*

| 域名                            | 内容 |
| ------------------------------- | ---- |
| slave address                   | 0x05 |
| function code                   | 0x10 |
| initial address of write (High) | 0x00 |
| initial address of write (low)  | 0x13 |
| quantity of write (high)        | 0x00 |
| quantity of write (low)         | 0x03 |
| Byte Count of Write Data        | 0x06 |
| data1 (high)                    | 0x01 |
| data1 (low)                     | 0x55 |
| data2 (high)                    | 0x01 |
| data2 (low)                     | 0x56 |
| data3 (high)                    | 0x01 |
| data3 (low)                     | 0x57 |
| crc (low)                       | 0xB5 |
| crc (high)                      | 0xC1 |

**响应报文**

| 域名                            | 内容 |
| ------------------------------- | ---- |
| slave address                   | 0x05 |
| function code                   | 0x10 |
| initial address of write (High) | 0x00 |
| initial address of write (low)  | 0x13 |
| quantity of write (high)        | 0x00 |
| quantity of write (low)         | 0x03 |
| crc (low)                       | 0x70 |
| crc (high)                      | 0x49 |



#### 0X17 ReadWriteMultiRegisters 同时读写多个地址

**请求报文**
| 域名                                      | 所占字节数量 | 注释                                |
| ----------------------------------------- | ------------ | ----------------------------------- |
| Slave Address                             | 1            | 从机地址                            |
| Function Code                             | 1            | 功能码                              |
| Initial Address of Read (高字节在前)      | 2            | 读起始地址                          |
| Quantity to Read (高字节在前)             | 2            | 要读取的地址数量                    |
| Initial Address of Write (高字节在前)     | 2            | 写起始地址                          |
| Quantity to Write (高字节在前)            | 2            | 要写的地址数量                      |
| Byte Count of Write Data                  | 1            | 2 * N                               |
| Data to Write  (每个地址的数据高字节在前) | 2  *  N      | 要写的数据,this part May be deleted |
| CRC Check-Sum (低字节在前)                | 2            |                                     |

*N = Quantity to Write (请求报文要写的地址数量)*

**正常响应报文**

| 域名                                | 所占字节数量 | 注释                                      |
| ----------------------------------- | ------------ | ----------------------------------------- |
| Slave Address                       | 1            |                                           |
| Function Code                       | 1            |                                           |
| Byte Count                          | 1            | 2 * N，Read Data的字节数量                |
| Read Data(每个地址的数据高字节在前) | 2 * N        | 已经读到的数据， this part May be deleted |
| CRC Check-Sum                       | 2            |                                           |

*N = Quantity to Read (请求报文要读的地址数量)*


`此功能码也支持只写不读或只读不写。`
`只读不写时，请求报文的Initial Address of Write，Quantity to Write，Byte Count of Write Data都置0，移除Data to Write，请求报文不携带它。`
`只写不读时，请求报文的Initial Address of Read，Quantity to Read置0，正常响应报文的Byte Count会被置0，Read Data会被移除。`

### 3区功能码(ReadOnly|输入寄存器|Input Register|ushort)





#### 2 读取1个或多个离散寄存器 Read Discrete Inputs

和1相同，不再讲述。

#### 3 读取1个或多个保持寄存器 Read Holding Registers

**请求报文**

| 地址 | 功能码 | 起始地址高位 | 起始地址低位 | 寄存器数量高位 | 寄存器数量低位 | CRC校验低位 | CRC校验高位 |
| ---- | ------ | ------------ | ------------ | -------------- | -------------- | ----------- | ----------- |
| 0x07 | 0x03   | 0x00         | 0xC8         | 0x00           | 0x03           |             |             |

起始地址由两个字节构成，取值范围是0x0000 – 0xFFFF  (0-65535)

寄存器数量由两个字节构成，取值范围是0x0000-0x07D0 (1-2000)，最多一次性读取2000个。

起始地址  0x00C8 + 0x0001 = 0x00C9 = 00201

读取数量 0x0003 = 3

结束地址 00201 + 3 - 1 = 00203

读取范围 00201 - 00203

**响应报文**

| 地址 | 功能码 | 数据域字节数 | 数据1（高位） | 数据1(低位) | 数据2（高位） | 数位2（低位） | 数据3（高位） | 数据3（低位） | CRC校验低位 | CRC校验高位 |
| ---- | ------ | ------------ | ------------- | ----------- | ------------- | ------------- | ------------- | ------------- | ----------- | ----------- |
| 0x07 | 0x03   | 0x06         | 0x03          | 0x53        | 0x01          | 0xF3          | 0x01          | 0x05          |             |             |

请求报文请求3个地址，每个地址表示一个word = 2个字节，所以响应报文的数据域字节数应该是6.

如果读取保持寄存器地址为40001开始的一个16位的无符号数，那么返回2个字节，并可以从40002开始读取下一个16位的无符号数。如果需读取寄存器地址40001开始的是一个32位浮点数，则需要返回4个字节，即必须连续读取40001和40002的内容，而且下一个32位的浮点数必须从40003开始读取，对于浮点数（或者32位的整数）而言，连续读取的两个寄存器之间存在字节序和大小端的问题，这一点在开发时必须引起注意。

```csharp
ushort[] ReadHoldingRegisters(byte slaveAddress, ushort startAddress, ushort numberOfPoints);

ushort[] values = ReadHoldingRegisters(7,201,3);
values从左至右依次是保持寄存器地址增长方向的字的值，每个ushort元素的MSB对应先发送过来的高位字节。
```

<font color=red>（1）为什么NModbus要把读到的每个字都返回成ushort呢？</font>

因为寄存器的每一个地址是一个字，一个字等于两个字节，刚好可以构成一个ushort。

<font color=red>（2）NModbus的ushort是怎么计算得来的？</font>

一个寄存器地址的存储空间是2个字节，填充完毕16个bit，分两次发送出来，一次发送8个bit，也就是一个字节。先收到的字节被当作高位，假设是0x12，后收到的字节被当作低位，假设是0x34，NModbus拼出0x1234，即1 0010 0011 0100，然后将此01串解释成ushort. 我们只需要将ushort通过右移操作符就可以得到高位字节和低位字节，就可以知道一个寄存器地址的两个字节，并可以知道哪个字节先被发送来的(ushort的高位先被发送来的)。

<font color=red>（2）寄存器地址的一个字是怎么填充的？</font>

寄存器地址的一个字是16个bit，可以使用01填充。

<img src="https://img2022.cnblogs.com/blog/1037641/202209/1037641-20220929035410320-1966778720.png" alt="image-20220929033047316" style="zoom:50%;" />

如上图，10001100组成的字节先被发送出去，NModbus接收到后把它当作ushort的高位，后续01000100被发送出来，NModbus接收到后把它当作ushort的低位，1000110001000100就是NModbus的ushort。

其实NModbus把接收到的1000110001000100视作无意义的bit流，怎么解释都可以，可以认为bit流表示一个int16类型，也可以表示成两个Latin字符。

寄存器地址可以存放正数和负数，比如存放-66.

<img src="https://img2022.cnblogs.com/blog/1037641/202209/1037641-20220929035243947-892800680.png" alt="image-20220929033819129" style="zoom:50%;" />


-66对应的二进制是 1111 1111 1011 1110，1111 1111 是寄存器地址字的高位，1011 1110是低位，NModbus收到后，把1111 1111作为ushort的高位，把1011 1110作为ushort的低位，返回的ushort是65470！我们明知这个地址表示一个int16，那我们必须将ushort转换成高位字节和低位字节，然后再重新将高位字节和低位字节解读成int16.

#### 4 读取1个或多个输入寄存器

和3相同，不再讲述。

#### 5 写输出线圈单个地址 Write Single Coil

从设备地址是3，设置线圈地址00150为ON状态。

如果设置成1，变更数据必须是0xFF00，如果设置成0，变更数据必须是0x0000。变更数据要么是0xFF00，要么是0x0000，其他值均是非法的。

虽然只是设置一个bit，但是仍旧使用2个字节表示设置成1还是0.

**请求报文**

| 地址 | 功能码 | 起始地址高位 | 起始地址低位 | 变更数据高位 | 变更数据低位 | CRC校验低位 | CRC校验高位 |
| ---- | ------ | ------------ | ------------ | ------------ | ------------ | ----------- | ----------- |
| 0x03 | 0x05   | 0x00         | 0x95         | 0xFF         | 0x00         |             |             |

**响应报文**

响应报文和请求报文一摸一样。

#### 6 写保持寄存器单个地址 Write Single Register

从设备地址为3，设置寄存器地址40150为1200.

```csharp
void WriteSingleRegister(byte slaveAddress, ushort registerAddress, ushort value);
WriteSingleRegister(3,149,1200);
```

**请求报文**

| 地址 | 功能码 | 起始地址高位 | 起始地址低位 | 变更数据高位 | 变更数据低位 | CRC校验低位 | CRC校验高位 |
| ---- | ------ | ------------ | ------------ | ------------ | ------------ | ----------- | ----------- |
| 0x03 | 0x06   | 0x00         | 0x95         | 0x04         | 0xB0         |             |             |

设置寄存器地址

**响应报文**

响应报文和请求报文一摸一样。

```csharp
```



#### 10 写多个保持寄存器 Write Multiple Registers



```csharp
void WriteMultipleRegisters(byte slaveAddress, ushort startAddress, ushort[] data);

```



#### 15 写输出线圈的多个连续地址
功能：设置输出线圈的多个连续地址的值。
假设从站地址是5，需要设置线圈地址20-30的状态如下表。

| 值   | 1    | 0    | 0    | 0    | 1    | 0    | 1    | 1    | 1    | 0    | 1    | 0    | 0    | 0    | 0    | 0    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 线圈 | 20   | 21   | 22   | 23   | 24   | 25   | 26   | 27   | 28   | 29   | 30   | -    | -    | -    | -    | -    |

解析：

地址20-30，起始地址20-1=19=0x13。

数量30-20+1=11=0x0B。

数据域 = 地址值 + 地址值的字节数。

11个bit需要用两个字节表示，所以地址值的字节数=2.  

Modbus协议规定，按照线圈地址增长的方向，依次排列各个地址的值，然后每8个连续地址构成一个字节，字节内部的bit排序从左至右是大线圈地址的值->小线圈地址的值，不够8个的补0凑成一个字节，发送时，先发送小线圈地址对应的字节。

**发送报文**

| 从设备地址 | 功能码 | 起始地址(高位) | 起始地址(低位) | 寄存器数（高位） | 寄存器数（低位） | 字节数 | 变更数据（高位） | 变更数据（低位） | CRC校验低位 | CRC校验高位 |
| ---------- | ------ | -------------- | -------------- | ---------------- | ---------------- | ------ | ---------------- | ---------------- | ----------- | ----------- |
| 0x05       | 0x0F   | 0x00           | 0x13           | 0x00             | 0x0B             | 0x02   | 0xD1             | 0x05             |             |             |

**响应报文**
| 从设备地址 | 功能码 | 起始地址(高位) | 起始地址(低位) | 寄存器数（高位） | 寄存器数（低位） | CRC校验低位 | CRC校验高位 |
| ---------- | ------ | -------------- | -------------- | ---------------- | ---------------- | ----------- | ----------- |
| 0x05       | 0x0F   | 0x00           | 0x13           | 0x00             | 0x0B             |             |             |

```csharp
void WriteMultipleCoils(byte slaveAddress, ushort startAddress, bool[] data);
WriteMultipleCoils(0x05,0x13,new bool[]{true,false,false,false,true,false,true,true,true,false,true})
```



## 如何处理int32、int64、float、double

## 诊断功能

该功能码仅用于串行链路，主要用于检测主设备和从设备之间的通信故障，或检测从设备的各种内部故障，该功能码不支持广播。

## ModbusASCII

**报文构成**

| 帧头      | 地址    | 功能码  | 数据      | LRC     | 帧尾           |
| --------- | ------- | ------- | --------- | ------- | -------------- |
| 1个字符 : | 2个字符 | 2个字符 | 2*N个字符 | 2个字符 | 2个字符(CR LF) |

ModbusRtu的每个字节转换成其对应的十六进制的字符，便得到ModbusAscii报文，所以，Ascii报文长度约是Rtu的2倍。
LRC校验的数据范围不包括帧头:和帧尾CR LF，校验的时机是对报文中的每个原始字节进行ASCII编码之前，对每个原始字节进行校验，也就是对其对应的RTU报文数组的所有字节进行校验，可参考下面的源码。

`建议：构建ASCII报文，先构建RTU报文,再参考下面的代码转换得到最终的ASCII报文字符串。`

**ModbusRtu转换成ModbusAscii报文的C#源码：**

```C#
var bytes = new byte[6] { 0x03, 0x01, 0x00, 0x13, 0x00, 0x25 }; // RTU报文数组: 地址 + 功能码 + 数据
var sb = new StringBuilder(":"); // 添加ASCII报文起始字符
Array.ForEach(bytes, item => sb.Append(item.ToString("X2")));// 依次将RTU报文数组中的每个字节变成2个字符
sb.Append(LrcModbus(bytes).ToString("X2")); // 对RTU报文数组进行LRC校验得到一个字节，再将其转换成两个十六进制字符
sb.Append("\r\n"); // 添加ASCII报文结束字符
SerialPort.Send(sb.ToString()); // 发送ASCII请求报文
```

---

`如何解析ModbusASCII响应报文？`



## ModbusTCP

1996年施耐德公司推出基于以太网TCP/IP的Modbus协议：ModbusTCP。

Modbus设备可分为主站(poll)和从站(slave)。主站只有一个，从站有多个，主站向各从站发送请求帧，从站给予响应。在使用TCP通信时，主站为client端，主动建立连接；从站为server端，等待连接。

ModbusTCP默认端口502.

| Transaction Identifier                           | Protocol Identifier                             | Length                                          | Unit Identifier                           | 功能码 | 数据域 |
| ------------------------------------------------ | ----------------------------------------------- | ----------------------------------------------- | ----------------------------------------- | ------ | ------ |
| 2个字节，高位在前，低位在后，客户端每发次消息加1 | 2个字节，0x0000表示ModbusTCP,此字段固定为0x0000 | 2个字节，高位在前，低位在后，表示后续字节的数量 | 1个字节，表示从机地址（大部分情况下0xFF） |        |        |

**ModbusTCP和ModbusRTU区别**

* ModbusTCP和ModbusRTU相同点是都有功能码和数据域，Modbus开头无设备地址，结尾无CRC校验字段。
* 主站作为客户端，从站作为服务器，服务器IP地址唯一标识从站地址，所以，Unit Identifier无效，必须固定为0xFF
* 如果许多从站组成串行链路子网，然后挂到作为从站的服务器上，这时候Unit Identifier生效，标识服务器背后的串行链路子网的从站地址，客户端向服务器发送ModbusTCP报文，服务器收到后，提取Unit Identifier转发给背后的从站，此时IP地址类似于"路由器"，并不标识具体的从站设备。
* ModbusRTU基于串口，串口的CRC校验不是可靠检验，所以在ModbusRTU报文添加CRC校验字段保证可靠传输；ModbusTCP基于TCP/IP，是可靠传输，所以无需在报文中添加CRC校验了。
* ModbusRTU,接收方以<3.5字符时间间距区分一个完整报文，ModbusTCP靠Length字段区分一个完整报文。
* 因为Modbus是一发一收模式，发送完毕，响应报文肯定大于3.5个字符，所以，发送时不必考虑时间间隔的问题。



## ModbusPoll和ModbusSlave使用教程

以读取输入寄存器为例。

一个输入寄存器地址表示一个字，也就是2个字节。地址中可以存放有符号数(正数和负数)，也可以存放无符号数。核心点是无论你放什么数，都是16bit的0或1.



**NModbus的设计思路**

```csharp
bool[] ReadCoils(byte slaveAddress, ushort startAddress, ushort numberOfPoints);
```


```csharp
using (SerialPort port = new SerialPort("COM2"))
{
    port.BaudRate = 9600;
    port.DataBits = 8;
    port.Parity = Parity.Even;
    port.StopBits = StopBits.One;
    port.Open();
    var adapter = new SerialPortAdapter(port);
    var factory = new ModbusFactory();
    IModbusMaster master = factory.CreateRtuMaster(adapter);
    byte slaveId = 0x03; // 设备地址
    ushort startAddress = 0x13; // 起始地址，从0开始编号，0 - 65535 ，其真实的存储地址是 0x14
    ushort numberOfPoints = 0x25; // 读取的地址个数
    bool[] values = master.ReadCoils(slaveId, startAddress, numberOfPoints);
    Console.WriteLine(String.Join(",", values));
}
```

## Modbus异常响应报文

* 正常接收，正常处理，返回正常响应报文。
* 因为通信错误等原因，造成从站设备没有接收到查询报文，主站设备将按超时处理。
* 从站设备接收到的查询报文存在通信错误(例如CRC错误)，此时从站设备将丢弃报文不响应，主站设备将按超时处理。
* 从站设备接收到正确的报文，但是超过处理范围（例如，不存在的功能码或寄存器等），此时从站设备将返回包含异常码（Exception Code）的响应报文。

异常响应报文由从站地址，功能码以及异常码构成。其中，功能码与正常响应报文不同，在异常响应报文中，功能码最高位（即MSB）被设置成1,即异常功能码=正常功能码+0x80.
`功能码占1个字节。正常的功能码的范围是1-127（0x01-0x7F），异常功能码是正常功能码+128(0x80)，范围是129-255(0x81-0xFF).所以，理论上，正常请求报文、正常响应报文、异常响应报文都不会出现0和128这两个功能码。`

![image](https://img2024.cnblogs.com/blog/1037641/202404/1037641-20240424195104587-525592350.png)
![image](https://img2024.cnblogs.com/blog/1037641/202404/1037641-20240424195117150-536390430.png)

## modbus-rtu如何Resolve一个完整帧

### 发送具有连续不间断性

假设串口要发送N个字节，SerialPort.Send(byte[] bytes)，bytes(长度是N)会一次性全部发出，中途不会中断。串口发送线上的电平序列是:

**(起始位 + 数据位 + 校验位 + 停止位)~1~ + (起始位 + 数据位 + 校验位 + 停止位)~2~ + ... + (起始位 + 数据位 + 校验位 + 停止位)~N~ + 空闲位 + ... + 空闲位 + ...**

两个Frame之间不会出现空闲位，上一个Frame的停止位后立刻是下一个Frame的起始位；所有的Frame发送完毕后，发送线上一直维持着空闲位。

### 使用示波器验证电平序列

连发两个字节0x75(0111 0101) 0x6B (0110 1011)，串口设置：波特率9600，停止位1位，无奇偶校验位。

<img src="https://img2023.cnblogs.com/blog/1037641/202309/1037641-20230909215120983-626692775.png" alt="image" style="zoom: 33%;" />

`串口通信，字节内采用的是小端，先发送低权bit，所以bit序列是1010 1110 1101 0110.`

`两个Frame被连续发送，中间未插入任何空闲位。`

### 终论：3.5 characters

**modbus-rtu若超出3.5ms未收到任何字节，便把当前已经收到的所有数据作为一个完整的报文来处理。**

所以必须保证：

① 一个完整的报文的所有字节，必须通过SerialPort.Send(byte[] allbytes)一次性全部发出。如果将一条完整报文拆分成多批，多次调用SerialPort.Send()发出，那么每个字节片段之间会被插入许多空闲位；若接收者收到一个片段，超出3.5个字符还未收到下一个片段，便会错将片段当成完整报文处理。

② 3.5个字符的时间约是 3.5 * 10 / baudrate * 1000 msec。transmitter发送完一个完整报文后，必须等待至少前面计算的时间后才能发送下一个报文！以便receiver区分出完整报文。这段等待时间，发送线上一直维持着空闲位。

![image-20240613160327680](D:\OneDrive\markdownImages\image-20240613160327680.png)



# ModbusRTU如何区分一个完成帧

SerialPort.Send(byte[] bytes)

bytes会一次性全部发出，不会中断。

连发发送两个16进制数据0x75(0111 0101) 0x6B (0110 1011)
串口设置：波特率9600，停止位1位，无奇偶校验位。

![image](https://img2023.cnblogs.com/blog/1037641/202309/1037641-20230909215120983-626692775.png)

以下是一组主机询问与从机回答的波形，中间间隔7ms左右。注意，MODBUS规定两组数据之间必须有3.5字符的间隔，我的单个字符时长大约1ms，所以这个间隔不得小于3.5ms。 







ModbusRTU帧之间要间隔2-3个字符约20-30个bit的时长。

https://blog.csdn.net/weixin_43319854/article/details/109844860?utm_source=app&app_version=4.21.0&code=app_1562916241&uLinkId=usr1mkqgl919blen

停止位非常重要，所以有些串口设备的串口驱动程序，每发完一帧，强行插入1个高电平，以保证每个帧解析前都能监控1个下降沿。

Send("ABCDEFG")，共7个串口帧，1个帧发送完毕，下1帧立刻接着被发送，不过可能有1个电平时长的故意延时。
所以ModbusRTU帧不能太长，超出串口发送BUFF，被分多次发送了，如果多次发送时间间隔不小心大于2-3个字符时长，就GG了。



# Windows10上安装Mosquitto的步骤

# 前言
mosquitto是一款开源免费的软件，[官网链接](https://mosquitto.org/ "官网链接")。它是一些可执行文件的集合，通过这些可执行文件，它提供broker，publish，subscribe功能。我们安装mosquitto一般是为了让它作为MQTT的服务器，我们也可以使用它提供的客户端的订阅发布功能进行调试，但是有更好的MQTT客户端调试软件，如MQTT.fx。
安装mosquitto后，它会以服务的形式运行在主机上。只要我们开启服务，便可以让MQTT客户端连接。我们可以在官网查看文档学习使用mosquitto的更多功能。

# 1.下载安装
去[官网链接](https://mosquitto.org/ "官网链接")下载安装包。

![image](https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210824102825812-458736604.png)

安装过程中，建议把Mosquitto安装成服务。如果把Mosquitto安装成服务，那么Mosquitto Broker会开机自启，否则，每次开机我们需要手动运行Mosquitto.exe，才能让MQTT客户端连接订阅发布。

![image](https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210824103450918-2090795481.png)

安装完成后，我们去安装目录。默认安装目录是 `C:\Program Files\Mosquitto`.我们尽量不要在安装目录中有空格，所以我们尽量安装在`C:\Mosquitto`.
![image](https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210824104037261-1010507438.png)

| 文件                 | 功能                                                         |
| -------------------- | ------------------------------------------------------------ |
| mosquitto.exe        | 启动Broker功能。如果没有把mosquitto安装成服务，必须手动启动它，才能提供服务器功能。如果已经把mosquitto安装成服务并启动服务，则不需要也不能启动该可执行文件 |
| mosquitto_sub.exe    | 可以模拟客户端订阅话题                                       |
| mosquitto_pub.exe    | 可以模拟客户端发布话题                                       |
| mosquitto.conf       | 配置文件，可以设置一些MQTT相关参数                           |
| mosquitto_passwd.exe | 用于生成Broker账号密码                                       |
| pwfile_example       | 存放Broker的账户密码信息                                     |


# 2.设置Broker的IP和Port

1. 进入mosquitto安装目录，会看到文件`mosquitto.conf`，如果不存在该文件，可创建。打开mosquitto.conf，在文件尾部追加行。

```txt
listener 1883 172.15.7.43
```

说明：其中，`1883`是端口，`172.15.7.43`是IP，可根据个人需求设置具体的IP和端口。

Mosquitto若支持客户端无账号密码验证连接，需要配置`mosquitto.conf`.打开`mosquitto.conf`，在尾部添加`allow_anonymous true`.
至此Mosquitto配置完毕，重启Mosquitto broker服务使配置生效即可。
如果想让客户端必须经过账号密码验证，那就继续文章后面的配置。

# 3.设置账户和密码

mosquitto支持设置多个账户。

1. 打开mosquitto.conf，在文件尾部追加行

```txt
allow_anonymous false
password_file C:\Program Files\Mosquitto\pwfile.example
```

说明：mosquitto作为MQTT的broker，默认情况下是支持客户端无账号密码连接的。`allow_anonymous false`意思是禁止客户端采用无账号密码的匿名访问方式。`password_file C:\Program Files\Mosquitto\pwfile.example`意思是设置Broker存储账号密码的文件是`C:\Program Files\Mosquitto\pwfile.example`.

2. 以管理员身份打开Windows PowerShell
![image](https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210824110500802-830747633.png)
3. 切换到mosquitto安装目录

   ```powershell
   cd 'C:\Program Files\Mosquitto\'
   ```

   说明：powershell命令中，建议所有的路径使用单引号包裹，尤其路径中含有空格的情况，这样兼容性会更好。

4. 添加账户。假设我们要添加的账户是`admin`

   ```powershell
   .\mosquitto_passwd.exe  -c  .\pwfile.example  admin
   ```

   说明：`admin`是要添加的账户名，放到命令的末尾。`.\pwfile.example`意思是指把账户信息写入到文件`pwfile.example`,`-c`意思是如果`.\pwfile.example`不为空，即已存在账户信息，那么就把原来的账户信息抹除掉，即覆盖(override)`pwfile.example`。运行完此命令后，mosquitto现在只存在一个账户`admin`。

5. 如果我们还想再添加其他账号，则继续运行下面命令；否则结束。

   假设我们继续添加第二个账户`test`

   ```powershell
   .\mosquitto_passwd.exe .\pwfile.example test
   ```

   **注意** : 本次命令中不要加`-c`，否则会覆盖掉`pwfile.example`，抹除之前的所有账户信息，导致只存在账户`test`.

# 4.重启服务mosquitto broker

![image](https://img2020.cnblogs.com/blog/1037641/202108/1037641-20210824110547540-1708168107.png)

说明：和MySQL一样，修改完配置文件，需要重启服务生效。

# 5.如何让Windows服务开机自启
![image](https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210923232433469-1797880936.png)

------------

![image](https://img2020.cnblogs.com/blog/1037641/202109/1037641-20210923232528853-1883963949.png)



# 网络编程常用的C# API

```c#
// 字节数组转换成16进制
BitConverter.ToString(byte[] bytes);  // 55-00-54-00-46-00-2D-00-38-00-60-4F

// 10进制表示整数，其他进制表示字符串。所以10进制和其他进制的相互转换就是整数和字符串之间的相互转换。

// 10进制转2进制和16进制
Convert.ToString(100, 2);
Convert.ToString(100, 16);

// 2进制转和16进制转10进制
Convert.ToInt32("1010", 2);
Convert.ToInt32("1FF", 16);

// 0进制转转16进制自动补0
Convert.ToString(100, 16).PadLeft(4, '0'); // 不满4位补0，超出4位什么也不做，得到的16进制字符串可能是5、6、7...位
```





# 字节序、位序、CRC

## C#验证主机字节序代码

<table><td bgcolor=gray>方式一</td></table>

```csharp
public static bool IsLittleEndian()
{
    int a = 0x01020304;
    var bytes = BitConverter.GetBytes(a);
    if (bytes[0] == 0x01)
    {
        return false;
    }
    else if (bytes[0] == 0x04)
    {
        return true;
    }
    else
    {
        Debug.Assert(false);
        throw new Exception();
    }
}
```

说明：在C、C++、C#、Java、Python等常见的编程语言中，字节数组从左往右(索引增大的方向)内存地址是递增的(低地址->高地址)。bytes[0]是最低地址，如果等于最高权位0x01，则低地址放高权位，是大端模式。

<table><td bgcolor=gray>方式二</td></table>

BitConverter的静态属性IsLittleEndian，True 小端模式，False 大端模式。这个API是BCL内置的。

`BitConverter.IsLittleEndian`



![image](https://img2023.cnblogs.com/blog/1037641/202309/1037641-20230918193956839-1177273844.png)



CRC校验，对任意长01序列进行位运算得到1个任意长(一般会约束为8或16,或32bit长)01序列，短01序列能够唯一标识前者。
发送方的01序列规则，字节数组，索引小字节在前，字节内高位bit在前，如 byte[] sendbytes = new byte[3] {1,2,3}, 则拼接出的01序列是0000 0001 0000 0010 0000 0011.
网络发送相当于数组的完全平移，所以接收方也可以按照发送方的01序列进行位运算，得到冗余。
这就是CRC位运算的计算机原理。



比特(bit)的发送和接收顺序
 比特的发送、接收顺序是指一个字节中的bit在网络电缆中是如何发送、接收的。在以太网(Ethernet)中，是从最低有效比特位到最高有效比特位的发送顺序，也就是最低有效比特位首先发送，参考资料：frame。
 在以太网中这个规定有点奇怪，因为字节序我们是按照大端序来发送，但是比特序却是按照小端序的方式来发送，
————————————————
版权声明：本文为CSDN博主「NoneSec」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/liuxingen/article/details/45420455

比特的发送、接收顺序对CPU、软件都是不可见的，因为我们的网卡会给我们处理这种转换，在发送的时候按照小端序发送比特位，在接收的时候会把接收到的比特序转换成主机的比特序，下面是一个小端机器发送一个int整型给一个大端机器的示意图： 

可以假设存在ntohb、htonb(b代表bit)这样的两个函数，网卡进行了比特序的转换，不过是这两个函数是网卡自动调用的，我们平时不用关注。





# MQTT重点概念

MQTT客户端连接MQTT服务器的过程：
1. TCP网络连接成功
2. 发送CONNECT协议包
3. 服务器回复CONNECT确认包

After a Network Connection is established by a Client to a Server, the first Packet sent from the Client to the Server MUST be a CONNECT Packet.

A Client can only send the CONNECT Packet once over a Network Connection. The Server MUST process a second CONNECT Packet sent from a Client as a protocol violation and disconnect the Client.



CONNECT
连接参数：

4. 客户端标识 ClientID
（1）一般使用GUID
（2）恢复会话依赖客户端标识
（3）CLean Session 为false时，则不能使用GUID，必须是固定值

CleanSession
 （1）一般设置成true，清除上次会话。重新连接时，需要重新订阅。


1. 协议版本
（1）3.1.0
（2）3.1.1
（3）5.0

如果协议等级不被服务端支持，服务端必须响应一个包含代码0x01（不接受的协议等级）CONNACK包，然后断开和客户端的连接。

2. 账户名密码
（1）客户端连接服务器时，需要权限认证。
（2）服务器可以设置无需权限认证，此时客户端的CONNECT包不包含账号字段和密码字段。

3. 心跳包 keep alive
（1）Keep Alive是以秒为单位的时间间隔。
（2）客户端有责任确保两个控制包发送的间隔不能超过Keep Alive的值。如果没有其他控制包可发，客户端必须发送PINGREQ包。
（3）如果Keep Alive的值非0，而且服务端在一个半Keep Alive的周期内没有收到客户端的控制包，服务端必须作为网络故障断开网络连接
（4）如果客户端在发送了PINGREQ后，在一个合理的时间都没有收到PINGRESP包，客户端应该关闭和服务端的网络连接。
（5）Keep Alive的值为0，就关闭了维持的机制。这意味着，在这种情况下，服务端不会断开静默的客户端。
（6）Keep Alive的实际值是由应用程序指定的；典型的是几分钟。最大值是18小时12分钟15秒。


5. Will Topic
6. Will Message
Will QoS
发布遗言时的等级。

Will Retain
遗言被发送后服务器是否继续保留。

遗言在连接时被存储到服务器，所以客户端异常断开后，该客户端的遗言仍旧可以被下发到其他客户端。
如果客户端主动调用Disconnect函数并发送了Disconnect协议包，那么遗言不会被发送，且被服务器清除。


遗言等级

清除会话


CONNACK：
 The first packet sent from the Server to the Client MUST be a CONNACK Packet.

连接返回码Connect Return code values
如果服务端收到了一个格式良好的CONNECT包，但是服务端由于某种原因不能处理，那么服务端应该尝试发送一个带有非零返回码的CONNACK包。如果服务端发送了一个包含非零返回码的CONNACK包，必须关闭网络连接



Table 3.1 - COnnect Return code values

|Value  |Return Code Response                                   |Description
|0      |0x00 Connection Accepted                               |Connection accepted
|1      |0x01 Connection Refused, unacceptable protocol version |The Server does not support the level of the MQTT protocol requested by the Client
|2      |0x02 Connection Refused, identifier rejected           |The Client identifier is correct UTF-8 but not allowed by the Server
|3      |0x03 Connection Refused, Server unavailable            |The Network Connection has been made but the MQTT service is unavailable
|4      |0x04 Connection Refused, bad user name or password     |The data in the user name or password is malformed
|5      |0x05 Connection Refused, not authorized                |The Client is not authorized to connect
|6-255  |                                                       |Reserved for future use

如果上表中的返回码都不适用，那么服务端必须直接关闭网络连接，不发送CONNACK包。



0x01:非法的MQTT协议版本号
0x02：Clinet ID 非法

 



# [RS485中继器和集线器](https://www.cnblogs.com/euvio/p/17858883.html)

RS485中继器和集线器

https://www.szutek.com/product-96.html



# 字节序



# 重要规则

（1）多字节类型才存在字节序，如UInt16, Int32, float；

（2）内存的读写永远是先使用低地址，再使用高地址。

（3）高级语言的字节数组，0索引是最低地址，索引越大地址越大，从左至右是地址增长的方向。

（4）发送数据时，先发送低地址，后发送高地址。接收数据时，先接收到发送方先发送的数据，先接收到的数据存放到低地址，后接收到的数据存放到高地址。

（5）发送字节数组，先发送bytes[0],再发送bytes[1]，然后发送bytes[2] ......,接收方接收时，开辟一个字节数组，先收到的字节存放到recvedbytes[0]，后续收到的字节放到recvedbytes[1]......


# MSB和LSB

MSB和LSB是指一个数字的高位和低位，高和低是指权值。它纯粹是数学概念，只是存放到内存时，牵扯出大端和小端的问题。

MSB（Most Significant Bit/Byte）高位

LSB（Little Significant Bit/Byte）低位

比如一个十六进制的整数 0x12345678

| 0x12 | 0x34 | 0x56 | 0x78 |
| :--: | :--: | :--: | :--: |

0x12 就是 MSB，0x78 就是 LSB。而对于 0x78 这个字节而言，它的二进制是 01111000，那么最左边那个 0 就是 MSB，最右边那个 0 就是 LSB。

# 大端,小端和CPU,内存的关系
大端小端问题存在的根本原因是CPU体系不同。CPU都是先读低地址，再读高地址，有的CPU把低地址的字节作为MSB，有的CPU把高地址的字节作为MSB。
如果某个CPU把从低地址读到的字节作为多字节类型的MSB，那么我们必须把MSB字节放到内存的低地址，因为我们无法改变CPU的工作方式，只能迁就CPU，根据CPU的情况把MSB和LSB字节摆放在内存中的相应位置。
可以把内存看作有地址的小单元格，有的单元格地址小，有的单元格地址大，需要读取某个单元格时指定地址就可以了。

# 大端和小端

**大端（Big-endian）**：规定 MSB 在存储时放在低地址，在传输时放在流的开始；LSB 在存储时放在高地址，在传输时放在流的末尾。（即高位字节在前，低位字节在后。）

**小端（Little-endian）**：规定 MSB 在存储时放在高地址，在传输时放在流的末尾；LSB 在存储时放在低地址，在传输时放在流的开始。（即低位字节在前，高位字节在后。）

例如： 0x12345678 在不同机器中的存储是不同的，如下所示。

|        | 大端（Big-endian） | 小端（Little-endian） |
| :----: | :----------------: | :-------------------: |
| 0 字节 |        0x12        |         0x78          |
| 1 字节 |        0x34        |         0x56          |
| 2 字节 |        0x56        |         0x34          |
| 3 字节 |        0x78        |         0x12          |

**字节序（Byte Order）是指多字节类型的各个字节在计算机内存中的存储顺序或者网络传输时各字节的发送顺序。其有两种存储方式：大端（Big-endian）和小端（Little-endian），也有两种发送顺序：先发送高字节MSB（大端）和先发送低字节LSB（小端），TCP/IP协议规定，发送方要保证发送数据时，一定要先发送高字节后发送低字节，所以有个共识，网络字节序固定是大端模式！所以接收方可认为自己先收到的字节是高字节。**


# C#判断主机大端还是小端

```csharp
static void Main(string[] args)
{
    if (BitConverter.IsLittleEndian)
    {
        Console.WriteLine("小端");
    }
    else
    {
        Console.WriteLine("大端");
    }
}
```

或

```csharp
static void Main(string[] args)
{
    int a = 0x01020304;

    byte[] bytes = BitConverter.GetBytes(a);
    // 数组索引越大，地址越高。如果是小端，bytes[0]应是0x04，大端bytes[0]应是0x01.
    if (bytes[0] == 0x01)
    {
        Console.WriteLine("大端");
    }
    else
    {
        Console.WriteLine("小端");
    }

    Console.ReadKey();
}

```

# 说透大小端

以下所提及的高位和低位是指MSB和LSB。

字节数组从左至右是地址增长的方向，即索引越大，地址越大。

读写内存都是先读低地址，再读高地址。

发送方发送数据，先发送低地址，即从左至右依次逐个发送字节数组中元素，send(bytes[0])，send(bytes[1])，send(bytes[2])......

接收方接收数据，先存到低地址。开辟一个字节数组，收到一个字节就放到字节数组中，从左至右放。

发送方发送的数组被完整挪到接收方，接收方的数组和发送方的数组完全一样。此数组中高位字节在左(低地址),低位字节在右(高地址)，就像大端一样。接收方根据自身是大端模式还是小端模式进行解析。


**演示发送方发送整数0x12345678的过程**

网络字节序：先发送的是高位，后发送的是低位；如0x12345678，0x12先被发出，0x34后被发出，接着是0x56, 0x78。那么接收方先收到的字节被认为是高位，后接收到的字节被认为是低位。所以，发送方必须保证自己先发送高位，后发送低位。 

假设要发送整数0X12345678，应该先发送0X12，再发送0X34，继续发送0X56，最后发送0X78。发送方创建一个数组bytes，从左到右依次放0X12，0X34，0X56，0X78，然后调用API Send(bytes)。因为读或写内存都是从低地址开始，而数组索引越大，地址越大，索引0是最低地址！从而，保证了发送顺序是0x12,0x340x,56,0x78，满足网络字节序要求。

假设发送方假设要发送十进制的-66，先利用BitConverter.GetBytes()找到高位和低位。清晰的区分出高位字节和低位字节后，便可以先发送高位字节，再发送低位字节。

**发送方所做事宜**

```csharp
/* 方式一 */
int a = -66;
// bytes的长度为4，是已知的。但是bytes从左至右是高位->低位还是低位->高位还不确定
// 如果主机是小端模式则低地址存放低位，因为数组0索引是最低地址，bytes从左至右是低位->高位
// 如果主机是大端模式则低地址存放高位，因为数组0索引是最低地址，bytes从左至右是高位->低位
// 我们期望的是bytes从左至右是高位->低位，因为发送数据时，先发送低地址，也就是从左至右逐个发送字节数组中的字节，从而满足网络字节序的要求：先发送高位，后发送低位。

byte[] bytes = BitConverter.GetBytes(a);
// 如果主机是小端，反转数组，保证bytes从左至右是高位->低位，也就是大端模式
if (BitConverter.IsLittleEndian)
{
     Array.Reverse(bytes);
}
// 从左至右逐个发送字节数组中的字节(先发送低地址)，这样就一定先发送的高位，后发送的低位，满足网络字节序
Send(bytes);


/* 方式二 */
int a = -66;
// 在BitConverter.GetBytes(a)先转换成网络字节序，这样能保证字节数组bytes从左至右是高位->低位
a = IPAddress.HostToNetworkOrder(a);
bytes = BitConverter.GetBytes(a);
Send(bytes);

/* 方式三 */
int a = -66;

byte byte1 = (byte)(a >>> 24);
byte byte2 = (byte)(a >>> 16);
byte byte3 = (byte)(a >>> 8);
byte byte4 = (byte)(a >>> 0);

Send(new byte[] { byte1, byte2, byte3, byte4 });
```



接收方按照发送顺序接收数据，先接收到的字节存放到低地址，再接收的字节存放到下一个更大的地址，也就是接收方创建一个字节数组，将接收的字节从左至右依次放到数组中，那么数组从左至右是高位->低位，即低地址放高位，大端模式。接收方操作字节数组转换成类型。

```csharp
byte[] bytes = new byte[4];
bytes[0] = byte1;  
bytes[1] = byte2;
bytes[2] = byte3;
bytes[3] = byte4;


/* 方式一 */

// 如果主机是小端，反转数组
if (BitConverter.IsLittleEndian)
{
    Array.Reverse(bytes);
}

int result = BitConverter.ToInt32(bytes,0);

/* 方式二 */
int result = IPAddress.NetworkToHostOrder(BitConverter.ToInt32(bytes,0));
```

**综上所述，可以得出下图。**

![image](https://img2022.cnblogs.com/blog/1037641/202209/1037641-20220928015541014-1420694897.png)


**从宏观来看，发送方先将提取发送的多字节类型的高位和低位，按照大端模式存放到字节数组中，接收方接收完毕后，相当于发送方的字节数组一成不变的“挪到”接收方，接收方的字节数组是大端模式，接收方如何将字节数组转换成类型，还要考虑自身的主机字节序。**

# 利用Modbus及MobusSlave实战字节序

下面是设置一个保持寄存器的值的Modbus协议帧。

| 从设备地址 | 功能码 | 起始地址(高位) | 起始地址(低位) | 变更地址(高位) | 变更地址(低位) | CRC低位 | CRC高位 |
| ---------- | ------ | -------------- | -------------- | -------------- | -------------- | ------- | ------- |
| 0x03       | 0x05   | 0x00           | 0x95           | 0xFF           | 0x00           |         |         |

这个帧从左至右表示的是指`发送顺序`，按照此顺序发送，Modbus接收设备才可以正常解析。

创建一个byte数组bytes，从左到右按照表格中的顺序存放字节，网络发送的规则是：先发送低地址的字节！高级语言的数组的地址规则是：索引越小，地址越小。所以，先发送bytes[0]，再发送bytes[1]......，刚好满足Modbus协议帧规定的发送顺序。

想发送十进制的255时，转换成十六进制就是0xFF00，我们按照Mobuds协议帧规定，0xFF放到字节数组的前面，0x00放到字节数组的后面即可，所以，发送数据时，无需考虑大端小端的问题，至于Modbus接收到数据如何存放，那是Modbus设备驱动自己搞定的，我们只要按照要求的顺序发送即可。

```csharp
List<byte> bytes = new List<byte>();
bytes.Add(0x03);
bytes.Add(0x05);
ushort startAddress = 149;
byte[] startAddressByte = BitConverter.GetBytes(startAddress);
if (BitConverter.IsLittleEndian)
{
    bytes.Add(startAddressByte[1]);
    bytes.Add(startAddressByte[0]);
}
else
{
    bytes.Add(startAddressByte[0]);
    bytes.Add(startAddressByte[1]);
}
ushort value = 255;
byte[] valueByte = BitConverter.GetBytes(value);
if (BitConverter.IsLittleEndian)
{
    bytes.Add(valueByte[1]);
    bytes.Add(valueByte[0]);
}
else
{
    bytes.Add(valueByte[0]);
    bytes.Add(valueByte[1]);
}

bytes.AddRange(NModbus.Utility.ModbusUtility.CalculateCrc(bytes.ToArray()));

Send(bytes.ToArray());
```



```tex
Tx:006-03 06 00 95 00 FF D8 44
Rx:007-03 06 00 95 00 FF D8 44
```

**ModbusSlave设置**

![image](https://img2022.cnblogs.com/blog/1037641/202209/1037641-20220928015616944-1982131226.png)


**ModbusPoll设置**

![image](https://img2022.cnblogs.com/blog/1037641/202209/1037641-20220928015625716-1663432643.png)


下面是读取多个保持寄存器的值时响应的Modbus协议帧。

**发送报文**

| 设备地址 | 功能码 | 起始地址(高位) | 起始地址(低位) | 寄存器数(高位) | 寄存器数(低位) | CRC校验低位 | CRC校验高位 |
| -------- | ------ | -------------- | -------------- | -------------- | -------------- | ----------- | ----------- |
| 0x07     | 0x03   | 0x00           | 0xC8           | 0x00           | 0x02           |             |             |

**响应报文**

| 地址 | 功能码 | 数据域字节数 | 数据1（高位） | 数据1(低位) | 数据2（高位） | 数位2（低位） | CRC校验低位 | CRC校验高位 |
| ---- | ------ | ------------ | ------------- | ----------- | ------------- | ------------- | ----------- | ----------- |
| 0x07 | 0x03   | 0x06         | 0x03          | 0x53        | 0x01          | 0xF3          |             |             |

上述表格报文，是Modbus设备发送响应报文的字节顺序，先发送地址，再发送功能码，然后发送数据域字节数，紧接着发数据高位，低位.....。上位机作为接收方，先收到地址，再接收到功能码，然后数据域字节数......

```csharp
byte[] recvedBuff = null;

// 应先进行CRC校验，校验步骤省略

int devAddr = recvedBuff[0];
int functionCode = recvedBuff[1];
int dataByteCount = recvedBuff[2];

byte[] data = new byte[dataByteCount];
Array.Copy(recvedBuff, 3, data, 0, data.Length);

// 如果两个地址都是Int16类型
List<Int16> int16List = new List<Int16>();

for (int i = 0; i < data.Length; i+=2)
{
    if (BitConverter.IsLittleEndian)
    {
        int16List.Add(IPAddress.NetworkToHostOrder(BitConverter.ToInt16(data, i)));
    }
    else
    {
        int16List.Add(BitConverter.ToInt16(data, i));
    }
}

// 如果两个地址表示一个float
List<float> floats = new List<float>();
for (int i = 0; i < data.Length; i += 4)
{
    if (BitConverter.IsLittleEndian)
    {
        // ABCD 此处值演示ABCD,对于其他形式CDBA，BADC,DCBA不再做演示
        byte[] temp = new byte[4];
        temp[0] = data[i + 3];
        temp[1] = data[i + 2];
        temp[2] = data[i + 1];
        temp[3] = data[i + 0];
        floats.Add(BitConverter.ToSingle(temp, 0));
    }
    else
    {
        floats.Add(BitConverter.ToSingle(data, i));
    }
}

假设Modbus读取保持寄存器的多个地址如0,1,2,3，返回的Modbus帧，从左至右，依次是0号地址的字，1号地址的字，2号地址的字，3号地址的字。
NModbus把一个字解析成UShort,它怎么做到的呢？其实很简单，以0号地址的字为例，根据Modbus协议帧的定义，一个字的两个字节高位在前，低位在后，NModbus收到一个字的2个字节，认为前面是UShort的高位，后面是UShort的低位。把UShort拆分成字节也很简单，利用左移或右移，拿到UShort的高位和低位，则高位和低位就是Modbus协议帧中的一个地址对应的字的数据高位和数据低位。
NModbus
```
# 如何看待发送图片

对于图片，文档之类的媒体类型的文件，和大端小端没关系！假设调用API拿到一张Bitmap的字节数组，发送方直接发送数组就行了，接收方接到数组后，直接遍历数组即可。
