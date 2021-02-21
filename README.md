# ClientAPI_Doc
通讯接口文档
因为团队Github Page收费，所以就写这儿了


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
