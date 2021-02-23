# ClientAPI_Doc
通讯接口文档
因为团队Github Page收费，所以就写这儿了

[相关客户端需求](https://github.com/MotdPlatform/ClientAPI_Doc/blob/main/CILENT_REQUIRE.md)

接口地址：`https://motd.52craft.cc/api/`
请求方式：`GET/POST`

返回参数:

参数 | 类型 | 备注
 --- | --- | --- 
status | Number | 将严格按照[HTTP_STATUS](https://github.com/MotdPlatform/ClientAPI_Doc/blob/main/HTTP_STATUS.md)规则
msg | String | 附加信息
client| Object | 客户端信息
data | String | 请求数据`Token`

示例:
```json
{
  "status":400,
  "msg":"参数错误",
  "client":{
    "ip":"123.158.111.51",
    "ua":"Mozilla\/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/88.0.4324.182 Safari\/537.36"
  },
  "data":null
}
```

## 请求变更Token令牌（mcrypt）

将所有请求参数都写入一个参数内：`token`

例如：https://motd.52craft.cc/api/?token=Yv2tGV6cMnrJBK%2FShLGRzA%3D%3D

可以看到携带了`token`，并只携带了`token`

### 如何得到`token`

JavaScript方法：

```js
  doCrypt: function (string, key, operation) {
    let d = new Date(),
      dMonth = (d.getMonth() + 1).toString(),
      dDate = d.getDate().toString(),
      dHour = d.getHours().toString(),
      dKey = d.getFullYear().toString() + ((dMonth.length == 2) ? dMonth : 0 + dMonth).toString() + ((dDate.length == 2) ? dDate : 0 + dDate).toString() + ((dHour.length == 2) ? dHour : 0 + dHour).toString();

    key = CryptoJS.MD5(dKey + key.toString()).toString();
    let iv = CryptoJS.enc.Utf8.parse(key.substring(0, 16));
    key = CryptoJS.enc.Utf8.parse(key.substring(16));

    if (operation) {
      return CryptoJS.AES.decrypt(string, key, {
        iv: iv,
        padding: CryptoJS.pad.Pkcs7
      }).toString(CryptoJS.enc.Utf8);
    }
    return CryptoJS.AES.encrypt(JSON.stringify(string), key, {
      iv: iv,
      mode: CryptoJS.mode.CBC,
      padding: CryptoJS.pad.Pkcs7
    }).toString();
  }
```

PHP方法

```php
  static function doCrypt($string, $key, $operation = false)
  {
      $key = md5(date("Ymd") . $key);
      $iv = substr($key, 0, 16);
      $key = substr($key, 16);
      if ($operation) return openssl_decrypt(base64_decode($string), "AES-128-CBC", $key, OPENSSL_RAW_DATA, $iv);
      return base64_encode(openssl_encrypt($string, "AES-128-CBC", $key, OPENSSL_RAW_DATA, $iv));
    }
```

[JAVA??](https://blog.csdn.net/weixin_34297300/article/details/88849565)

使用测试秘钥`69dcff60ade65ebb803b1b56ba6a3874`，该秘钥在测试阶段将不会变更，请保留（添加）更改方法。

为了保证KEY的时效性，请在KEY后拼接日期，示例`202102231569dcff60ade65ebb803b1b56ba6a3874`前面的`2021022315`即为日期（精准到小时，24H）。

将将要加密的内容以String类型生成密文，作为`token`参数请求API即可，同样，服务器返回的也是密文。

例如：

```json
client: {ip: "123.158.110.205",…}
data: "AV36iChBELJbngHIT5SsTtE8TC5WszpA0bPkaYEFv6YsCgQJw+9iliha37BOVku5zd4BxnVXVlPHR8Prnek6XKXonwo/BFTajSyBEYrBsDvi5bWTpFwR13HWdjpHKu6fLBnwMn+yf+fmnuGCUnP2ixxuQNjtuA81AAovkYsyl4Ym0b8mblbPh20qlQ7d+l3IhErLxi3B7PB4IebhottbT/JSjyBnemnBCL3lceIRcDmqEQPKHWUhDcZfy2S2XfaSl/iwnoci4yfsvTl9TljShsC8fZKNfEYiiyfpgIElReS/ouY1NJHmg6M5+TwZ/o9oRUAPtmVY8dPBWLyD2O5kJwN9LiPmV3gy17EKyH0RKwPMfGxwiRwNLa+QOeykBoucYO/k6QwDGOQ6DfVySqrsgjKLEEEkgnfuxmJLKI7H7/SA+Q2ht22dWtAWq/dKjOCW9k/BZS3r9UktxDFxzkaWCpsgBOzS8JFIEfrJE3QLbO+3arOaYKxEs0yLaJW4xyJhkdm72fxXCcAkugTE5Wio7gCcDtTGpzJk8maV/IG2ymi1Q1a6OMwOBaj6IDcz72C/2zWvKqkt4omksM4xigtsmlxdKqFFsvUBtRRult9BZZI0Ls8XWPlKd38nHqdo9ltBZMUEjTR/TZlNMbWGsUS5rKhmYtVtcBATJJSDR4nl3JrqxJJ6SOaCR8qVNSs56uw71Um5CuoMQgW6kWgqIfvFacz/bwq3aTyxK/UgjOOxtENuLQuwN455GnGBbldod6SgNHT6Szpl8B/l6qqVTkUFL3OqvkWL8fDXri/DXU82/sPs6I+9Cfh69OgY36MLEzr9gbaRUKsj8/hFlUr9PAhNFY4kitfpBzFQLv8jV0qKRqkO8kC+/I8ZLR4zgoWZoXnDEJNWg+yvcq98gTpk+NEdq8Hfb+z6PLDVm6f94/8Ht68tsKHnRJ2aZBGuMWn8UeYxZ6rgVYwUs4YKDAFD6jhybkqjn3lrL55OjrUwvoNEkQPehMT+n0hf6rMg4DpRFe/2POqi37YrCFxyqRVUQeztiP1vxbD1gpsV82IyOSaym/TlhwaNI7NXVFufb7TLEY97JstbRupJnDcNmWz5gXOHwiIY9ZhOrP1NuOxHPgTqoGEBdsUq7RJNQ+Y9VRHGwfXp5dmNGze1JJZozjZZ+pSTeXLGhBpY3SICO/bfvxF+Yga5naIZFco7fAI2HJRj87iNUpIgGuVXud35Y7+iLnO4qEr0y80UaKpDwsmhpgn9fG89QWdgkkf+v3a8Pb4M7E3uVVn8kII1OPfTR78ildIUpQTqCa1nAkj4O0sgtMnAlipMmfsImdON/flJxWli/JiSAk49L4C3ub88tSNBXDSEpHk9U1uYn31LkFKUJhdYHcZcnPSLXhkj5A6iY8KPAwtc5F0REC0sGAYnpRaJOPfmIjP4tJBo4v0kKAbgegXGRa0wSlSnb6N+HnnL5UJJKB+rcXD5VCb90jl8iKIk6X91SrJK+sVEbduGIG8fVlGuTVlFLiFzn4qTmareXcSJ8I/pmQF1oS7IN1iU+ihkQVgjUw=="
msg: "提示请求成功"
status: 200
```

只需要用解密方法，拿取信息即可。

将此`data`解密为：

```json
[
  {
    "tid": 1,
    "content": "要让服务器排名更靠前，需要实时维护服务器状态。"
  },
  {
    "tid": 2,
    "content": "低延迟的服务器排名更容易上升。"
  },
  {
    "tid": 3,
    "content": "平台不提供游戏伺服，只是对伺服进行数据处理。"
  },
  {
    "tid": 4,
    "content": "感谢您来到MOTD，使用工具栏查询服务器就可以排队上传哦。"
  },
  {
    "tid": 5,
    "content": "你对你的服务器很有信心吗？那就来看看吧"
  },
  {
    "tid": 6,
    "content": "时间不代表一切，平台保证了高质量的检测和记录。"
  },
  {
    "tid": 7,
    "content": "你是否能在这儿鳌头独占？"
  },
  {
    "tid": 8,
    "content": "您的每一次请求都将被记录，请勿进行非法请求！"
  }
]
```


## 更新服务器的客户端缓存（SQL）

请求参数:

参数 | 类型 | 备注 | 示例
 --- | --- | --- | ---
t | String | 目标 | auth
ip | String | 服务器地址 | 192.168.168.111
port | String | 服务器端口 | 19132
data | Json | 需要存储的数据 | \[{"test":100, "name":"joy"},{"test1":"havefun!"}\]
interface | String | 中转双向返回的通讯接口，只能在Create(第一次请求)时保存，后将无法更改，服务器将通过该接口发送相关数据（题目，算法）进行动态转调（测试中），或者从中获得服务器的相关数据（插件列表，在线玩家uuid，等） | http(s)://127.0.0.1:2333/router/

返回`data`: 检索到的服务器相关信息
