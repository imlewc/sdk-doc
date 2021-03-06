# 中汇移动支付，用户接口规范
> **Beta:**
> 该API仍处于 Beta 阶段

```
1. 该接口适用对象：服务商的服务器
2. 该接口实现功能：用户的创建与管理，商户的创建与管理，设备的绑定与管理
3. 该接口调用规范：采用REST规范的HTTPS请求与中汇的服务器进行通信
4. 该接口调用权限：暂未对资源独立控制
```
> **注：**
> 文中所有 `<>` 标注的字段，均需根据你的实际情况替换（无需 `<>` 符号，仅作标注之用）
> 文中所有 `:id` 标注的字段，均需根据该资源的实际 `id` 值替换
> 文中所有 `{x|y|...}` 标注的字段，均需根据你的实际情况用其中一个 `x` 或者 `y`（ `|` 分割）替换

## API 接口地址
```
https://api.vcpos.cn # 生产环境
http://zftapi.21er.net:15080 # 测试环境
```

## 标准请求
```sh
curl -X {GET|POST|PUT|DELETE} \
    http://zftapi.21er.net:15080/<资源路径> \
    -H "Authorization: SIGN <appid>:<signature>" \
    -H "Date: Wed, 8 Apr 2015 15:51 GMT"
    # 其他可选参数...
```

## 标准响应
* 签名校验通过的情况下  
```
HTTP/1.1 200 OK
Server: Nginx
Date: Thu, 09 Apr 2015 11:36:53 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Cache-Control: no-cache
Content-Length: 100
x-zftapi-request-id: <uuid>

{
  "code": "00",
  "msg": "成功",
  "data": {}
}

...body...
```
> **注：**
> `x-zftapi-request-id` 是由API服务创建，并唯一标识这个response的UUID。如果在使用API服务时遇到问题，可以凭借该字段联系技术人员，快速定位问题。 

* 签名校验未通过的情况下  
```
HTTP/1.1 401 Unauthorized
```
* 请求 Method 不被支持的情况下  
```
HTTP/1.1 405 Method Not Allowed
```
* 请求参数不正确的情况下  
```
HTTP/1.1 403 Forbidden
```

## 时间校验
调用所有接口需要进行Date时间校验，通过HTTP Request Header 中添加 `Date` 的方式进行校验：
```
Date: Thu, 09 Apr 2015 11:36:53 GMT
```
*`Date`* 的检验主要验证服务器时间与调用者的时间是否基本一致，时间差需要控制在前后3分钟内。例如，它与订单创建者设定的订单有效期相关，这个有效期可以使用相对时间，也可以使用服务器绝对时间

## 授权认证
调用所有接口需要进行授权认证，通过在 HTTP Request Header 中添加 `Authorization` 的方式来进行权限验证：  
```
Authorization: <method> <appid>:<signature>
```
其中 *`method`* 表示的是签名算法，*`appid`* 为服务商获取的 **appid** 身份，*`signature`* 为使用服务商获取的 **appkey**，根据参数计算出来的签名。
> **注：**
> **`appkey`** 作为服务商或者API使用者使用接口的特定凭证，是不能对外公开或在客户端存储使用的，是一个私有数字凭证

### 签名算法
#### 1\. SIGN签名
SIGN签名算法，*`method`* 取值为 `SIGN`，适用任何类型的接口  
授权认证过程中的 *`signature`* 参数通过下面的方式获得：
> 1. 将 *`appid`* 作为 `<id>`  
> 2. 将HTTP请求行中的数据当成头部数据 `<head>`，例如 `GET /order HTTP/1.1`  
> 3. 如果该请求含有 body，将整个 body 字节组作为 `<body>`，否则为空  
> 4. 将 *`appkey`* 作为 `<key>`  
> 5. 连接整个数据 `<id><head><body><key>` 并对其进行MD5签名，得到 *`signature`*

特别地，未指明编码时，必须使用 Encoding `utf-8` 编码处理

#### 2\. TOKEN签名
TOKEN签名算法，*`method`* 取值为 `TOKEN`，适用于除了获取token接口的所有接口，该方法暂未实现  
授权认证过程中的 *`signature`* 为服务器回传的 `x-zftapi-request-token`

#### 3\. FORM签名
FORM签名算法，*`method`* 取值为 `FORM`，适用 form 类型的接口，该方法暂未实现  
授权认证过程中的 *`signature`* 参数通过下面的方式获得：
> 1. 将 *`appid`* 作为 `<id>`  
> 2. 将HTTP请求行中的数据当成头部数据 `<head>`，例如 `GET /order HTTP/1.1`  
> 3. 如果该请求含有 body，将整个 body 字节组按照以下规则组合为 `<body>`，否则为空  
> > a) 请求体为 form-data 格式，将每一对 key value 连接在一起，每行一对  
> > b) 每行数据按照二进制做8字节异或，不足8字节右补0x00，每行都可以得到一个8字节的异或结果  
> > c) 将所有行做8字节异或，最终的8字节作为 `<body>`  
> 4. 将 *`appkey`* 作为 `<key>`  
> 5. 连接整个数据 `<id><head><body><key>` 并对其进行MD5签名，得到 *`signature`*

特别地，未指明编码时，必须使用 Encoding `utf-8` 编码处理

## <a name="content-title"></a>RESTful资源路径
| 资源名称     | 路径                                     | 可使用的方法            | 重要记录值   |
|--------------|------------------------------------------|-------------------------|--------------|
| 注册验证码   | /registercode                            | POST                    |              |
| 用户         | /user                                    | POST                    | id           |
| 单一用户     | /user/:id                                | GET / PUT               |              |
| 用户实名资料 | /user/:id/realname                       | GET / PUT               |              |
| 商户         | [/merchant](#merchant)                   | `POST`                  | merchantCode |
| 单一商户     | [/merchant/:idOrCode](#merchant1)        | `GET` / PUT / `DELETE`  |              |
| 商户业务     | [/merchant/:idOrCode/business](#business)| `GET` / `PUT`           | merchantNo   |
| 单一收单业务 | [/acq/:merchantno](#acq)                 | `GET` / `PUT`           | info         |
| 终端设备     | [/acq/:merchantno/device](#device)       | `POST` / `GET`          | id           |
| 单一设备     | [/acq/:merchantno/device/:id](#device1)  | `GET` / `PUT` / DELETE  |              |
| 订单         | [/order](#order)                         | `POST` / GET            | orderNo      |
| 单一订单     | [/order/:orderNo](#order1)               | `GET`                   | status       |
| 交易结果     | [/order/:orderNo/status](#order2)        | `GET`                   | receiptUrl   |
| 交易小票     | [/acq/receipt/:name](#receipt)           | `GET`                   |              |
  
--------------------------------------------------------------------------------------------------
> 注： 方法中 使用 `METHOD` 标注了的表示已经完成了的功能

### <a name="regcode"></a>注册验证码  /registercode
#### 1\. 通过注册手机号发送注册验证码
请求：  
```
POST /registercode HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 30

{
  "cellphone": "13811190292",
  "action": "create",
  "timeout": 180000
}
```
响应：  
```
HTTP/1.1 200 OK
Server: Nginx
Date: Thu, 09 Apr 2015 11:36:53 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Cache-Control: no-cache
Content-Length: 100
x-zftapi-request-id: 81e1d8cf-60b1-426c-bd96-8e39c9f57235

{
  "msg": "验证码发送成功",
  "code": "00"
}

// or

{
  "msg": "验证码请求次数过多",
  "code": "Q4"
}

// or

{
  "msg": "手机号已被注册",
  "code": "P2",
}
```
#### 2\. 通过接收验证码验证手机号，以获取registertoken，用于注册
请求：  
```
POST /registercode HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 30

{
  "cellphone": "13811190292",
  "action": "verify",
  "registercode": "4028"
}
```
响应：  
```
{
  "msg": "验证码验证成功",
  "code": 0,
  "registertoken": "1aef1dacf4424acaf1e3"
}

// or

{
  "msg": "验证码错误",
  "code": 1,
}

// or

{
  "msg": "验证码已过期",
  "code": 2,
}
```
##### [返回目录↑](#content-title)
### <a name="user"></a>用户 /user
#### 1\. 通过registertoken创建用户
请求：  
```
POST /user HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 100

{
  "registertoken": "1aef1dacf4424acaf1e3",
  "cellphone": "13811122334",
  "username": "vcposuser",
  "realname": "小明"
}
```
响应：  
```
{
  "msg": "用户注册成功",
  "code": "00",
  "data": {
    "_id": 3579246,
    "_createtime": 1928739283,
    "avatar": "",
    "username": "vcposuser",
    "cellphone": "13811190292",
    "realname": "小明"
  }
}
```
##### [返回目录↑](#content-title)
### <a name="merchant"></a>商户 /merchant
#### 1\. 创建一个商户
请求：  
```
POST /merchant HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 10

{
  "label": "小刚的马家堡火锅店"
}

```
响应：  
```
{
  "code": "00",
  "msg": "成功",
  "data": {
    "_id": 3579246,
    "_createtime": 1928739283,
    "label": "小刚的马家堡火锅店",
    "merchantcode": "M12130000000001",
    "userid": null
  }
}
```
##### [返回目录↑](#content-title)
### <a name="merchant1"></a>单一商户 /merchant/:idOrCode
> `idOrCode` 指内容为纯 `id` 或者 `code-{merchantCode}`

#### 1\. 获取商户信息
请求：  
```
GET /merchant/3579246 HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT

```
响应：  
```
{
  "code": "00",
  "msg": "成功",
  "data": {
    "_id": 3579246,
    "_createtime": 1928739283,
    "label": "小刚的马家堡火锅店",
    "merchantcode": "M12131",
    "userid": null
  }
}
```
##### [返回目录↑](#content-title)
### <a name="business"></a>商户业务 /merchant/:idOrCode/business
> `idOrCode` 指内容为纯 `id` 或者 `code-{merchantCode}`

#### 1\. 获取商户业务数据
请求：  
```
GET /merchant/3579246/business HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT

```
响应：  
```
{
  "code": "00",
  "msg": "成功",
  "data": {
    "SJSDSDK": {
      "_id": 123414512,
      "_createtime": 1928739283,
      "businessname": "手机收单SDK",
      "businesscode": "SJSDSDK",
      "merchantno": "500100002000120",
      "type": 2,
      "status": 1
    }
  }
}
```
#### 2\. 修改商户业务数据
请求：  
```
PUT /merchant/3579246/business HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 25

{
  "add": {   // 表示新增业务
    "SJSDSDK": {  // 手机收单业务
      "merchantno": "500100002000120", // 当type为0或1时，需提供商户号
      "type": 2, // 0表示标准进件商户，1表示大商户，2表示个人商户
      "status": 1, // 0表示禁用，1表示启用
      "serial": "00001111001010FF", // 当type为2时，需提供激活码
      "mobile": "13811111111" // 当type为2时，需提供手机号
    }
  }
}

```
响应：  
```
{
  "code": "00",
  "msg": "成功",
  "data": {
    "SJSDSDK": {  // 手机收单业务
      "_id": "123123123",
      "_createtime": 1431228824888,
      "businessname": "手机收单SDK",
      "businesscode": "SJSDSDK",
      "merchantno": "500100002000120",
      "type": 2, // 0表示标准进件商户，1表示大商户，2表示个人商户
      "status": 1 // 0表示禁用，1表示启用
    }
  }
}
```
##### [返回目录↑](#content-title)
### <a name="acq"></a>单一收单业务 /acq/:merchantno
> `merchantno` 指代商户的收单业务中的商户编号，和单一商户的商户代号有别

#### 1\. 获取商户收单业务详情
> 注：该商户必须为匹配appuser的代理商发展的商户  
> 当商户为个人商户时，可以获取到4审的状态，涵盖在 `info` 字段中  
> 任何类型商户都可以获取到商户的基本信息，该信息为只读信息，不能修改  
> 基本信息包括：中汇商户名，中汇商户类型，维护负责人，开通状态(个人只看4审，其他看enabled)

请求：  
```
GET /acq/500100002000120?merchantcode=M12130000000001&businesscode=SJSDSDK HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT

```
响应：  
```
{
  "code": "00",
  "msg": "成功",
  "data": {
    "merchantname": "小刚的小店",
    "type": "P",
    "salesman": "李大伟",
    "enabled": 1,
    "info": {
      "realname": {
        "name": null, // 姓名，当status不为0时有值
        "id": null, // 身份证，当status不为0时有值
        "status": 0, // 0表示未提交，1表示等待审核，2表示通过，3表示失败
        "reason": null // 当status为3时有值
      },
      "merchant": {
        "name": "小刚的小店", // 收单商户名，默认为商户label
        "license": null, // 营业执照号，当status不为0时有值
        "address": "营业地址"，当status不为0时有值
        "status": 0, // 0表示未提交，1表示等待审核，2表示通过，3表示失败
        "reason": null // 当status为3时有值
      },
      "account": {
        "cardno": null, // 结算卡号，个人不允许对公卡号
        "bank": null, // 开户支行
        "bankunion": null, // 联行号
        "status": 0, // 0表示未提交，1表示等待审核，2表示通过，3表示失败
        "reason": null // 当status为3时有值
      },
      "signature": {
        "status": 0, // 0表示未提交，1表示等待审核，2表示通过，3表示失败
        "reason": null // 当status为3时有值
      }
    }
  }
}
```
#### 2\. 修改商户收单业务详情
> 商户收单业务的修改，只有在商户为个人状态时，可以进行修改。修改的主体内容为4审资料  
> 包括：  
> `realname` 实名认证资料提交  
> `merchant` 商户认证资料提交  
> `account` 账户认证资料提交  
> `signature` 协议签名资料提交  

> 其中，由于不支持PATCH HTTP请求，所以必须每次都提供某一个审核资料的全部信息  
> 采用PUT HTTP请求，请求的 `Content-Type` 为 `multipart/form-data`，并需要上传图片

请求：  
```
PUT /acq/500100002000120 HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: multipart/form-data; boundary=ABCD
Content-Length: 100

--ABCD
Content-Disposition: form-data; name="merchantcode"

M12130000000001
--ABCD
Content-Disposition: form-data: name="businesscode"

SJSDSDK
--ABCD
Content-Disposition: form-data; name="patch"
Content-Type: application/json; charset=utf-8
Content-Transfer-Encoding: 8BIT
Content-Length: 30

{
  "realname": {
    "name": "小刚",
    "id": "110222199001012314",
    "files": [
      "personal",
      "personalBack"
    ]
  }
}
--ABCD
Content-Disposition: form-data; name="personal"; filename="personal.png"
Content-Type: image/png
Content-Transfer-Encoding: Base64
Content-Length: 20

1234567890123123123123213=
--ABCD
Content-Disposition: form-data; name="personalBack"; filename="personalBack.png"
Content-Type: image/png
Content-Transfer-Encoding: Base64
Content-Length: 20

12345ABCDFEFED23123123213=
--ABCD--
```
响应：  
```
{
  "code": "00",
  "msg": "成功",
  "data": {
    "merchantname": "小刚的小店",
    "type": "P",
    "salesman": "李大伟",
    "enabled": 1,
    "info": {
      "realname": {
        "name": "小刚",
        "id": "110222199001012314",
        "status": 1,
        "reason": null
      },
      "merchant": {
        "name": "小刚的小店", // 收单商户名，默认为商户label
        "license": null, // 营业执照号，当status不为0时有值
        "address": "营业地址"，当status不为0时有值
        "status": 0, // 0表示未提交，1表示等待审核，2表示通过，3表示失败
        "reason": null // 当status为3时有值
      },
      "account": {
        "cardno": null, // 结算卡号，个人不允许对公卡号
        "bank": null, // 开户支行
        "bankunion": null, // 联行号
        "status": 0, // 0表示未提交，1表示等待审核，2表示通过，3表示失败
        "reason": null // 当status为3时有值
      },
      "signature": {
        "status": 0, // 0表示未提交，1表示等待审核，2表示通过，3表示失败
        "reason": null // 当status为3时有值
      }
    }
  }
}
```
##### [返回目录↑](#content-title)
### <a name="device"></a>设备 /acq/:merchantno/device
> `merchantno` 指代一个中汇商户号

#### 1\. 新建一个终端设备并绑定KSN
请求：  
```
POST /acq/500100002000120/device HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 100

{
  "ksn": "600010000001",  // KSN编号
  "isbluetooth": true, // 是否是蓝牙设备，如果不是，就没有bluetooth字段，服务器会验证该字段
  "bluetooth": {
    "btmac": "AA:BB:CC:DD:EE:00",
    "btname": "M35-66775678"
  },
  "model": "landim35",
  "merchantcode": "M12130000000001", // 商户代号，必填，服务器会进行验证
  "businesscode": "SJSDSDK", // 商户的业务名，必填、服务器会进行验证
  "label": "我的联迪刷卡器" // 标签
}

```
响应：  
```
{
  "status": "00",
  "成功",
  "data": {
    "_id": "235234", // 终端id
    "_createtime": 2342421341,
    "terminalno": "89746572", // 中汇终端号
    "ksn": "600010000001", // 绑定的KSN号
    "status": 1, // 0表示未使用，1表示已使用，2表示已作废，只有值为1，才有后续数据
    "isbluetooth": true,
    "bluetooth": {
      "btmac": "AA:BB:CC:DD:EE:00",
      "btname": "M35-66775678"
    },
    "model": "landim35",
    "merchantno": "500100002000120",
    "label": "我的联迪刷卡器" // 标签
  }
}
```
#### 2\. 查询单个手机收单商户的所有设备，包括绑定终端和未绑定的终端
请求：  
```
GET /acq/500100002000120/device?merchantcode=M12130000000001&businesscode=SJSDSDK&pagesize=20&pageno=1 HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT

```
响应：  
```
{
  "status": "00",
  "成功",
  "data": {
    "pagesize": 20,
    "pageno": 1,
    "total": 1,
    "devices": [
      {
        "_id": "235234",
        "terminalno": "89746572", // 查询的设备都含有终端号
        "ksn": "600010000001" // 查询的设备不一定含有ksn，此时ksn为null，这表示该商户还有终端号未参与绑定
      }
    ]
  }
}
```
##### [返回目录↑](#content-title)
### <a name="device1"></a>单一设备 /acq/:merchantNo/device/:id
> `merchantNo` 指代一个商户号，`id` 指代一个设备终端id

#### 1\. 查询单一设备
请求：  
```
GET /acq/500100002000120/device/235234?merchantcode=M12130000000001&businesscode=SJSDSDK HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT

```
响应：  
```
{
  "status": "00",
  "成功",
  "data": {
    "_id": "235234", // 终端id
    "_createtime": 2342421341,
    "terminalno": "89746572", // 中汇终端号
    "ksn": "600010000001",
    "status": 1, // 0表示未使用，1表示已使用，2表示已作废，只有值为1，才有后续数据
    "isbluetooth": true,
    "bluetooth": {
      "btmac": "AA:BB:CC:DD:EE:00",
      "btname": "M35-66775678"
    },
    "model": "landim35",
    "merchantno": "500100002000120",
    "label": "我的联迪刷卡器" // 标签
  }
}
```
#### 2\. 更换一个已知设备，或者绑定一个新设备到已有终端
> 更换一个设备后，原有的KSN状态将被置为作废状态

请求：  
```
PUT /acq/500100002000120/device/235234 HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 80

{
  "ksn": "600010000002",  // KSN编号，
  "isbluetooth": true, // 是否是蓝牙设备，如果不是，就没有bluetooth字段，服务器会验证该字段
  "bluetooth": {
    "btmac": "AA:BB:CC:DD:EE:01",
    "btname": "M35-66775679"
  },
  "model": "landim35",
  "merchantcode": "M12130000000001",
  "businesscode": "SJSDSDK",
  "label": "我的联迪刷卡器2" // 标签
}
```
响应：  
```
{
  "status": "00",
  "成功",
  "data": {
    "_id": "235234", // 终端id
    "_createtime": 2342421341,
    "terminalno": "89746572", // 中汇终端号
    "ksn": "600010000002",
    "status": 1, // 0表示未使用，1表示已使用，2表示已作废，只有值为1，才有后续数据
    "isbluetooth": true,
    "bluetooth": {
      "btmac": "AA:BB:CC:DD:EE:01",
      "btname": "M35-66775679"
    },
    "model": "landim35",
    "merchantno": "500100002000120",
    "label": "我的联迪刷卡器2" // 标签
  }
}
```
##### [返回目录↑](#content-title)
### <a name="order"></a>订单 /order
#### 1\. 创建一个订单
请求：  
```
POST /order HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 100

{
  "amount": 12300,  // ￥123.00
  "currency": "CNY", // default
  "merchantcode": "M12130000000001",
  "ordername": "",
  "orderinfo": {
    "url": null,
    "orderdetail": null
  },
  "expired": "2d5h2m10s" // or numeric time acquired like new Date().getTime()
}

```
响应：  
```
{
  "status": "00",
  "成功",
  "data": {
    "_id": 3579246,
    "_createtime": 1928739283,
    "orderno": "20150504091240100001",
    "amount": 12300,  // ￥123.00
    "currency": "CNY", // default
    "merchantcode": "M12130000000001",
    "businesscode": null, // 当status 不为0才有值，否则为null
    "ordertoken": "c7691dc9-f37c-4757-8572-3add9fba776d", //时效令牌
    "expired": 1928739998,
    "status": 0,
    "ordername": "乐高玩具",
    "orderinfo": {
      "url": "http://taobao.com/234243324",
      "orderdetail": null
    }
  }
}
```
##### [返回目录↑](#content-title)
### <a name="order1"></a>单一订单 /order/:orderNo
> `orderNo` 指代一个订单号

#### 1\. 根据订单号查询一个订单
请求：  
```
GET /order/20150504091240100001 HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT

```
响应：  
```
{
  "status": "00",
  "成功",
  "data": {
    "_id": 3579246,
    "_createtime": 1928739283,
    "orderno": "20150504091240100001",
    "amount": 12300,  // ￥123.00
    "currency": "CNY", // default
    "merchantcode": "M12130000000001",
    "businesscode": "SJSDSDK",
    "expired": 1928739998,
    "status": 1, // 0表示init，1表示pending，2表示trading，3表示成功，4表示失败
    "ordername": "乐高玩具",
    "orderinfo": {
      "url": "http://taobao.com/234243324",
      "orderdetail": null
    }
  }
}
```
##### [返回目录↑](#content-title)
### <a name="order2"></a>订单详情 /order/:orderNo/status
> `orderNo` 指代一个订单号

#### 1\. 根据订单号查询一个订单的详情结果
请求：  
```
GET /order/20150504091240100001/status HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT

```
响应：  
```
{
  "status": "00",
  "成功",
  "data": {
    "status": 3, // 订单结果, 只有status为3，才会有交易详情detail的结果，否则detail为null或{}
    "detail": {
      "transno": "1324354657", // 交易代号 意义由实际交易业务决定
      "businesscode": "SJSDSDK",
      "merchantname": "小刚的小店",
      "merchantno": "500100002000120",
      "terminalno": "89746572",
      "amount": 12300, // ￥123.00
      "cardno": "6222020200068682136", // 卡号
      "transtype": "sale",
      "authno": "", // 授权码有时候没有
      "refno": "123456789123456", // 参考号
      "patchno": "147895461", // 批次号
      "voucherno": "123145", // 流水号
      "transdate": 1928739283,
      "operater": "01",
      "receipturl": "/acq/receipt/xxxxxxxxxxxxx" // 小票下载
    } 
  }
}
```
##### [返回目录↑](#content-title)
### <a name="receipt"></a>交易小票 /acq/receipt/:name
> `name` 指代一个收单交易详情中的小票名称

#### 1\. 交易小票图片的获取
请求：  
```
GET /acq/receipt/20150504091240100001.jpg HTTP/1.1
Host: api.vcpos.cn
Authorization: SIGN appid:md5signature
Date: Wed, 8 Apr 2015 15:51 GMT

```
响应：  
```
图片的binary
```
##### [返回目录↑](#content-title)
