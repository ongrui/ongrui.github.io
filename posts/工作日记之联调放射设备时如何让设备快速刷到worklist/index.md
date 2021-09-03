# 工作日记-让放射设备快速从Dicom服务上刷到worklist的方法


##  联调放射设备时如何一把让设备刷到worklist

## （只适用于替换别家的PACS:grin:）

有些医院有的放射设备维保期过了，或者是没有设备文档的情况下，dicom联调时服务就变得非常麻烦



按照下面步骤基本可以一把成功刷到worklist

- 第一步

  登录现在正在使用的pacs服务器，装一个wireshark

- 第二步

  过滤条件选择ip.addr={设备IP}||dicom

- 第三步

  从设备上请求worklist

- 第四步

  然后查看描述信息为：P-DATA, C-FIND-RQ-DATA的包

  展开DICOM, C-FIND-RSP, C-FIND-RSP-DATA

  然后再展开PDV, C-FIND-RSP-DATA

  就可以看到返回的所有tag信息

- 第五步

  在自己的服务器上对照着上面的抓到的tag信息配置上
  
  完活儿~

注意：有些tag如果返回了但是值为空的话可能会被设备过滤掉，最好与抓到的包中的信息一致



调设备给我人调麻了:sob:


