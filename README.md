# ClientAPI_Doc
通讯接口文档
因为团队Github Page收费，所以就写这儿了

[相关客户端需求](https://github.com/MotdPlatform/ClientAPI_Doc/blob/main/CILENT_REQUIRE.md)

接口地址：`https://motd.52craft.cc/api/`
请求方式：`GET/POST`

返回参数:

参数 | 类型 | 备注
 --- | --- | --- 
status | Number | 将严格按照[HTTP STATUS](https://github.com/MotdPlatform/ClientAPI_Doc/blob/main/HTTP_STATUS.md)规则
msg | String | 附加信息
client| Object | 客户端信息
data | Object | 请求数据

示例:
```json
{
  "status":400,
  "msg":"requested params error!",
  "client":{
    "ip":"123.158.111.51",
    "ua":"Mozilla\/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/88.0.4324.182 Safari\/537.36"
  },
  "data":[]
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
      dHour = (d.getHours() + 1).toString(),
      dKey = d.getFullYear().toString() + ((dMonth.length == 2) ? dMonth : 0 + dMonth).toString() + ((dDate.length == 2) ? d.getDate() : 0 + dDate).toString() + ((dHour.length == 2) ? d.getDate() : 0 + dHour).toString();

    key = CryptoJS.MD5(dKey + key.toString()).toString();
    let iv = CryptoJS.enc.Utf8.parse(key.substring(0, 16));

    key = CryptoJS.enc.Utf8.parse(key.substring(16));
    string = JSON.stringify(string);
    if (operation) {
      return CryptoJS.AES.decrypt(string, key, {
        iv: iv,
        padding: CryptoJS.pad.Pkcs7
      }).toString(CryptoJS.enc.Utf8);
    }
    return CryptoJS.AES.encrypt(string, key, {
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

(JAVA??)[https://blog.csdn.net/weixin_34297300/article/details/88849565]

使用测试秘钥`69dcff60ade65ebb803b1b56ba6a3874`，该秘钥在测试阶段将不会变更，请保留（添加）更改方法。

为了保证KEY的时效性，请在KEY后拼接日期，示例`202102231569dcff60ade65ebb803b1b56ba6a3874`前面的`2021022315`即为日期（精准到小时，24H）。

将将要加密的内容以String类型生成密文，作为`token`参数请求API即可，同样，服务器返回的也是密文。

例如：

```json
client: {ip: "123.158.110.205",…}
addr: "浙江省嘉兴市"
ip: "123.158.110.205"
ua: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.182 Safari/537.36"
data: "ByGXIaidMHBJWaRPMK0PKOwaSuvXgJdc/rCrKxa7ivex+B5cHfKUJ8BzufMlWlNxsrAEPMgJ3l8TfTtk8xGoEadW3bWXsLcanBUlQ/Mj3Ki7FhW95z6QaV8MtsZKqIoemh3ZqzWhRutsYPPq7iDIIWQhuRjBTU8WVSd5VybI7hGAQu01ONDNBRlft0xCdtc6mR4E9sNPLEuGGH9+NXNwZyWASEabtbakiM1/mEUPjC7A5AFSY9tmpyMWZbcAFww338Y/F2Z7eiXMrogQgG8g1EsRxtazVAcpmTd2U1ihzJmMoEYR5ma8MFp0vFZzpra+fW7rJYeqIUx5LflKKkRuVhFWKSxlYUAIMEX8hz7eRPMwA1LzxdSIdsFSuOdVgwoUqbUyZCVfFE0rXXapV0bknnsr+ggHweGWK4wPqvXMGeKNK0R+EIgIahjLziupCbPjINzGg2TaJqnPLnKSKW4pYAUH4Idm/LoMOnBKzRHK8I0sehfxZ6aJ1MWMdJsTwvXI00lhCacosf1gOOuKcUqiEmGAXcXG0sdW742OMvQPMwqNxpJrKrbwzUl7NMnIydmSoT8CPpWgY251wA1TrJal+Ygylsz7Y1+Vwr4FdJJvOL5ydiie+LOuEmwb5GIES1UGVNV+KUV0XxJXWkYB4sCdQUq+k8t92qECB+PnPOoIP4oklG/Fr4KuwaVkoilsVFRXCZ5LpPXWSDQ0aEB6YU73tQqqmROLfEUMi0R5o0kavuXVk63pxqrN+KIPxjw79LaMzMLcmTMP14+sPG23LcYIUWaAb9y6u4E5gI2aujDMW8LrwEQfHBNzn9uhZxJTvzJdbc5DUoyXaeoMXfvBUCxQVK+0hQGfubkhfF7/EHcHLnVZFlVDzkfkZWXvpvuali8Nr3m5FJqlxOSuuzo2qmVqZUk+QXbutwMcRbXoxl3ChNL3j1JuUA3NjYVefh6r2s0GkCIHQ+wYULjTQpNkKxYPod67wEhT56UiY8zZB42jhkYTNzJstbalWERfBk+6iEf0gYnnz7YvOQGU+x6gpKYzTqQEVLvOoHve/r/MdoNOH05Fu8RzdVcHMbk+auAkRs/vsZ3VLFuS4zOBwwUfpyEzahj3jKX9rS4fSQbW74CFaJXWs0nx5Kof38RjVN8hxhfrHGBhpLNCoSrQ5aY8bNhPeonTq70DroYnvwPWk4XDbUmLoTHB7CuNlDmdz3N5BUxqusCOeNU2yH5tLY6SiVt4pkUzXeQmvn0MbBBbtcYuTZUH2oIMpcyy5e7eXlC8Ng5x593IjUtBkZCunzxBhvNTtfahr+t1Nrbb5OVnDcSYYvBFUE0PtlFt0OGqp6jib471MtVJKR7azkHdzuNZHJNjf6RtR4mkNHNUIxRAUgaMEK5pJv4w8Akhn1A4xMnLkmwr3aLdhv6ZNTTxeqL5Ms5ds1F5CC3Q3NL8qkr8GcsUV/NTudKtY3QS94+YT9iQ3Zxi7gbGFviTirIgIUmmMwoh5JUIAjZn7vQYaP4l+x2VrKMVFF6JVgJIOvbKwJfseIt5VSa/GhTbyBW5DuWRpdb7nb9ZN7uVtQq0nbarUxuMRMTDWMC8VbkcTvcwYPObfHFd1/etdr791B1sND8cbJR9PYCp8mftXyHFgI+poOYVhobrhWebImBij0cHqdUn8fe+o0DMwbuvzyGgF/HV5w4ZnBN6pSvd6KWXalbsUHxMSk/WK77dycH6TmiEXURn2TGQtZACqde54EmYruT+0LLr5b/mNpssXuC1W2PyxF9EahG86Yhj8xPfew8vx/brxxKeg2VBh5/5Vn8QmOzXDC+4XA5yaozh3tUUvS/RXDcMKrRk9ioL2NTcJUD0qEjOR9UwqRGG5vUN8yIWku31gKyz0z8GKT/qcX38W/t2Lx+GN21u9G676MrQv+6VBVeV9Rm+oG84JS0HYAxmvJw2Xtm3c3pX8hzDUcELOpFDAqCtSrk="
msg: "提示请求成功"
status: 200
```

只需要用解密方法，拿取信息即可。


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
