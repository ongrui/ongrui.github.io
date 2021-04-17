# code200文档


code200的文档说明

## 配置文件

```yaml
case1: '{
        "method":"POST",
        "header":{
            "content-Type":"application/json",
            "Authorization":"Bearer d15a43e2-ade8-4566-95b6-b3f7e7821d63"
        },
        "bodyType":"multipart/form-data",
        "body":{
            "sourceSystem": "corex",
            "stAppno": "200000030139",
            "stGrpNo": "2",
            "rpFindings": "   肝：  形态、大小",
            "rpInspectCon": "    申请检查范围内未见明显异常",
            "osrContent": "",
            "rpKeyword": "",
            "rpChkUserId": "manager",
            "rpChkUsername": "管理员",
            "rpChkDt": "2020-07-01T18:42:40.000",
            "rpPositivePpt": "",
            "rpCriticalGrade": "",
            "tisID": "df319296aa714e4dafef813781f6b7a1"
        },
        "files":{
            "file":"C:/Users/rui/Pictures/Saved Pictures/test.png",
            "file1":"C:/Users/rui/Pictures/Saved Pictures/test.png"
        },
        response:{
            "data,token_type&data,access_token":"case3,header,Authorization",
        }
    }'
```

- `method`：参数值：`GET`、`POST`

- `header`：键值对

- `bodyType`：参数值：

  - `urlParams`：如果是`URL`上的参数写到`body`中的时候，`bodyType`需要写入该值
  - `json`：`json`形式的请求体
  - `multipart/form-data`：表单格式的请求体

- `body`：键值对，支持嵌套

  - `GET`：`url`上的键值对可以写到`body`上，不能嵌套
  - `POST`：支持嵌套的`json`类型
  - 表单格式：键值对的参数可以写到`body`上，如果有文件上传，写到下面的`files`

- `files`：键为文件的`key`，值为文件地址，可以写多个键值对，不能嵌套

- `response`：将`json`返回值的参数取出，写入到其它`case`的参数中

  - 一个参数对应一个参数，取`data,token_type`中的参数放到`case3,header,Authorization`中

  - ```json
    response:{
    	"data,token_type":"case3,header,Authorization",
    }
    ```

  - 多个参数对应一个参数，取`data,token_type`和`data,access_token`中的值拼接起来放到`case3,header,Authorization`中

  - ```json
    response:{
    	"data,token_type&data,access_token":"case3,header,Authorization",
    }
    ```


