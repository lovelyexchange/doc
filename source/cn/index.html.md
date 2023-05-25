---
title: Coinstore官方API文档

language_tabs: # must be one of https://git.io/vQNgJ
  

toc_footers:

includes:
  
language: 简体中文

other_language: English

present_url: /cn

url: /en

active: active

other_active:

menu: 菜单

create_api: 创建 API Key

spot_goods: 现货

spot_goods_active: active

spot_goods_url: 'index.html'

contract: 永续合约

contract_url: 'futures.html'

searchText: 搜索

search: true

code_clipboard: true
---

# 介绍

欢迎使用Coinstore开发者文档，此文档是Coinstore API的唯一官方文档。

本文档提供了相关API的使用方法介绍。

RESTful API包含了资产，订单及行情等接口。

Websocket则提供了行情相关的接口及推送服务。

Coinstore API提供的服务会在此持续更新，请大家及时关注。



# 快速开始

## 接入准备

如需使用API，请先登录网页端，通过【用户中心】-【API管理】创建一个API key，再据此文档详情进行开发和交易。

您可以点击 'https://www.coinstore.com/#/user/bindAuth/ManagementAPI' 创建 API Key。

每个用户可创建5组API Key，每组API key可以绑定5个不同的IP地址。API key一旦绑定了IP地址，则只能从绑定的IP地址使用该API key调用API接口。出于安全考虑，强烈建议您为API key绑定相应的IP地址。

创建成功后请务必记住以下信息：

- `API Key`  API 访问密钥
- `Secret Key` 签名认证加密所使用的密钥

## 接口类型
Coinstore为用户提供两种接口，您可根据自己的使用场景和偏好来选择适合的方式进行查询行情、交易。

**REST API**

REST，即Representational State Transfer的缩写，是一种流行的互联网传输架构。它具有结构清晰、符合标准、易于理解、扩展方便的，正得到越来越多网站的采用。其优点如右：

- 在RESTful架构中，每一个URL代表一种资源；
- 客户端和服务器之间，传递这种资源的某种表现层；
- 客户端通过四个HTTP指令，对服务器端资源进行操作，实现“表现层状态转化”。 

交易或资产等一次性操作，建议开发者使用REST API进行操作

**WebSocket API**

WebSocket是HTML5一种新的协议（Protocol）。它实现了客户端与服务器全双工通信，通过一次简单的握手就可以建立客户端和服务器连接，服务器可以根据业务规则主动推送信息给客户端。

市场行情和买卖深度等信息，建议开发者使用WebSocket API进行获取。

### 接口鉴权

以上两种接口均包含公共接口和私有接口两种类型。

公共接口可用于获取基础信息和行情数据。公共接口无需认证即可调用。

私有接口可用于交易管理。每个私有请求必须使用您的API Key进行签名验证。

## 接入URLs

**REST API**

`https://api.coinstore.com/api`

**WebSocket**

`wss://ws.coinstore.com/s/ws`

为保证API服务的稳定性，建议使用日本AWS云服务器进行访问。如使用中国大陆境内的客户端服务器，连接的稳定性将难以保证。


## <span id="a4">签名认证</span>

**签名说明**

API 请求在通过 internet 传输的过程中极有可能被篡改，为了确保请求未被更改，除公共接口（基础信息，行情数据）外的私有接口均必须使用您的 API Key 做签名认证，以校验参数或参数值在传输途中是否发生了更改。

**签名算法**

使用HmacSHA256哈希函数作为签名函数
```
Mac hmacSha256 = Mac.getInstance("HmacSHA256");
```

**签名步骤**
    
1、签名有效字符串为请求参数和请求体
注意：请求参数和请求体不进行任何排序，直接拼接成字符串作为payload。

 ```java
 String payload = “symbol=aaaa88&size=10”；
 ```

实例1：GET请求查询字符串

 ?symbol=aaaa88&size=10

```java
 String payload = “{"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,
 "timestamp":1627384801051}”；
 ```

实例2 :Post请求体

 {"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,"timestamp":1627384801051}

```java
 String payload = “symbol=aaaa88&size=10{"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,
 "timestamp":1627384801051}”；
 ```

实例3：混合请求
 
 ?symbol=aaaa88&size=10
 {"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,"timestamp":1627384801051}


```java
 String time = String.valueOf(X-CS-EXPIRES / 30000);
 hmacSha256.init(new SecretKeySpec(Secret_Key.getBytes(), "HmacSHA256"));
 byte[] hash = hmacSha256.doFinal(time.getBytes());
 String key = Hex.toHexString(hash);
```

2、使用签名函数对时间戳计算哈希值

>注意： X-CS-EXPIRES为13位时间戳，需要除以30000获取一个类时间戳，对其进行签名函数计算，获得函数值作为第三步的秘钥(第3步中的变量key的值）




```java
 hmacSha256.reset();
 hmacSha256.init(new SecretKeySpec(key.getBytes(), "HmacSHA256"));
 hash = hmacSha256.doFinal(payload.getBytes());
 String sign= Hex.toHexString(hash);
```

3、使用签名函数对有效字符串计算哈希值
 

>注意： key的值为第2步计算出来的哈希值。

**Python示例**

 https://coinstore-sg-encryption.s3.ap-southeast-1.amazonaws.com/filesUpload/ex1/public/coinstore.py

# API接入说明

## <span id="a3">请求格式</span>
所有的API请求都是restful，目前只有两种方法：GET和POST。
- GET请求：所有的参数都在路径参数里
- POST请求: 路径里可以设置参数，参数可以以JSON格式发送在请求主体（body）里，没有参数的需要传{}

一个合法的请求由以下几部分组成：
- 方法请求地址：即访问服务器地址api.coinstore.com，比如https://api.coinstore.com/api/trade/order/place
- 必须和可选参数。
- X-CS-APIKEY： 即用户申请的API Key。
- X-CS-EXPIRES：您发出请求的时间戳。如：1629291143107。
- X-CS-SIGN：签名计算得出的字符串，用于确保签名有效和未被篡改。

**注意：X-CS-APIKEY ，X-CS-EXPIRES ，X-CS-SIGN 三个参数都在请求头中，另外需要设置'Content-Type':'application/json'。**

## <span id="a3">返回格式</span>

所有的接口返回都是JSON格式。在JSON的第一层有3个字段code，msg和data。前两个字段表示请求状态和描述，实际的业务数据在data字段里。

```json
{
    "code": "0",
    "message": "suc",
    "data": // per API response data in nested JSON object
}
```
以下是一个返回格式的样例：

| 字段 | 数据类型  | 描述       |
| ---- | -----  | ---------- |
| code | int | 返回状态 |
| message  | string | 状态或错误描述 |
| data | object | 业务数据  |


## 错误信息

**HTTP状态码**

HTTP常见的错误码如下：
- 400 Bad Request – Invalid request forma 请求格式无效

- 401 signature failed – Invalid API Key 无效的API Key

- 404 service not found 没有找到请求

- 429 too many visits 请求太频繁被系统限流

- 500 internal server error – 服务器内部错误

**业务状态码**

如果失败，response message 带有错误描述信息, 对应的状态码描述如下：

| 状态码 | 说明                             | 备注                       |
| ------ | -------------------------------- | -------------------------- |
| 0      | 成功                             | code=0 成功， code >0 失败 |


# 账户相关


## <span id="1">资产余额</span>

获取用户资产余额

### HTTP请求: 
- POST /spot/accountList


> 响应 

```json
{
    "data": [
        {
            "uid": 315,
            "accountId": 1134971,
            "currency": "USDT",
            "balance": "0.05001",
            "type": 4,
            "typeName": "FROZEN"
        },
        {
            "uid": 315,
            "accountId": 1134971,
            "currency": "USDT",
            "balance": "13297.94999",
            "type": 1,
            "typeName": "AVAILABLE"
        }
    ],
    "code": 0
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|  |  |  |  |

### 响应数据

|       code        |  type  |                  comment                       |
| ---- | ----- |---------- |
| code | int      |   0：成功，其他失败         |
| message | String  | 错误信息 |
| data | Object []  |  业务数据 |
|- userId | Long  | 用户id |
|- accountId | Long  | 账户id |
|- currencyId | int  | 币种id |
|- balance | Decimal  | 金额 |
|- type | Decimal |账户状态 1:可用 4:冻结 |

# 资金相关
## <span id="2">资金划转</span>

#### 目前支持合约<一>现货划转, API域名地址 `https://futures.api.coinstore.com/api`  调用支持ApiKey与Token

### HTTP请求:
- POST /common/account/transfer

### 请求参数

|       param        |  type  |                  required                       |comment|
| ---- | ----- |---------- |-----|
| amount | String      | Y|  划转数量, 数量不正确返回错误码 `102150400`        |
| currencyCode | String      | Y|  划转币种，如`USDT`, 币种不正确或不支持返回错误码 `101040502` 目前支持划转的币种: `USDT`、`BOBC`、`DREAMS`        |
| clientId | String      | N|  客户自定义ID字母（区分大小写）与数字的组合，可以是纯字母、纯数字且长度要在1-32位之间。 格式不符合要求返回错误码 `101040502`       |
| transferType | String      | Y|  划转方向, 合约 -> 现货 : `future-to-spot`, 现货 -> 合约 : `spot-to-future`, 非次两种类型返回错误码 `101040502`     |


> 请求示例

```json
{
    "currencyCode":"USDT",
    "amount":"5",
    "transferType":"spot-to-future",
    "clientId":"SFT1679454569469"
}
```


> 响应数据:失败参数示例:

```json
{
  "code": 101040502,
  "msg": "request field missing",
  "data": null
}
```
> 响应数据:成功:

```json
{
    "code": 200,
    "msg": null,
    "data":0
}
```
### 其他响应code说明

|       code       |  remark                      |
| ---- | ----- |
| 200| 划转成功 |
| 102150400| 划转数量不正确 |
| 101040502| 可能币种名称、自定义ID、划转类型不正确、划转币种不支持 |
| 102150808| 可能余额不足、重复订单 |
| 102150400| 失败 |


# 订单相关

## <span id="2">获取当前订单</span>
获取当前订单

### HTTP请求: 
- GET /trade/order/active

> 响应

```json
{
    "data": [
        {
            "symbol": "LUFFYUSDT",
            "baseCurrency": "LUFFY",
            "quoteCurrency": "USDT",
            "timestamp": 1642402062196,
            "side": "BUY",
            "timeInForce": "GTC",
            "accountId": 1138204,
            "ordPrice": 9.1E-10,
            "cumAmt": "0",
            "cumQty": "0",
            "leavesQty": "0",
            "clOrdId": "b1b2ea5e00a84b888d419cb73f8eb203",
            "ordAmt": "0",
            "ordQty": "1",
            "ordId": "1722183748419690",
            "ordStatus": "SUBMITTED",
            "ordType": "LIMIT"
        }
    ],
    "code": 0
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|  |  |  |  |


### 响应数据

|       code        |  type  |                         comment                       |
| ----------------- | ------ |------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string    |         错误信息                                    |
| data            |  list  |    |
| ├─ ordId            | long      订单id            |
| ├─ clOrdId            | string     |   客户端订单id            |
| ├─symbol | string |   币对 |
| ├─baseCurrency| string |   base币|
| ├─quoteCurrency| string |   quot币|
| ├─ordPrice | string |    订单价格|
| ├─ordQty| string |   订单数量|
| ├─ordAmt| string |   订单金额|
| ├─side | string |    BUY,SELL|
| ├─cumAmt| string | 成交金额  |
| ├─cumQty| string |  成交数量 |
| ├─leavesQty| string |  剩余数量  |
| ├─ordStatus | string |  订单状态 NOT_FOUND,SUBMITTING,SUBMITTED,PARTIAL_FILLED,CANCELED,FILLED |
| ├─ordType | string |   MARKET,LIMIT,POST_ONLY|
| ├─timeInForce| string | GTC,IOC,FOK |
| ├─timestamp| long | 时间戳 |


## <span id="3">获取当前订单V2</span>

获取当前订单 v2 版本

#### 新接口的 API域名地址 `https://api.coinstore.com`  调用支持ApiKey与Token

### HTTP请求:
- GET /api/v2/trade/order/active

> 响应

```json
{
    "data": [
        {
          "baseCurrency": "A123",
          "quoteCurrency": "USDT",
          "side": "BUY",
          "cumQty": "0",
          "ordId": 1758065956225025,
          "clOrdId": "tLCGVA4g19zuEBwsITXi9g3U624Al0Bw",
          "ordType": "LIMIT",
          "ordQty": "10",
          "cumAmt": "0",
          "accountId": 1134912,
          "timeInForce": "GTC",
          "ordPrice": "100",
          "leavesQty": "10",
          "avgPrice": "0",
          "ordStatus": "SUBMITTED",
          "symbol": "A123USDT",
          "timestamp": 1676622290389
        }
    ],
    "code": 0
}
```

### 请求参数

|    code    | type   | required | comment |
| ---------- |--------|--------|---------|
|symbol | string | N      | 交易对     |
|ordId| int    | N      | 委托ID    |
|clOrdId| String | N      | 客户端委托ID  |


### 响应数据

|       code    |  type  | comment                                 |
| ------------- | ------ |-----------------------------------------|
| code          | int    | 0：成功，其他失败                               |
| message       | string    | 错误信息                                    |
| data          |  list  |                                         |
| ├─baseCurrency| string | base币                                   |
| ├─quoteCurrency| string | quot币                                   |
| ├─timeInForce| string | GTC,IOC,FOK                             |
| ├─ordStatus | string | 订单状态 NOT_FOUND,SUBMITTING,SUBMITTED,PARTIAL_FILLED,CANCELED,FILLED |
| ├─avgPrice | string | 成交均价	                                   |
| ├─ordPrice | string | 订单价格                                    |
| ├─leavesQty| string | 剩余数量                                    |
| ├─ordType | string | 委托类型 MARKET,LIMIT,POST_ONLY             |
| ├─ordQty| string | 订单数量                                    |
| ├─cumAmt| string | 成交金额                                    |
| ├─side | string | BUY,SELL                                |
| ├─cumQty| string | 成交数量                                    |
| ├─ ordId      | long  |     订单id                                |
| ├─ clOrdId      | string     | 客户端订单id                                 |
| ├─symbol | string | 币对                                      |
| ├─timestamp| long | 时间戳                                     |

## <span id="3">获取用户最新成交</span>
获取全部成交记录

### HTTP请求: 
- GET /trade/match/accountMatches


> 响应

```json
{
    "data": [
        {
            "id": 3984987,
            "remainingQty": 0E-18,
            "matchRole": 1,
            "feeCurrencyId": 44,
            "acturalFeeRate": 0.002000000000000000,
            "role": 1,
            "accountId": 1138204,
            "instrumentId": 15,
            "baseCurrencyId": 44,
            "quoteCurrencyId": 30,
            "execQty": 40.900000000000000000,
            "orderState": 50,
            "matchId": 258338866,
            "orderId": 1717384395096065,
            "side": 1,
            "execAmt": 8.887161000000000000,
            "selfDealingQty": 0E-18,
            "tradeId": 11523732,
            "fee": 0.081800000000000000,
            "matchTime": 1637825389,
            "seq": null
        }
    ],
    "code": 0
}
```

### 请求参数

|    code    |  type   | required | comment                |
| ---------- | ------- | -------- |------------------------|
|symbol | string | Y | 交易对                    |
|ordId| int | N | 委托ID                   |
|pageNum| int | N | 当前页码，如果不传参，则默认返回 第 1 页 |
|pageSize| int | N | 每页数量，如果不传参，则默认返回 20 条  |
|side| int | N | 方向 -1卖出 1买入            |

### 响应数据

|       code        |  type  |                      comment                       |
| ----------------- | ------ |---------------------------------- |
| code            | int    |  0：成功，其他失败                                               |
| message         | string    |   错误信息                                    |
| data | object|   |
| ├─accountId| long| 账户id |
| ├─instrumentId| int | 币对id |
| ├─baseCurrencyId| int | 基础货币 BTC |
| ├─quoteCurrencyId| int | 计价货币USDT |
| ├─matchRole| int | 撮合角色 TAKER(1),MAKER(-1) |
| ├─feeCurrencyId| int | 手续费币种id |
| ├─role| String | 角色 TAKER,MAKER |
| ├─execQty| String | 成交数量 |
| ├─orderState| int | 订单状态 |
| ├─matchId| long | 撮合id |
| ├─orderId| long | 订单id |
| ├─side| int | 订单方向 | BUY(1), SELL(-1) |
| ├─execAmt| String | 成交额 |
| ├─selfDealingQty| String | 自成交额 |
| ├─tradeId| long | 成交id |
| ├─fee| String | 手续费 |
| ├─matchTime| long  | 撮合时间 |
| ├─remainingQty| String | 剩余数量 |


##  <span id="4">取消委托单</span>
取消委托单

### HTTP请求: 
- POST /trade/order/cancel

> 请求体

```json
{
  "ordId": 1722183748419690,
  "symbol":"LUFFYUSDT"
}
```

> 响应

```json
{
    "code": 0,
    "data": {
        "clientOrderId": "b1b2ea5e00a84b888d419cb73f8eb203",
        "state": "CANCELED",
        "ordId": 1722183748419690
    }
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| symbol     | string | true | |
| ordId     | long     | true  |  |

### 响应数据

|       code        |  type  |                      comment                       |
| ----------------- | ------ |--------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string    |   错误信息                                    |
| data            |    |   返回数据                    |
| ├─state| string|   |
| ├─ordId| long |   |

##  <span id="5">一键撤单</span>


### HTTP请求: 
- POST /trade/order/cancelAll

> 请求体

```json
{
  "symbol":"LUFFYUSDT"
}
```

> 响应

```json
{
    "code": 0
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| symbol     | string | false| 不传撤销所有币对订单 |

### 响应数据

|       code        |  type  |                      comment                       |
| ----------------- | ------ |--------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string    |    错误信息                                    |


## <span id="6">创建订单</span>
创建订单

#### 普通用户的账号同一时间只允许持有 50 笔活动委托，做市账号不受此限制。如果您是做市商但有账号未添加做市账号权限，请联系 Coinstore 交付部门。

> 请求体

```json
{
	"symbol": "LUFFYUSDT",
	"side": "BUY",
	"ordType": "LIMIT",
	"ordPrice": 9.2e-10,
	"ordQty": "12323231243",
	"timestamp": 1642407805168
}
```

> 响应

```json
{
    "code": "0",
    "data": {
        "order_id":11594964764657880
    }
}
``` 

### HTTP请求: 
- POST /trade/order/place

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| clOrdId | string | false    |   客户端订单ID。客户自定义的唯一订单ID。 如果未发送，则自动生成  |
| symbol        | string | true    |  |
| side   | string    | true    |   BUY,SELL    |
| ordType     | string    | true    |  MARKET,LIMIT,POST_ONLY   |
| timeInForce    | string     | false    | defaultValue = "GTC"   |
| ordPrice    | long     | false    | 限价单必选 |
| ordQty    | long     | false    | 限价单必选，市价卖单必选 |
| ordAmt    | long     | false    | 市价买单必选 |
| timestamp    | long     | true    |时间戳  |                                                                    


                                

### 响应数据

|       code        |  type  |                    comment                       |
| ----------------- | ------ |------------------------------- |
| code            | int     |     0：成功，其他失败                                               |
| message         | string    |    错误信息                                    |
| data            |     |   返回数据                                                 |
| ├─ ordId            | long   |  订单id                                                 |

## <span id="13">批量下单</span>
批量下单

#### 普通用户的账号同一时间只允许持有 50 笔活动委托，做市账号不受此限制。如果您是做市商但有账号未添加做市账号权限，请联系 Coinstore 交付部门。

> 请求体

```json
{
	"symbol": "LUFFYUSDT",
	"orders": [
		{
			"side": "BUY",
			"ordType": "LIMIT",
			"ordPrice": 9.2e-10,
			"ordQty": "1"
		},
		{
			"side": "BUY",
			"ordType": "LIMIT",
			"ordPrice": 9.2e-10,
			"ordQty": "1"
		}
	],
	"timestamp": 1642574848089
}
```

> 响应

```json
{
    "code": 0,
    "data": [
        {
            "ordId": 1722364585836585,
            "clOrdId": "c09e68b49a724c8a95ef76526bf6bc05"
        },
        {
            "ordId": 1722364585836586,
            "clOrdId": "d0344830639d4f6d94be4110bf98c759"
        }
    ]
}
```

### HTTP请求: 
- POST /trade/order/placeBatch

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| symbol        | string | true    |  |
| timestamp    | long     | true    |  |
| orders        | list | true    |  |
| ├─ clOrdId | string | false    |   客户端订单ID。客户自定义的唯一订单ID。 如果未发送，则自动生成  |
| ├─ side   | string    | true    |   BUY,SELL    |
| ├─ ordType     | string    | true    |  MARKET,LIMIT,POST_ONLY   |
| ├─ timeInForce    | string     | false    | defaultValue = "GTC"   |
| ├─ ordPrice    | long     | false    | 限价单必选 |
| ├─ ordQty    | long     | false    | 限价单必选，市价卖单必选 |
| ├─ ordAmt    | long     | false    | 市价买单必选 |


### 响应数据

|       code        |  type  |                     comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |    0：成功，其他失败                                               |
| message         | string        |    错误信息                                    |
| data            |  list    |   返回数据                                                 |
| ├─ ordId            | long     |   订单id            |
| ├─ clOrdId            | string     |   客户端订单id            |
| ├─errno| int|   |  |
| ├─errMsg| string| | |

## <span id="27">根据订单id批量进行撤单</span>
根据订单id批量进行撤单

### HTTP请求: 
- POST trade/order/cancelBatch

> 请求体

```json
{
  "orderIds": [1722364585836586,1722364585836585],
  "symbol":"LUFFYUSDT"
}
```

> 响应

```json
{
    "data": {
        "success": [
            1722364585836586,
            1722364585836585
        ],
        "reject": []
    },
    "code": 0
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | Y |  "BTCUSDT"|
|orderIds| LIst| Y |[1,2,3]  |



### 响应数据

|       code        |  type  |          comment                       |
| ----------------- | ------ |---------------------------------------- |
| code         | string    |   0：成功，其他失败                               |
| message         | string    |    错误信息                                   |
| data            | Object   |                                      |
| --success|   List   |     成功的订单id集合                                         |
| --reject  |    List     |  失败的订单id集合                      |


## <span id="14">获取订单信息</span>
获取订单信息

> 响应

```json
{
    "data": {
        "baseCurrency": "LUFFY",
        "quoteCurrency": "USDT",
        "symbol": "LUFFYUSDT",
        "timestamp": 1642574848089,
        "side": "BUY",
        "accountId": 1138204,
        "ordId": 1722364585836586,
        "clOrdId": "d0344830639d4f6d94be4110bf98c759",
        "ordType": "LIMIT",
        "ordState": "CANCELED",
        "ordPrice": "0.00000000092",
        "ordQty": "1",
        "ordAmt": "0.00000000092",
        "cumAmt": "0",
        "cumQty": "0",
        "leavesQty": "0",
        "avgPrice": "0",
        "feeCurrency": "LUFFY",
        "timeInForce": "GTC"
    },
    "code": 0
}
```

### HTTP请求: 
- GET trade/order/orderInfo

### 请求参数:

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|ordId | long | true  | 订单id |

### 响应数据:

|       code        |  type  |               comment                       |
| ----------------- | ------ | -------------------------------------- |
| code            | int     |     0：成功，其他失败                                               |
| message         | string    |    错误信息                                    |
| data | object|  | 
| ├─ordId| long | 订单ID |
| ├─clOrdId| String | 客户端订单ID |
| ├─accountId| long | 账户ID |
| ├─symbol | String |   | 交易对名称 |
| ├─baseCurrency| String | 基础货币 |
| ├─quoteCurrency| String | 计价货币 |
| ├─ordPrice | String | 价格 |
| ├─ordQty| String | 数量 |
| ├─ordAmt| String | 金额 |
| ├─side | Stirng | 方向 BUY,SELL|
| ├─cumAmt| String | 累计成交额 |
| ├─cumQty| String | 累计成交量 |
| ├─leavesQty| String | 剩余数量 |
| ├─ordState | Stirng | 状态 NOT_FOUND, REJECTED, SUBMITTING, SUBMITTED, PARTIAL_FILLED, REPLACING, REPLACED, CANCELING, CANCELED, EXPIRED, STOPPED, FILLED |
| ├─ordType | Stirng | 订单类型 MARKET,LIMIT,POST_ONLY  |
| ├─flags| Stirng | 订单类型 POST_ONLY,REDUCE_ONLY,HIDDEN |
| ├─timeInForce| Stirng | 成交限制类型 GTC,IOC,FOK |
| ├─timestamp| long | 订单时间 毫秒 |
| ├─avgPrice| long | 成交均价  |



## <span id="15">获取订单信息V2</span>
获取订单信息V2

#### 新接口的 API域名地址 `https://api.coinstore.com`  调用支持ApiKey与Token


### HTTP请求:
- GET /api/v2/trade/order/orderInfo

> 响应

```json
{
    "data": [
        {
          "baseCurrency": "A123",
          "quoteCurrency": "USDT",
          "side": "BUY",
          "cumQty": "0",
          "ordId": 1758065956225025,
          "clOrdId": "tLCGVA4g19zuEBwsITXi9g3U624Al0Bw",
          "ordType": "LIMIT",
          "ordQty": "10",
          "cumAmt": "0",
          "accountId": 1134912,
          "timeInForce": "GTC",
          "ordPrice": "100",
          "leavesQty": "10",
          "avgPrice": "0",
          "ordStatus": "SUBMITTED",
          "symbol": "A123USDT",
          "orderUpdateTime": 1676622290389,
          "timestamp": 1676622290389
        }
    ],
    "code": 0
}
```

### 请求参数

|    code    | type   | required | comment |
| ---------- |--------|--------|---------|
|ordId| int    | N      | 委托ID    |
|clOrdId| String | N      | 客户端委托ID  |


### 响应数据

|       code    |  type  | comment                                 |
| ------------- | ------ |-----------------------------------------|
| code          | int    | 0：成功，其他失败                               |
| message       | string    | 错误信息                                    |
| data          |  list  |                                         |
| ├─baseCurrency| string | base币                                   |
| ├─quoteCurrency| string | quot币                                   |
| ├─timeInForce| string | GTC,IOC,FOK                             |
| ├─ordStatus | string | 订单状态 NOT_FOUND,SUBMITTING,SUBMITTED,PARTIAL_FILLED,CANCELED,FILLED |
| ├─avgPrice | string | 成交均价	                                   |
| ├─ordPrice | string | 订单价格                                    |
| ├─leavesQty| string | 剩余数量                                    |
| ├─ordType | string | 委托类型 MARKET,LIMIT,POST_ONLY             |
| ├─ordQty| string | 订单数量                                    |
| ├─cumAmt| string | 成交金额                                    |
| ├─side | string | BUY,SELL                                |
| ├─cumQty| string | 成交数量                                    |
| ├─ ordId      | long  |     订单id                                |
| ├─ clOrdId      | string     | 客户端订单id                                 |
| ├─symbol | string | 币对                                      |
| ├─orderUpdateTime| long | 订单状态更新时间：若未成交，则系统返回“订单创建时间”；若已成交，则系统返回最近一笔成交的时间             |
| ├─timestamp| long | 时间戳                                     |


# 行情相关

##  <span id="7">Ticker</span>
 市场所有交易对的Ticker
 
### HTTP请求: 
- GET /v1/market/tickers

> 响应

```json
{
  "data": [
    {
      "channel": "ticker",
      "bidSize": "454.2",
      "askSize": "542.6",
      "count": 41723,
      "volume": "24121351.85",
      "amount": "1693799.10798965",
      "close": "0.067998",
      "open": "0.071842",
      "high": "0.072453",
      "low": "0.067985",
      "bid": "0.0679",
      "ask": "0.0681",
      "symbol": "TRXUSDT",
      "instrumentId": 2
    }
  ],
  "code": 0
}
```

### 请求参数: 

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |

### 响应数据: 

|       code        |  type   |                      comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string       |    错误信息                                    |
| data | Object|   |
| ├─channel| String |标识  |
| ├─instrumentId| Integer  |交易对ID |
| ├─symbol| String  |交易对 |
| ├─count | Integer  |24h滚动交易笔数 |
| ├─volume|String  |24h以报价币种计量的交易量 |    
| ├─amount|  String  |24h以基础币种计量的交易量 |           
| ├─close|  String  | 末一笔成交价| 
| ├─open|  String  | 第一笔成交价| 
| ├─high|  String  | 最高一笔成交价| 
| ├─low|  String  | 最低一笔成交价| 
| ├─bid|  String  | 买一价| 
| ├─bidSize|  String  | 买一量| 
| ├─ask|  String  | 卖一价| 
| ├─askSize|  String  | 卖一量| 
                     



 
##  <span id="7">获取深度</span>
获取深度数据

> 响应

```json
{
	"data": {
		"channel": "4@depth@5",
		"level": 5,
		"a": [
			[
				"41995",
				"0.018144",
				-1
			],
			[
				"41995.5",
				"0.008279",
				-1
			]
		],
		"b": [
			[
				"41991.5",
				"0.548567",
				1
			],
			[
				"41991",
				"0.013168",
				1
			]
		],
		"symbol": "BTCUSDT",
		"instrumentId": 4
	},
	"code": 0
}
 ```
 
### HTTP请求: 
- GET /v1/market/depth/{symbol}

### 请求参数: 

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | true | 交易对，如“BTCUSDT” |
|depth | int | false | 深度的数量，如“5, 10, 20, 50, 100”,默认20 |

### 响应数据: 

|       code        |  type   |                      comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string       |    错误信息                                    |
| data | Object|   |
| ├─channel| String |标识  |
| ├─level| Integer  |数量 |
| ├─instrumentId| Integer  |交易对ID |
| ├─symbol| string  |交易对 |
| ├─a | List  |卖盘，[${价格}, ${数量}, ${方向}] ] |
| ├─b | List  |买盘，[${价格}, ${数量}, ${方向}] ] |


##  <span id="7">获取K线</span>
获取交易K线

> 响应

```json
{
	"data": {
		"channel": "4@kline@week_1",
		"item": [
			{
				"endTime": 1641744059,
				"amount": "98630624.103511453",
				"interval": "week_1",
				"startTime": 1641744000,
				"firstTradeId": 14547884,
				"lastTradeId": 14799534,
				"volume": "2307.742209",
				"close": "43196.0002",
				"open": "41628.2582",
				"high": "52250.3884",
				"low": "39654.039"
			}
		],
		"symbol": "BTCUSDT",
		"instrumentId": 4
	},
	"code": 0
}
 ```
 
### HTTP请求: 
- GET /v1/market/kline/{symbol}

### 请求参数: 

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | ------------------ |
|symbol | string | true | 交易对，如“BTCUSDT” |
|period | String | false | 数据粒度，如“1min, 5min, 15min, 30min, 60min, 4hour,12hour, 1day, 1week”,默认20 |
|size | Integer | false | 数据条数，[1,2000] |

### 响应数据: 

|       code        |  type   |                      comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string       |    错误信息                                    |
| data | Object|   |
| ├─channel| String |标识  |
| ├─instrumentId| Integer  |交易对ID |
| ├─symbol| String  |交易对 |
| ├─item | List  |K 线集合 |
| ├─├─interval | String  |K线间隔 |
| ├─├─endTime| Long |结束时间，单位 s |         
| ├─├─amount|String  |成交额 |               
| ├─├─startTime|Long  |开始时间，单位 s|     
| ├─├─firstTradeId|Long |第一笔成交ID |    
| ├─├─lastTradeId|Long  |末一笔成交ID |    
| ├─├─volume|  String  |成交量 |           
| ├─├─close|  String  | 末一笔成交价|            
| ├─├─open|   String  | 第一笔成交价|            
| ├─├─high|  String  |最高成交价 |             
| ├─├─low|   String  |最低成交价 |             



##  <span id="7">最新成交</span>
获取最新成交记录

### HTTP请求: 
- GET /v1/market/trade/{symbol}

> 响应

```json
 {
 	"data": [
 		{
 			"channel": "4@trade",
 			"time": 1642495112,
 			"volume": "0.011102",
 			"price": "41732.3305",
 			"tradeId": 14867136,
 			"takerSide": "BUY",
 			"seq": 14867136,
 			"ts": 1642495112000,
 			"symbol": "BTCUSDT",
 			"instrumentId": 4
 		}
 	],
 	"code": 0
 }
```

### 请求参数: 

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | true | 交易对，如“BTCUSDT” |
|size | Integer | false | 数据条数，[1,100] |

### 响应数据: 

|       code        |  type   |                      comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string       |    错误信息                                    |
| data | Object|   |
| ├─channel| String |标识  |
| ├─instrumentId| Integer  |交易对ID |
| ├─symbol| String  |交易对 |
| ├─interval | String  |K线间隔 |
| ├─time| Long |时间，单位 s |         
| ├─ts|Long  |时间，单位 ms |                  
| ├─tradeId|Long |第一笔成交ID |    
| ├─takerSide|String  |方向 "BUY"，"SELL" |    
| ├─volume|  String  |成交量 |           
| ├─price|  String  | 末一笔成交价|                       



##  <span id="7">获取所有交易对最新价格</span>
获取所有交易对最新价格

> 响应

```json
{
  "code": 0,
  "message": "",
  "data": [
    {
      "id": 1,
      "symbol": "BTCUSDT",
      "price": "400"
    },
    {
      "id": 4,
      "symbol": "AUTUSDT",
      "price": "1"
    }
  ]
}
 ```
 
### HTTP请求: 
- GET /v1/ticker/price

### 请求参数: 

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | true | 交易对，区分大小写，逗号分割 |

**默认：查询所有交易对**
**查询指定交易对: /v1/ticker/price;symbol=BTCUSDT,EOSUSDT,AUTUSDT**

### 响应数据: 

|       code        |  type   |                      comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string       |    错误信息                                    |
| data | List[]|   |
| ├─id| long |  |
| ├─symbol| string  |交易对 |
| ├─price| string  |价格 |

  
  
# Websocket行情数据

## **简介**

### 接入URL
wss://ws.coinstore.com/s/ws

1. 所有wss接口的 baseurl为: wss://<host:port>/s/ws

2. stream名称中所有交易对均为 小写

3. 每个链接有效期不超过24小时，请妥善处理断线重连。

4. 每3分钟，服务端会发送ping帧，客户端应当在10分钟内回复pong帧，否则服务端会主动断开链接。允许客户端发送不成对的pong帧(即客户端可以以高于10分钟每次的频率发送pong帧保持链接)。


### 服务端推送数据类型说明

> Format Schema

```lang=json
{
    "S": 1, // session 级别的 response 消息序号，session 重连后重置，可用来判断是否漏消息
    "T": "echo|resp", // 响应类型，详见 `Types` 部分
    "C": 200,  // code
    "M": "sub.channel.success",  // meaning of this code
    "sid": "3ae985ff-cadd-9c96-7a42-4ed29e77f83b",
    "echo": {},  // client request command
    "payload": "external msg"
}
```

#### Types

##### request-response 类型的消息

* `echo`: 服务器接受到到每一个消息都会以该种形式返回一个message，确认消息已经被接收
* `resp`: 服务器接受到到每一个消息都会以该种形式返回一个处理结果的message，标记消息处理，返回信息中包含处理结果信息


##### subscribe 消息类型
以下类型的消息，从执行对应 channel 的 `SUB` 命令开始，到执行 `UNSUB` 结束，如果服务器端数据发生变化则推送，最小推送间隔 100ms
```lang=json
{
    "S": 1,
    "T": "kline|ticker|depth|trade|account",
    ...:  // 详细见下面到每种数据订阅格式 
}
```
* `kline`

* `ticker`

* `depth`

* `trade`

* `account`

* `order`



### Pong
服务器端支持 websocket pong frame 和 pong message 两种形式端 pong 响应

> Pong message format

```lang=json
{
  "op": "pong",
  "epochMillis": 1604040975
}
```

#### Websocket Pong frame

1. https://tools.ietf.org/html/rfc6455#section-5.5.3 

2. https://developer.mozilla.org/zh-CN/docs/Web/API/WebSockets_API/Writing_WebSocket_servers#Pings%E5%92%8CPongs%EF%BC%9AWebSockets%E7%9A%84%E5%BF%83%E8%B7%B3 




> Example

```lang=shell
$>wscat -c 'ws://127.0.0.1:8080/s/ws'
< {"S":1,"T":"resp","sid":"3ae985ff-cadd-9c96-7a42-4ed29e77f83b","C":200,"M":"established"}
> {"op":"SUB","channel":["80004@trade"],"id":1}
< {"S":2,"T":"echo","sid":"3ae985ff-cadd-9c96-7a42-4ed29e77f83b","C":200,"M":"command.received","echo":{"op":"SUB","channel":[{"name":"80004@trade","type":{"name":"trade","auth":"PUB"},"instrumentId":"80004","lifeStartTime":1604040964,"msgCount":0,"pub":true}],"id":1}}
< {"S":3,"T":"resp","sid":"3ae985ff-cadd-9c96-7a42-4ed29e77f83b","C":200,"M":"sub.channel.success"}
< {"S":4,"T":"trade","channel":"80004@trade","time":1604040975,"price":"9811.7494086","takerSide":"SELL","tradeId":26461,"volume":"7.505","symbol":"EOSUSD","instrumentId":80004}
```

## **实时订阅/取消数据流**

> 订阅一个信息流

```lang=json
{
    "op": "SUB",
    "channel": [
        "btcusdt@ticker",
        "btcusdt@depth"
    ],
    "id": 1
}
```

> 取消订阅一个信息流

```lang=json
{
    "op": "UNSUB",
    "channel": [
        "btcusdt@depth"
    ],
    "id": 312
}
```

> 已订阅信息流

```lang=json
{
    "op": "LIST",
    "channel":[],
    "id": 3
}
```

* 以下数据可以通过websocket发送以实现订阅或取消订阅数据流。示例如下。
* 响应内容中的id是无符号整数，作为往来信息的唯一标识。
* 如果相应内容中的 result 为 null，表示请求发送成功。

### Client `op` Types

1. `SUB`
2. `UNSUB`
3. `LOGIN`
4. `REQ`
5. `LIST`



> Respon

```lang=json
{
    "S": 12,
    "T": "resp",
    "subs": [
        {
            "name": "80004@trade",
            "type": {
                "name": "trade",
                "auth": "PUB"
            },
            "instrumentId": "80004",
            "lifeStartTime": 1604043594,
            "msgCount": 212
        }
    ],
    "sid": "b1beeb73-7c84-862f-3d03-619de1e1a5dc"
}
```

> NOTE: `<symbol>` 参数暂时请使用 交易对的id，后续支持使用交易对的名称

> IMPORTANT:  请优先使用  `symbol` 字段，` instrumentId` 标记为 `Deprecated`

> NOTE:  所有返回数据的时间，单位都是 `秒`


## **逐笔交易**

> Send

```lang=json
 {
 	"op": "SUB",
 	"channel": [
 		"28@trade"
 	],
 	"param": {
 		"size": 2
 	},
 	"id": 1
 }
```
> Receive全量数据：

```lang=json
 {
   "S": 15,
   "T": "trade",
   "data": [
     {
       "channel": "28@trade",
       "price": "384.32",
       "tradeId": 21704,
       "seq": 10157,
       "ts": 1612685697087,
       "takerSide": "BULL",
       "time": 1612685697,
       "volume": "2",
       "symbol": "BTCJJ",
       "instrumentId": 28
     },
     {
       "channel": "28@trade",
       "price": "644.96",
       "tradeId": 21705,
       "seq": 10158,
       "ts": 1612685698157,
       "takerSide": "SELL",
       "time": 1612685698,
       "volume": "1.1932988",
       "symbol": "BTCJJ",
       "instrumentId": 28
     }
   ]
 }
```
> Receive增量数据：

```lang=json
{
  "instrumentId": 88066,
  "symbol": "USDTBTC",    // 交易对
  "tradeId": 12345,       // 交易ID
  "takerSide": "BUY",    // taker side
  "price": "0.001",     // 成交价格
  "volume": "100",       // 成交数量
  "time": 123456785,   // 成交时间 单位 s
  "ts": 1612685313400, // 单位 ms
  "seq": 9935         // 唯一自增序号
}
```

### Stream Name
`<symbol>@trade`

eg: `88066@trade`
 
### param
 `param":{"size":2}`
 




## **K线 Streams**
K线stream逐秒推送所请求的K线种类(最新一根K线)的更新。

> Send

```lang=json
 {"op":"SUB","channel":["4@kline@min_1"],"id":1}
```
> Receive

```lang=json
{
   "instrumentId": 88066, 
   "startTime": 1603732500,  // 这根 k线的开始时间，单位 s
    "endTime": 1603732559,  // 这根 k线的开始时间，单位 s
    "symbol": "USDTBTC",  // 交易对
    "interval": "1m",      // K线间隔
    "firstTradeId": 100,       // 这根K线期间第一笔成交ID
    "lastTradeId": 200,       // 这根K线期间末一笔成交ID
    "open": "0.0010",  // 这根K线期间第一笔成交价
    "close": "0.0020",  // 这根K线期间末一笔成交价
    "high": "0.0025",  // 这根K线期间最高成交价
    "low": "0.0015",  // 这根K线期间最低成交价
    "volume": "1000",    // 这根K线期间成交量
    "amount": "1.0000",  // 这根K线期间成交额
}
```

### Stream Name
`<symbol>@kline@<interval>`
  
eg: `88066@kline@min_1`

### interval 可选值
* min_1
* min_5
* min_15
* min_30
* hour_1
* hour_4
* hour_12
* day_1
* week_1



## **K线 Request**
请求历史k线

> Send

```lang=json
 {
 	"op": "REQ",
 	"param": {
 		"channel": "4@kline@min_1",
 		"endTime": null,
 		"limit": 200
 	},
 	"id": 1
 }
```
> Receive

```lang=json
{
    "channel": "88066@kline@min_1",
    "symbol": "USDTBTC",
    "instrumentId": 88066,
    "item": [
        {
            "interval": "min_1",
            "startTime": 1603766280,
            "endTime": 1603766339,
            "lastTradeId": 4739656,
            "firstTradeId": 4739656,
            "close": "4.91",
            "open": "4.91",
            "high": "4.91",
            "low": "4.91",
            "volume": "8.488",
            "amount": "41.67608"
        },
        ...
    ]
}
```

###  param

* `channel`: `<symbol>@kline@<interval>` ,eg:88066@kline@min_1
* `limit`: 最大不超过 200
* `endTime`: 可选，exclusive





## **按 Symbol 的 Ticker信息**
按Symbol刷新的最近24小时精简 ticker 信息

> Send

```lang=json
 {"op":"SUB","channel":["4@ticker"],"id":1}
```
> Receive

```lang=json
{
  "instrumentId": 88066,
  "symbol": "USDTBTC",      // 交易对
  "open": "0.0010",      // 整整24小时前，向后数的第一次成交价格
  "high": "0.0025",      // 24小时内最高成交价
  "low": "0.0010",      // 24小时内最低成交加
  "close": "0.0025",      // 最新成交价格
  "volume": "10000",       // 24小时内成交量
  "amount": "18"          // 24小时内成交额
}
```

### Stream 名称
`<symbol>@ticker`, eg:  `88066@ticker`



## **深度信息**
每秒或每100毫秒推送有限档深度信息。

> Send

```lang=json
 {"op":"SUB","channel":["4@depth@20"],"id":1}
```

> Receive

```lang=json
{
  "level": 5,
  "instrumentId": 88066,
  "symbol": "BTCUSDT"
  "b": [             // Bids to be updated
    [
      "0.0024",         // Price level to be updated
      "10"              // Quantity
    ]
  ],
  "a": [             // Asks to be updated
    [
      "0.0026",         // Price level to be updated
      "100"            // Quantity
    ]
  ]
}
```

### Stream Names
`<symbol>@depth@<levels>` 

 eg:  `88066@depth@50`
 
### levels 可选值
levels表示几档买卖单信息, 可选 5/10/20/50/100档




## **登陆**
登陆时可以同时指定订阅数据

```lang=json
{
    "op": "LOGIN",
    "channel": [
        "7@account",
        "80002@order"
    ],
    "auth": {
        "user-id":4930,     // 用户登陆账号的 user id 或者子账号的 user id
        "token": "eb737c84862f3d"  // 登陆之后的 token； 端上登陆用户使用登陆后的token， api 用户填写  api-key
        "type": "token"      // 可选值范围：token | api , 默认为 token， api 用户填写 api
    }
    "id": 2
}
```

## **账户**

 Stream Name: `<currency>@account`, or `!@account` all currency

```lang=json
{
    "accountId": 12,
    "currency": "BTC",
    "frozen": "21.03",
    "available": "3128.29",
    "timestamp": 1602493840  // 单位 s
}

```

## **成交订单**

 Stream Name: `<symbol>@order`, or `!@order` all symbol's

```lang=json
{
    // increment
    
    "version": 12,

    // order base info, never change

    "accountId": 1,
    "ordId": 12,
    "symbol": "BTCUSDT",
    "side": "BUY",
    "ordType": "LIMIT",
    "timeInForce": "GTC",
    "ordPrice": "128.21",
    "ordQty": "11.21",
    "ordAmt": "231.2",

    // order state
    "ordState": "PARTIAL_FILLED",

    // sum every trade time

    "execQty": "12.2",
    "execAmt": "312.12",
    "remainingQty": "1.2",

    // current trade info,  present meaning values when `matchQty` is bigger than 0

    "matchId": 12,
    "tradeId": 123,
    "matchRole": "TAKER",
    "matchQty": "1.9",
    "matchAmt": "390.29",
    "selfDealingQty": "1.9",
    "actualFeeRate": "0.002",
    "feeCurrencyId": 12,
    "fee": 0.21,

    "timestamp": 1602493840 // 单位 s
}
```

## **字典**

### OrderState

* REJECTED
* SUBMITTING
* SUBMITTED
* PARTIAL_FILLED
* CANCELING
* CANCELED
* EXPIRED
* STOPPED
* FILLED

### OrderType

* MARKET 
* LIMIT 
* POST_ONLY

### TimeInForce

* IOC
* GTC

### MatchRole

* TAKER
* MAKER

### Side

* BUY
* SELL


