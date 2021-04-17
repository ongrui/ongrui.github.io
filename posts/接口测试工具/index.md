# 接口测试小工具


- 开篇啰嗦

这篇废话太多了，做完了新开一篇总结的

之前也写过接口测试工具，不过测试数据是放到Excel里面维护的

前段时间和两个测试大佬聊，他们好像不太看好把测试数据维护到Excel里面

所以现在利用业余时间重新做一个，把数据放到配置文件里面

思路和过程还有进度就展示在这里了

啥时候有雏形的时候就放到`github`上去



## 2021.04.17 12:40更新

## 已经完成的

- 1.读取配置文件
- 2.发送请求
- 3.把指定的返回值拼接到指定的`KEY`上

## 下一步的计划

- 1.对指定的返回值做判断
- 2.写一个日志模块
- 3.把`case`的`val`换成`yaml`格式的（这周看到了一个yaml的详细教程，不是不好解析，而是我不会:sweat:）

## `Github`地址

`https://github.com/ongrui/code200`



## 2021.04.10 23:38

周五开始做的，目前先把配置文件的思路讲一下

### 配置文件

- 接口地址放到一个`yaml`文件中，每个接口的case放到一个`yaml`文件中

#### URL存放的配置文件

- `conn.yaml`

```yaml
url:
  - url: http://127.0.0.1/corexapi/oauth/token
    case: case1,case2
  - url: http://127.0.0.1/corexapi/external/pacs/cancelRegister
    case: case3,case4
```

- `url`：要测试的接口地址

- `case`：每个URL要执行的case，可以是多个值，对应下面的`case.yaml`的第一个`key`

#### case存放的配置文件

- `case.yaml`

```yaml
case1:
  method: GET
  header:
    Content-Type: application/json
  bodyType: params
  body:
    test: 1
    token: 
  respParame:
    data,access_token: case2,header,token
case2:
  method: POST
  header: json
    Content-Type: application/json
    token: 
  bodyType: 
  body:
    test: 
  respParame:
```

- `method`：请求方式
- `header`：请求头，下面是请求头的内容，成对写入
- `bodyType`：`body`类型
- `body`：`body`内容
- `respParame`：目的是把返回值中想要的参数拼接到其它`case`上
  - `data,access_token`：为返回参数的`key`，要支持嵌套结构，比如下面的例子，`token`在`data`一层里面的时候，也可以获取到
  - `case2,header,token`：取到想要的参数后，把该参数拼接到的地方

```json
{
    "success":true,
    "code":200,
    "msg":"请求成功",
    "data":{
        	"access_token":"96c0549c-f56c-4991-b351-8dad3adb40c2",
        	"token_type":"bearer",
        	"refresh_token":"de2ee487-6036-4770-a0a0-898086199157",
       		"expires_in":86399,
       		"scope":"read write"
        	}
}
```

### 工作流程

- 先读取`conn.yaml`，然后去遍历对应的`case`，拿到请求后调用请求方法，把请求对象传进去，然后再对返回值做处理

## 2021.04.12 20:22

### 今日工作汇报

领导把我薅到济南分部来了，今天跟领导沟通了挺多关于服务端质量保障体系相关的东西

在这边儿倒是停清闲，不过宾馆桌子太小了，还没插座:sob:，明天在公司多呆会儿

{{< image src="/img/6EB9B7A5-619F-4A43-B634-0B1C01A19CDA.jpeg" src_s="/img/6EB9B7A5-619F-4A43-B634-0B1C01A19CDA.jpeg" src_l="/img/6EB9B7A5-619F-4A43-B634-0B1C01A19CDA.jpeg" >}}

{{< image src="/img/1CF695B2-456D-45A9-B80B-9639E8B46CAC.jpeg"  src_s="/img/1CF695B2-456D-45A9-B80B-9639E8B46CAC.jpeg" src_l="/img/1CF695B2-456D-45A9-B80B-9639E8B46CAC.jpeg" >}}

嘿嘿，可以没有鼠标，但是不能没有键盘

废话不说了，汇报工作正式开始

#### 修改`yaml`配置文件

- 1.我把`case`的`val`改成了`json`形式的，`yaml`不好解析，改完就是下面这样

```yaml
case1: '{
        "method":"POST",
        "header":{
            "content-Type":"application/json"
        },
        "bodyType":"form-data",
        "body":{
            "sourceSystem": "corex",
            "stAppno": "0xx83hd8j3972",
            "stGrpNo": "2",
            "rpFindings": "测试所见",
            "rpInspectCon": "测试结论",
            "osrContent": "",
            "rpKeyword": "",
            "rpChkUserId": "manager",
            "rpChkUsername": "管理员",
            "rpChkDt": "2021-01-01T18:42:40.000",
            "rpPositivePpt": "",
            "rpCriticalGrade": "",
            "tisID": "df3192o4icu14e4daf9jf83781f6b7a1"
        },
        "files":{
            "file":"test.png",
            "filePath":"../files/test.png"
        }
    }'
```

#### 配置文件读取相关代码

- 2.今天把读取配置文件的写完了，贴个图吧，写的太烂，面向`struct`编程理解的太浅（看图得新建个标签页:joy:）

{{< image src="/img/image20210412215219.png">}}

## 2021.04.17 16:01

完成了读取配置文件的业务、请求业务，还可以将`json`格式的`response`中指定`KEY`的值写入到其它的请求体或者请求头里面去

这周看到了一篇关于`YAML`的详细文档，发现也很好解析，当时只是我自己菜，没有去网上仔细搜下:disappointed_relieved:，回头再改回`yaml`格式的​

我的垃圾代码已经提交到了`github`上了，算是做个记录吧`https://github.com/ongrui/code200`

还有关于文章的

图片可以放大了，把文章头部预览去掉了
