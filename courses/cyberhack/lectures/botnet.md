# Botnet 

## Botnet 取证


### Botnet IOC （Indicators of Compromise）提取

> 360 网络安全研究院 安全专家 刘亚、金晔


Botnet IOC （Indicators of Compromise）提取是对抗Botnet的重要一环，无论是后续的封堵还是跟踪，都依赖从样本中提取到的C2这类IoC信息。目前最常用的IoC自动提取技术是沙箱。

#### 背景
Botnet的狭义的IOC 指的是：域名、ip、端口。广义的指：样本的md5哈希值、配置信息等。


目前有两种IOC提取技术：
- 基于静态提取（小众），适用于特征明显，C2存储格式固定。
- 基于沙箱的提取
- 基于动态分析的LWE（轻量级仿真）的提取

某些家族因为IoC在样本中的保存格式和位置相对固定，可以用静态读取的方式来抽取。由于这种技术不需要执行指令，因此我们称之为静态提取。静态特征不太多的时候可以使用。但更多的还不能通过静态的方式提取出IOC。

从技术层面分析，沙箱技术使用广泛相对成熟，技术门类也更加丰富，但却存在一些经典的问题，比如：对抗（sandbox evasion）、通信加密等。

比较典型的分析手段是将样本置入沙箱（例如ANY RUN, cuckoo）中去运行，运行几分钟，在此期间收集各种进程、网络行为。提取报文特征（pcap文件），找到对应的session，把对应的域名和ip提取出来。

botnet可能会识别沙箱然后进行沙箱逃逸，或者botnet中有多个C2，不容易在短期提取出完整的C2。

LWE本质上属于动态分析，分析对象是一段代码，分析它的行为。这种方式比静态分析要更加稳定和可靠。

#### 轻量级仿真（LWE）

||沙箱提取IOC技术|LWE提取IOC技术|
|-|-|-|
|是否动态分析技术|是|是|
|处理对象|PE,ELF,DOC,PDF,...|一段代码|
|指令级控制|非必须|必须（为更细粒度的控制）|
|系统服务|提供|不提供，或部分提供|
|输出|行为报告，网络PCAP|指令执行记录，CPU寄存器值，内存镜像|


##### 开源轻量级仿真引擎的选择
- libemu、pyemu、x86emu

均实现了cpu指令解析和基本的内存管理
差别在于指令完整度和特殊指令不支持
支持的cpu比较单一


- unicorn

BHUSA2015大会提出。
脱胎于QEMU，支持x86/x64/arm/mips，非常适合IoT botnet分析处理。
支持细粒度控制。
支持多种语言编程。

Unicorn的Python 编程步骤：
- 初始化
  - 设置cpu类型
  - 映射内存、包括代码和数据
  - 设置初始寄存器值
  - 安装hook回调函数：指令控制、内存读写

- 制定初始运行地址，开始运行
- 读取结果，输出
  - 读特征寄存器
  - 读内存

##### 提取mirai的IOC
Mirai 2016年出现，造成了一系列的断网事件：
- DYN断网
- 德国电信断网

因为代码泄露，变种泛滥；代码被别的家族所借鉴，例如Gafgyt、Tsunami。

Mirai极具特点的加密配置机制往往会被一并借鉴。

仿真 table_init()获取配置信息的关键点：
- 拦截call，获取参数
  - ```call_to_malloc(arg0)```，通过arg0获取密文长度
  - ```call_to_memcpy(arg0,arg)```


##### 提取gafgyt的IOC
##### 提取emotet的IOC