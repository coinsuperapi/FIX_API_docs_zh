# FIX API

简介

```
Exchange API 是交易所面向外部开发者提供的开发接口。
```

------

更新历史

| 版本        | 时间       | 说明   | 作者     |
| --------- | -------- | ---- | ------ |
| v1.0.0(α) | 20190114 | 创建文档 | liyong |

------

## 一、全局规则

#### 接口访问地址信息

- 接口访问地址：apifix.coinsuper.com
- 接口访问端口：1443
- 接口访问协议：TCP(SSL)

#### 接口规则

- 采用SSL socket长连接访问
- 接口请求均采用 FIX(4.4) 协议
- 接口统一请求数据满足FIX协议数据格式
- 接口统一返回数据满足FIX协议数据格式
- 当服务端接收到用户请求后，会在稍后进行相应，具体的响应格式参考下列参数列表
- 接口调用前均需要登录，登录方式及参数请参考下列参数列表

##### 全局数据格式定义

参数约定 

```
1. 参数说明：
   1.1请求参数：
   客户端固定参数：
   BeginString=FIX.4.4
   TargetCompID=COINSUPER
   HeartBtInt=30
   1.2响应参数：
   具体响应信息请参考下方接口列表，异常说明请参考下方的【全局通用状态码】表。
2. 业务参数精度：
   2.1若symbol为币币交易，则价格精度为小数点后8位，数量参数表示精度为4位；
   2.2若symbol为法币交易，则价格精度为小数点后2位，数量参数表示精度为4位；
   2.3在请求参数中，若参数格式不符合要求，系统将根据自行统一精度。
3. 签名:
   所有请求均需要按本文档中的签名规则传递参数签名，签名生成规则请参考下方的参数签名规范；
4. 其它行情数据等可通过REST接口或WebSocket接口获取；
5. 发送请求之前，需先发送Logon消息进行登录；
```

##### 登录签名描述 

```
交易所将对请求数据的内容进行验签，以确定携带的信息是否未经篡改，因此定义生成 sign 字符串的方法。

a. Logon请求中，将SendingTime,MsgType,MsgSeqNum,SenderCompID,TargetCompID,$secretKey将参数按参数名从小到大排序，并用","连接进行组合md5加密签名生成签名sign;

b. Logon请求中，SendingTime,MsgType,MsgSeqNum,SenderCompID,TargetCompID为必传字段；

c. 为了账户安全，Logon请求中，请勿传递$secretKey；

d. Logon请求中，将sign作为RawData设置到Logon的message中，sign中的字母请采用小写;

注：
SenderCompID=$accessKey；
$accessKey和$secretKey通过开通API获得
```

[开通API](https://www.coinsuper.com/setting/apiCenter)

#####  签名生成方法示例 

```
假设有一个调用请求，其中，
$accesskey : zhangsan，$secretkey : zhangsan，TargetCompID : SERVERTARGET

Logon请求参数如下:
"MsgSeqNum" -> "1"
"MsgType" -> "A"
"SenderCompID" -> "zhangsan"
"SendingTime" -> "20181228-13:08:08.091"
"TargetCompID" -> "SERVERTARGET"

i：经过 a 过程排序后的字符串 string 为：
1,A,zhangsan,20181228-13:26:54.497,SERVERTARGET,zhangsan

ii：经过 b 过程后得到 sign 为 :
signValue = md5(string);

完整请求数据 :
8=FIX.4.49=11735=A34=149=zhangsan52=20181228-13:26:54.49756=SERVERTARGET95=3296=74c544ec967aae34fe84a30bae59520798=0108=3010=188
```

#### FIX请求标准消息头

注：每个接口请求必须传递的消息参数，具体使用方式请参考下方接口定义及请求示例

| 字段名          | 字段Code | 填写类型 | 描述                            |
| ------------ | ------ | ---- | ----------------------------- |
| BeginString  | 8      | 必填   | FIX协议版本(固定值：FIX.4.4)          |
| BodyLength   | 9      | 必填   | 消息长度的字节数. 永远在整条消息的第二个字段(不加密). |
| MsgType      | 35     | 必填   | 请求消息类型                        |
| MsgSeqNum    | 34     | 必填   | 整型消息序列号.                      |
| SenderCompID | 49     | 必填   | $accesskey，通过开通API获得          |
| SendingTime  | 52     | 必填   | 请求发送时间(UTC时间)                 |
| TargetCompID | 56     | 必填   | 服务端标识符，请参考上方[登录签名描述]          |

####  全局通用异常信息 

| 异常text                       | 异常描述                           |
| ---------------------------- | ------------------------------ |
| user not exist!              | 错误的TargetCompID                |
| has no authentication        | 无对应的接口访问权限                     |
| request too frequently       | 请求超过限流值                        |
| SendingTime accuracy problem | 客户端UTC时间和服务端UTC时间不匹配(请对照标准世界时) |
| system internal error        | 系统内部异常                         |

注：异常信息通常以Reject响应返回；各接口有更详细的异常信息定义；

------



## 二、详细接口定义

### API 接口定义

#### 1. 会话类

##### 1.1 登录

功能描述: 

登录请求（会话长连接由登录成功时创建，其他所有请求都需依赖登录成功）

请求MsgType:

```
Logon(A)
```

接口请求参数:

| 字段名          | 字段Code | 填写类型 | 描述                          |
| ------------ | ------ | ---- | --------------------------- |
| SendingTime  | 52     | 必填   | 请求发送时间(UTC时间)               |
| MsgType      | 35     | 必填   | 消息类型                        |
| MsgSeqNum    | 34     | 必填   | 消息序列号                       |
| SenderCompID | 49     | 必填   | API用户accessKey              |
| TargetCompID | 56     | 必填   | 固定参数，请使用上方[参数约定]中描述的值       |
| RawData      | 96     | 必填   | 登录参数签名( 签名规则请参考上方[登录签名描述] ) |

响应MsgType：

```
Logon(A)
```

响应参数:

| 字段名          | 字段Code | 描述         |
| ------------ | ------ | ---------- |
| ClOrdID      | 11     | 用户的编号      |
| OrderID      | 37     | 用户邮箱       |
| ExecType     | 150    | 系统时间戳(毫秒数) |
| OrdStatus    | 39     | 返回结果       |
| TransactTime | 60     | 总余额        |

请求示例：  

```
8=FIX.4.49=11435=A34=249=zhangsan52=20190102-03:41:14.32956=COINSUPER95=3296=6cc719376923d980cbb5c882191d4e2898=0108=3010=058
```

  返回示例: 

```
8=FIX.4.49=7235=A34=449=COINSUPER52=20190102-03:41:14.46156=zhangsan98=0108=3010=011
```

------

##### 1.2 注销

功能描述: 

注销请求，发起后将会关闭会话长连接

请求MsgType:

```
Logout(5)
```

接口请求参数:

无

响应MsgType：

```
Logout(5)
```

响应参数:

无

请求示例：  

```
8=FIX.4.49=11435=534=249=zhangsan52=20190102-03:41:14.32956=COINSUPER95=3296=6cc719376923d980cbb5c882191d4e2898=0108=3010=058
```

  返回示例: 

```
8=FIX.4.49=7235=534=449=COINSUPER52=20190102-03:41:14.46156=zhangsan98=0108=3010=011
```

------

##### 1.3 心跳

功能描述: 

心跳请求，固定时间发送，用于维持会话长连接，服务端30秒内仅会响应一次

请求MsgType:

```
Heartbeat(0)
```

接口请求参数:

无

响应MsgType:

```
Heartbeat(0)
```

响应参数:

无

请求示例:

```
8=FIX.4.49=6035=034=349=zhangsan52=20190102-07:31:01.57256=COINSUPER10=223
```

 返回示例: 

```
8=FIX.4.49=6235=034=46349=COINSUPER52=20190102-07:31:01.69056=zhangsan10=076
```

------

#### 2. 交易类

##### 2.1 下单委托

功能描述: 

交易下单委托请求

请求MsgType:

```
NewOrderSingle(D)
```

接口请求参数:

| 字段名          | 字段Code | 填写类型 | 描述                                |
| ------------ | ------ | ---- | --------------------------------- |
| ClOrdID      | 11     | 必填   | 客户端自定义的订单ID(不能重复)                 |
| Symbol       | 55     | 必填   | 交易对                               |
| Price        | 44     | 必填   | 成交限价(限价单专用，市价单请传入0)               |
| Side         | 54     | 必填   | 买卖类型( BUY(1)=买单，SELL(2)=卖单 )      |
| OrdType      | 40     | 必填   | 委托单类型( MARKET(1)=市价，LIMIT(2)=限价 ) |
| OrderQty     | 38     | 必填   | 标的币数量(限价买，限价卖，市价卖时使用。市价买请填0)      |
| CashOrderQty | 152    | 必填   | 计价币数量(市价买时使用。限价买，限价卖，市价卖时请填0)     |
| TransactTime | 60     | 必填   | 请求时间(UTC时间)                       |

```
参数附加说明：

BTC/USD交易对，OrderQty和Amount使用举例：

6300 USD限价买0.1 BTC：Symbol=BTC/USD,Side=1,OrdType=2,Price=6300,OrderQty=0.1,CashOrderQty=0;

6301 USD限价卖0.2 BTC：Symbol=BTC/USD,Side=2,OrdType=2,Price=6300,OrderQty=0.2,CashOrderQty=0;

500 USD市价买BTC：Symbol=BTC/USD,Side=1,OrdType=1,Price=0,OrderQty=0,CashOrderQty=500;

市价卖0.5 BTC：Symbol=BTC/USD,Side=2,OrdType=1,Price=0,OrderQty=0.5,CashOrderQty=0;
```

响应MsgType：

```
ExecutionReport(8)
```

响应参数:

| 字段名          | 字段Code | 描述              |
| ------------ | ------ | --------------- |
| ClOrdID      | 11     | 客户端自定义的订单ID     |
| OrderID      | 37     | 服务端生成的订单ID      |
| ExecType     | 150    | 执行结果(固定为NEW)    |
| OrdStatus    | 39     | 委托状态(固定为NEW)    |
| TransactTime | 60     | 响应消息发送时间(UTC时间) |

请求示例：  

```
8=FIX.4.49=16735=D34=3249=zhangsan52=20190104-10:08:43.31456=COINSUPER11=ord000138=0.340=244=450054=255=BTC/USD60=20190104-18:08:43.308152=010=091
```

  返回示例: 

```
8=FIX.4.49=22335=834=18349=COINSUPER52=20190104-10:08:42.34256=zhangsan6=014=017=a0cbfd23455e4b6faaacbe5fb36caf9920=037=162172399493790924939=054=255=BTC/USD60=20190104-18:08:42.341150=0151=0.310=036
```
| 异常text                                   | 异常描述       |
| ---------------------------------------- | ---------- |
| symbol not trading                       | 交易对不能交易    |
| order amount or quantity less than min setting | 交易数量小于要求的值 |
| price out of range                       | 价格超出范围     |
| action not support                       | 不支持的交易类型   |
| order type not support                   | 不支持的订单类型   |
| user account forbidden                   | 账户禁止交易     |
| balance not enough                       | 余额不足       |

------

##### 2.2 撤单委托

功能描述: 

交易撤单委托请求

请求MsgType:

```
OrderCancelRequest(F)
```

接口请求参数:

| 字段名     | 字段Code | 填写类型 | 描述         |
| ------- | ------ | ---- | ---------- |
| OrderID | 37     | 必填   | 服务端生成的订单ID |

响应MsgType：

```
ExecutionReport(8)
```

响应参数:

| 字段名          | 字段Code | 描述                         |
| ------------ | ------ | -------------------------- |
| OrderID      | 37     | 服务端生成的订单ID                 |
| ExecType     | 150    | 执行结果(撤单处理中：PENDING_CANCEL) |
| OrdStatus    | 39     | 委托状态(撤单处理中：PENDING_CANCEL) |
| TransactTime | 60     | 响应消息发送时间(UTC时间)            |

请求示例:  

```
8=FIX.4.49=11235=F34=6549=zhangsan52=20190104-09:40:25.49556=COINSUPER37=162172076101837209710=112
```

  返回示例: 

```
8=FIX.4.49=15435=834=12749=COINSUPER52=20190104-09:40:24.38956=zhangsan20=137=162172076101837209739=460=20190104-17:40:24.389150=410=041
```

各种原因失败时响应参数 Text 中取值：

| 异常text             | 异常描述     |
| ------------------ | -------- |
| order no not exist | 未完成订单不存在 |
| order has canceled | 订单已撤销    |
| order has execute  | 订单已成交    |

------

#### 3. 信息查询类

##### 3.1 查询未完成订单

功能描述: 

查询未全部成交且未撤销的委托单

请求MsgType:

```
OrderStatusRequest(H)
```

接口请求参数:

| 字段名     | 字段Code | 填写类型 | 描述                         |
| ------- | ------ | ---- | -------------------------- |
| OrderID | 37     | 必填   | 服务端生成的订单ID(*表示查询最近20条下单信息) |

响应MsgType:

```
ExecutionReport(8)
```

响应参数:

| 字段名          | 字段Code | 描述                                 |
| ------------ | ------ | ---------------------------------- |
| OrderID      | 37     | 服务端生成的订单ID                         |
| Side         | 54     | 买卖类型( BUY(1)=买单，SELL(2)=卖单 )       |
| ExecType     | 150    | 执行结果(ORDER_STATUS)                 |
| OrdStatus    | 39     | 委托状态(PENDING_NEW/PARTIALLY_FILLED) |
| AvgPx        | 6      | 订单成交均价                             |
| CumQty       | 14     | 已成交数量                              |
| LeavesQty    | 151    | 剩余未成交数量 （CumQty+LeavesQty=下单时总数量）  |
| Symbol       | 55     | 交易对                                |
| TransactTime | 60     | 响应消息发送时间(UTC时间)                    |

请求示例:

```
8=FIX.4.49=11235=H34=1849=zhangsan52=20190104-10:01:52.86256=COINSUPER37=162172183852353945710=111
```

返回示例: 

```
8=FIX.4.49=22335=834=16949=COINSUPER52=20190104-10:01:51.70456=dba8ef1f-6e3f-43ed-ae67-0668c3e933636=014=017=37e01f58367948fdaedf8dace2ea62ff20=337=162172183852353945739=A54=255=BTC/USD60=20190104-18:01:51.704150=I151=0.210=199
```

各种原因失败时响应参数 Text 中取值：

| 异常text             | 异常描述     |
| ------------------ | -------- |
| order no not exist | 未完成订单不存在 |

