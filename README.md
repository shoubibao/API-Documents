# 收币宝接口文档

所有请求接口数据格式均为application/json形式。

请求的参数格式如下：

```json
{
    "timestamp": "",
    "nonce": "",
    "body": "",
    "sign": ""
}
```

其中timestamp是请求时的时间信息，UNIX TIME格式，nonce是6位数的随机数，body是业务请求内容的字符串化，sign是请求签名

```js
sign=sha256(body+ timestamp + nonce + key )
```


响应的参数格式如下：

```json
{
    "success": true,
    "msg": "",
    "data": {}
}
```

其中，success表示请求成功(true)或者失败(false)，msg表示请求返回消息，data是请求返回数据对象。

## 1. 创建地址

请求创建新的钱包地址，可以同时创建同一条链的支出不同币种的的地址。

接口： /v1/api/wallet/new, POST


请求数据：

```json
{
   "app": "app123",
   "chain": "ETH",
   "coins": ["USDT"]
}
```

其中，app是应用ID, chain是链的标识，coin是对应需要的创建的地址币种。

返回数据：

```json
{
    "address": ""
}
```

返回数据为address表示创建的地址名称。

## 2. 提币

接口： /v1/api/withdraw, POST

用于从一个应用中转出相应的币种到某个钱包。

请求数据：

```json
{
   "app": "app123",
   "chain": "ETH",
   "coin": "USDT",
   "address": "",
   "amount": "0.10",
   "orderId": "",
   "memo": ""
}
```
其中，app是应用id, chain是链的标识，以链的主币种标识，coin提币的币种，address的提币目标地址，amount是提币金额，金额的小数点按币种的小数点设计，orderId是唯一不重复的提币订单号，memo是提币交易里面的标识。


返回数据：

提币返回数据data为空，根据 success判断是否提币请求提交成功。


## 3. 地址有效性

判断用户提交的地址是否是有效的地址。

接口： /v1/api/address/valid, POST

请求数据：

```json
{
   "app": "app123",
   "chain": "ETH",
   "address": ""
}
```

返回数据：

```json
{
    "valid": true
}
```


## 4. 应用信息

返回应用支持的币种信息，以及币种的参数。

接口： /v1/api/app/info, POST

请求数据：

```json
{
   "app": "app123"
}
```

其中app为应用ID。

返回数据：

```json
{
	"app": "app123",
	"chain": [{
		"name": "ETH",
		"confirm": 12,
		"coins": [{
			"name": "USDT",
			"deposit": true,
			"withdraw": true,
			"depositMin": "1.0",
			"withdrawMin": "10"
		}, {
			"name": "HT",
			"deposit": true,
			"withdraw": true,
			"depositMin": "10.0",
			"withdrawMin": "100"
		}]
	}]
}
```

返回数据中，chain是应用支持的币种信息，name为链的主货币标识，confirm链需要的确认次数，coins是链支持的币种信息。每一个币种信息包括name币种名称，deposit和withdraw标识是否支持充币或提币，depositMin和withrawMint标识支持的最小充币和最小提币数量。


## 5. 回调通知

回调通知通过POST方式通知到应用设置的回调地址中，请求body参数包括：

```json
{
    "hash": "",
    "from": "",
    "to": "",
    "chain": "",
    "coin": "",
    "amount": "",
    "confirm": "",
    "incoming": "",
    "orderId": "",
    "block": ""
}
```

其中，hash标识通知的交易ID，from标识交易发起地址，to标识交易目标地址，chain标识交易所在的链，coin标识交易的货币名称，amount是交易金额，confirm是确认次数，incoming标识是充币(true)或者是提币(false)，orderId是提币交易回调时才有的提币订单号，block是当前交易所在的区块号。


回调通知会依据链的confirm次数依次通知相应的次数，如果在confirm此种均没有返回相应的结果，收币宝将最后一个confirm通知，依次在5秒、15秒、30秒、1分钟、3分钟、5分钟的时候再各自通知一次。


