# DICOM


## 什么是DICOM

DICOM，**D**igital **I**maging and **Co**mmunications in **M**edicine 医学数字成像和通信标准

## 通信

DICOM协议是以TCP/IP为基础的协议

该协议有三个部分

### 1.DICOM服务类

- DICOM Service Class

DICOM服务类完成了一些dicom中的功能

- IOD

Information Object Definition，信息对象定义，对具有相同属性的一类事务的抽象



服务类描述了有关指令和数据的语义，它和信息对象IOD共同构成了DICOM的基本单元，称为服务对象对SOP(Service-Object Pair)

一个SOP类的实现称为SOP实例。下表列出了常用的服务类以及相关的功能

| 服务类名称           | 功能描述                       |
| -------------------- | ------------------------------ |
| Image storage        | 用于医学对象的存储             |
| Image find           | 用于查找图像的信息             |
| Image move           | 提供图像的接收和转发服务       |
| Image query/retrieve | 提供图像的检索查询服务         |
| Result management    | 提供对检查、诊断结果的管理服务 |
| Image Print          | 提供图像的打印管理服务         |

### 2.DIMSE消息服务元素

DICOM Message Service Element

消息服务元素为对等DICOM应用实体进行医学影像及相关信息交换提供了一种应用服务元素定义，包括服务和协议

协议：

- DIMSE是基于DIMSE协议来提供服务，DIMSE协议规定了构造消息必须的编码规则。一条DICOM消息由指令集合（Command Set），和数据集合（Data Set）构成

服务：

- DIMSE服务因操作SOP类型的不同分为DIMSE-C Services和DIMSE-N Services
- DIMSE-C：作用对象是复合型IOD，主要用于完成医学影像的传输或交换
  - 复合型IOD：是对现实世界中对各相关的实体内在属性的组合，例如一个dcm图像，不仅包含了像素、日期等内在属性、还包含了检查信息和患者信息的一些外在属性
- DIMSE-N：作用对象是普通型IOD，主要为了完成影响设备间的相互操作，如胶片打印
  - 普通型IOD：用来表示单个实体的内在属性，例如病人IOD就是一个普通IOD，包含了患者姓名、性别、年龄等信息
- DIMSE-C服务主要包括：
  - C-ECHO：获取对等段回应
  - C-FIND：查询患者信息（worklist）
  - C-STORE：图像归档
  - C-MOVE：转存或获取病人图像信息
  - C-GET：为属性值检索匹配的SOP实例
- DIMSE-N服务主要包括：
  - N-EVENT-REPORT：报告状态
  - N-GET：检索属性值
  - N-SET：设置参数
  - N-CREATE：生成SOP实例
  - N-ACTION：触发服务过程
  - N-DELETE：删除SOP实例

DIMSE中所有的操作和通知都是确认服务，即一方发出的请求都需要得到对方的应答，如C-FIND-RQ是请求，C-FIND-RSP是应答

### 3.DICOM上层协议ULP

Upper Layer Protocol，上面学习到了DICOM协议是基于TCP/IP的，最底层的则是ULP，ULP主要负责和TCP对接

当应用实体通过DIMSE完成消息交换后，需要上层协议ULP来提供传输支持，他的作用主要包括：

- 传送和接收DIMSE命令流（Command Set）和数据流（Data Set），提供网络传输支持
- 提供应用层的连接控制服务，如建立连接和释放连接

ULP提供了5中连接控制服务

- A-ASSOCIATE：用来在通信实体双方间协商并建立关联，交换一些初始化信息，该服务是一个证实性服务
- A-RELEASE：用于在完成传输后释放关联，它是一个正常的终止方式，不会造成数据的丢失，也是一种证实性服务
- A-ABORT：当通信出现异常时用来终止关联，有可能造成数据丢失，非证实性服务
- P-DATA：用来传输DIMSE命令流和数据流，通信实体一方一旦把DIMSE消息流传送出去就认定另一方能够准确无误的收到，是非证实性服务，类似UDP
- A-P-ABORT：也是用来终止关联，主要是用于当网络连接失败时，使得上层能够及时获得操作相应信息

ULP提供了7个协议数据单元来实现上述的5中服务

- A-ASSOCIATE-RQ：建立ASSOCIATE请求
- A-ASSOCIATE-AC：接受ASSOCIATE建立
- A-ASSOCIATE-RJ：拒绝ASSOCIATE建立
- P-DATA-TF：进行数据传输
- A-RELEASE-RQ：ASSOCIATE释放请求
- A-RELEASE-RP：ASSOCIATE释放响应
- A-ABORT：ASSOCIATE终止

### 示例

该截图是在wireshark中抓到的设备刷worklist的包

![TajXgf.png](https://s4.ax1x.com/2021/12/25/TajXgf.png)

- 328号包是设备请求RIS的包，info信息可以看到，和上面学习到的一样，是建立连接，FLC1270和MOZI是设备和RIS的AET
- 629号包是RIS返回给设备的包，info信息中显示接受了连接建立
- 630号包是设备请求RISworklist的数据包，info中的P-DATA和C-FIND-RQ表示请求worklist数据
- 645号包是RIS返回给设备的worklist数据，通过info中的信息C-FIND-RSP可以看到是返回worklist数据
- 650号包是释放连接请求，A-RELEASE-RQ
- 653号包是释放连接响应，A-RELEASE-RP。至此一次通信就结束了
- 666号包是第二次通信建立连接的包



## worklist

现在在现场上PACS，因为这家医院信息化建设比较落后，所以设备没有接过worklist服务，我们是第一家，上一次写的刷worklist的方法在这儿也不顶用了，这边设备不是很多，但是接一台2013年的西门子数字胃肠时卡住了，接了好久，终于昨天弄好了，看到设备上刷到检查列表的那一瞬间，激动兴奋难以言表。

通过这次经历我学习到了worklist返回的属性是有具体的结构的，也学习了dcmtk的使用，后面再写dcmtk的用法

设备要的worklist数据请求时会发过来，就是上图的630号包，然后通过请求的数据配置上一般都能成功

每个tag的值的类型是不一样的，在https://blog.csdn.net/ljjliujunjie123/article/details/120829908?spm=1001.2014.3001.5501这篇文章中有对dcm格式的详细介绍

还有item序列，一个item的开始必须要有fffe,e000开始，fffe,e00d和fffe,e0dd来结束，VR为SQ的tag的值都是序列，这一点是在dcmtk工具中学习到的



## 学习资料

https://www.jianshu.com/p/6f178fc98a04

https://randomnamer.github.io/StaticBlog/2021/11/11/DICOM-protocol-internals/

