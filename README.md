# ClientAPI_Doc
通讯接口文档

[前端接口](https://github.com/MotdPlatform/MotdDocs/blob/main/Front_API.md)

[客户端接口](https://github.com/MotdPlatform/MotdDocs/blob/main/Client_API.md)

[相关客户端需求](https://github.com/MotdPlatform/MotdDocs/blob/main/Client_Requirements.md)

[HTTP状态](https://github.com/MotdPlatform/MotdDocs/blob/main/HTTPStatus.md)

接口地址：`https://motd.52craft.cc/api/`
请求方式：`GET/POST`

返回参数:

参数 | 类型 | 备注
 --- | --- | --- 
status | Number | 将严格按照[HTTP状态](https://github.com/MotdPlatform/MotdDocs/blob/main/HTTPStatus.md)规则
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

就是将本来要发送的参数全部一JSON的格式压缩为一个`token`

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
  $key = md5(date("YmdH") . $key);
  $iv = substr($key, 0, 16);
  $key = substr($key, 16);
  if ($operation) return openssl_decrypt(base64_decode($string), "AES-128-CBC", $key, OPENSSL_RAW_DATA, $iv);
  return base64_encode(openssl_encrypt($string, "AES-128-CBC", $key, OPENSSL_RAW_DATA, $iv));
}
```

[JAVA??](https://blog.csdn.net/weixin_34297300/article/details/88849565)

使用测试秘钥`69dcff60ade65ebb803b1b56ba6a3874`，该秘钥在测试阶段将不会变更，请保留（添加）更改方法。

为了保证`KEY`的时效性，请在`KEY`后拼接日期，示例`202102231569dcff60ade65ebb803b1b56ba6a3874`前面的`2021022315`即为日期（精准到小时，24H）。

将将要加密的内容以`String`类型生成密文，作为`token`参数请求`API`即可，同样，服务器返回的也是密文。

请求示例返回：

```json
{
    client: {
      ip: "123.158.110.205"
      ua: "xxxxx",
      addr: "xxxxxx"
    },
    data: "AV36iChBELJbngHIT5SsTtE8TC5WszpA0bPkaYEFv6YsCgQJw+9iliha37BOVku5zd4BxnVXVlPHR8Prnek6XKXonwo/BFTajSyBEYrBsDvi5bWTpFwR13HWdjpHKu6fLBnwMn+yf+fmnuGCUnP2ixxuQNjtuA81AAovkYsyl4Ym0b8mblbPh20qlQ7d+l3IhErLxi3B7PB4IebhottbT/JSjyBnemnBCL3lceIRcDmqEQPKHWUhDcZfy2S2XfaSl/iwnoci4yfsvTl9TljShsC8fZKNfEYiiyfpgIElReS/ouY1NJHmg6M5+TwZ/o9oRUAPtmVY8dPBWLyD2O5kJwN9LiPmV3gy17EKyH0RKwPMfGxwiRwNLa+QOeykBoucYO/k6QwDGOQ6DfVySqrsgjKLEEEkgnfuxmJLKI7H7/SA+Q2ht22dWtAWq/dKjOCW9k/BZS3r9UktxDFxzkaWCpsgBOzS8JFIEfrJE3QLbO+3arOaYKxEs0yLaJW4xyJhkdm72fxXCcAkugTE5Wio7gCcDtTGpzJk8maV/IG2ymi1Q1a6OMwOBaj6IDcz72C/2zWvKqkt4omksM4xigtsmlxdKqFFsvUBtRRult9BZZI0Ls8XWPlKd38nHqdo9ltBZMUEjTR/TZlNMbWGsUS5rKhmYtVtcBATJJSDR4nl3JrqxJJ6SOaCR8qVNSs56uw71Um5CuoMQgW6kWgqIfvFacz/bwq3aTyxK/UgjOOxtENuLQuwN455GnGBbldod6SgNHT6Szpl8B/l6qqVTkUFL3OqvkWL8fDXri/DXU82/sPs6I+9Cfh69OgY36MLEzr9gbaRUKsj8/hFlUr9PAhNFY4kitfpBzFQLv8jV0qKRqkO8kC+/I8ZLR4zgoWZoXnDEJNWg+yvcq98gTpk+NEdq8Hfb+z6PLDVm6f94/8Ht68tsKHnRJ2aZBGuMWn8UeYxZ6rgVYwUs4YKDAFD6jhybkqjn3lrL55OjrUwvoNEkQPehMT+n0hf6rMg4DpRFe/2POqi37YrCFxyqRVUQeztiP1vxbD1gpsV82IyOSaym/TlhwaNI7NXVFufb7TLEY97JstbRupJnDcNmWz5gXOHwiIY9ZhOrP1NuOxHPgTqoGEBdsUq7RJNQ+Y9VRHGwfXp5dmNGze1JJZozjZZ+pSTeXLGhBpY3SICO/bfvxF+Yga5naIZFco7fAI2HJRj87iNUpIgGuVXud35Y7+iLnO4qEr0y80UaKpDwsmhpgn9fG89QWdgkkf+v3a8Pb4M7E3uVVn8kII1OPfTR78ildIUpQTqCa1nAkj4O0sgtMnAlipMmfsImdON/flJxWli/JiSAk49L4C3ub88tSNBXDSEpHk9U1uYn31LkFKUJhdYHcZcnPSLXhkj5A6iY8KPAwtc5F0REC0sGAYnpRaJOPfmIjP4tJBo4v0kKAbgegXGRa0wSlSnb6N+HnnL5UJJKB+rcXD5VCb90jl8iKIk6X91SrJK+sVEbduGIG8fVlGuTVlFLiFzn4qTmareXcSJ8I/pmQF1oS7IN1iU+ihkQVgjUw==",
    msg: "提示请求成功",
    status: 200
}
```

> 上面的`data`内是加密过的内容，而其他参数则没有加密。
> 只需要用解密方法解密`data`内的信息，即可拿取信息即可。

该`data`解密结果：

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
详细前端接口请参照[前端接口](https://github.com/MotdPlatform/MotdDocs/blob/main/Front_API.md)

# 复现请求全过程

1.请求地址（接口）为https://motd.52craft.cc/api/

2.请求方式可以是`GET`/`POST`

3.根据[客户端接口](https://github.com/MotdPlatform/MotdDocs/blob/main/Client_API.md)请求更新服务器缓存

4.已知要传送参数`d` `ip` `port` `data` `interface`四个参数

5.将他们变为`JSON`格式，具体参照[Json传递数据两种方式](https://www.cnblogs.com/dand/p/10031854.html)，最后他们变成了：

```json
{
  d: "auth",
  ip: "127.0.0.1",
  port: 19132,
  data: [],
  interface: "//127.0.0.1/api/"
}
```

6.将以上内容`mcrypt`加密为`PDEKSL+juUCyMduVpanC/nECAP5YZcDHuDzBgwCwoelbrQsYwoSIRkMgj6SWLWZbp7e2Yoi14xRQeF3SYfOWoKKsja4HDrsombGKBRPnlcsuhOBQy/Nx8CJW1d91oFgw`

7.将该字段作为`token`传递：

> https://motd.52craft.cc/api/?token=PDEKSL+juUCyMduVpanC/nECAP5YZcDHuDzBgwCwoelbrQsYwoSIRkMgj6SWLWZbp7e2Yoi14xRQeF3SYfOWoKKsja4HDrsombGKBRPnlcsuhOBQy/Nx8CJW1d91oFgw

8.于是拿到了返回：

```json
{
  client: {
    ip: "123.158.110.205"
    ua: "xxxxx",
    addr: "xxxxxx"
  },
  data: "2301yRIfZq02RKMqJ214lJYTxWNmoEIClWpBHNKxMLAjpQ/Rq7UwJFIYFEWCa5DccqqY/jayueVn7TqzEG3m6LSARF0FF2MeVf9qSub45UjdWbnNDY76upFnalfZ1hCXF+eZW087vW8s4arVboP1JYw9jWqzzC1Nj7hoitFyqcNtb5/SI4zsh8QSu+5In0/Dmwzh1KIkksFnQa2v+QOvnff/B9ZQtYkdqOr6eO3aF4he3FRv+k0+HprTHY/mkTg1r4ARVxdbPoXo7QiCD8mVxaE0PfetRty2ddNypSvAf/FjEp2FPLusNtK4x0E1liA4rA+01v94XcCCQw/HwEfAeeLb+SbTIjiw7AgTG7NhEu+h8XE/HNmHS5B4V4ZQhEKBUkGDrJg4fH2iTklGpDyNydsOH4HoumbS5MvE+mkD6L5PA3DWSY0aLRd4DWJd6j82ng6psHa3F+GFV6nTrKj3ggrvMIUkZIuv6MIUwgbZzbWnasohS4kYFnNxI0y+UDnw/4wUlrqsgoaPeeFmpb8JuRqnRp0HKF8sysWjJoClV21lJVUmIfASev8I+00tdbk2Qf1wtI9faLDKbJpaZKlZTWAVGokeRxWZiIuA8S40RZv3SWzjcayyg/qqUfO/ihBBofYf2N74WRJbyG0FY5Bqy1K/jzeDXLy3UpjrSnoe9PDRzP6Pv9t8qOc4t315Qcr2LGUUEno746AeRQibHsKgvSoqYoNRZBGHnF4WRULAwfqDhFDmkIjUI2zdEyfwvi/7K+GYnnd+fY7byLzoe5k3tkC/QuhoXg0RUwkC74qeVr/+kYQMcu9BTnWWthEJJtM8k3uMCl3Spa8KFPOuP4smQUzAakOwZh9m9t8PMUYl/MD3HYX6xQ8GWJ8tu45hy9XH65E7wAnZeqUUAbLiZgydGYLrNsNop9kc84dxubPPsLFbOSMfjrnRGwIv4/cjR2qBQIPEtdq0Nq99QKLd4+tIIOJTNlNkBN5CIYo34ZfM9HcxANlenJt+mDO2Rez6Gn7UcU/wKV1p8gc6iTOCei/yeF0ZpBa1GwLt15sgcMR/wu+29TgSI03UMkI3RJzquFGkr2cp7Cc5Yz9sHcDX+AadI5aDyweQBEcZlHOSRkulW9/M6VNSSe9cm8C4TK3a0DjFJiYnQ35ySpPaCzOuKqqxGcG72RRX99UpdKG+LlN7MggYpebqBkRxO2Z62tbaGX8KkzxDHhAd6jTZJRjqRy1VMn4rIOB7r9Eew0GxJJAs0FRInqX/fYkT+iZ93e6KCZTvJYE0TkpbCvwedGkxWB8rBsklps6ktonMv8SaPrnJlj1CoswlGr/hEM16p2cSA2w5U3kDkKzg/ygveaVOhDGNUZrV/YYkzBa2PKfzNNp+HPpYgnWrh031OAvxWI3DjKHsdJ5B0qd6inraNe1CQ2o/NMmZHEy44QoZ7iMytxI5z2I="
  msg: "标签请求成功",
  status: 200
}
```

9.可见的`data`参数是加密的，解密获得：

```json
[
  {
    "bid": 1,
    "type": "warning",
    "tag": "老兵",
    "filter": "[{\"query\":10000, \"symbol\":1}, {\"queried\":\"{query}/2\",\"symbol\":1}]"
  },
  {
    "bid": 2,
    "type": "danger",
    "tag": "稳定",
    "filter": "[{\"query\":2000, \"symbol\":1}, {\"queried\":\"{query} / 1.5\", \"symbol\":1}]"
  },
  {
    "bid": 3,
    "type": "warning",
    "tag": "高压电",
    "filter": "[{\"fast\":\"{query}/1.6\", \"symbol\":1}]"
  },
  {
    "bid": 4,
    "type": "success",
    "tag": "新晋之星",
    "filter": "[{\"query\":60, \"symbol\":2}]"
  },
  {
    "bid": 5,
    "type": "primary",
    "tag": "先锋",
    "filter": "[{\"timestamp\":1613809149, \"symbol\":2}]"
  },
  {
    "bid": 6,
    "type": "info",
    "tag": "骚",
    "filter": "[{\"query\":100, \"symbol\":1}, {\"queried\":\"{query}\",\"symbol\":0}]"
  },
  {
    "bid": 7,
    "type": "danger",
    "tag": "土豆服务器",
    "filter": "[{\"delay\":1000, \"symbol\":1}]"
  },
  {
    "bid": 8,
    "type": "danger",
    "tag": "轮椅",
    "filter": "[{\"delay\":250, \"symbol\":1}, {\"delay\":1000,\"symbol\":2}]"
  },
  {
    "bid": 9,
    "type": "success",
    "tag": "闪电侠",
    "filter": "[{\"fast\":\"{query}/1.5\", \"symbol\":1}, {\"delay\":100, \"symbol\":2}]"
  }
]
```
