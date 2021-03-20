# 客户端接口

## 更新服务器的客户端缓存（SQL）

请求参数：（带\*字样是必传参数）

参数 | 类型 | 备注 | 示例
 --- | --- | --- | ---
\*t | String | 目标 | auths
\*ip | String | 服务器地址 | 192.168.168.111
\*port | String | 服务器端口 | 19132
\*data | String | 需要存储的数据 | {"players":\[\], "plugins":\[\]}
\*timestamp | Number | 请求时的时间戳（s） | 1615006264
key | String | 特殊鉴权的key，用于交互机器人 | xxxxxxxxxxxxxxxxx
email | String | 配置文件下的邮箱（空则不进行邮件服务） | 
\*name | String | 服务器的名称 (请使用Base64转码) | Nukkit 服务器
\*tps | String | 服务器的TPS | 20.0
\*load | String | 服务器的Load状态 | 0.0

interface | String | 中转双向返回的通讯接口，只能在Create(第一次请求)时保存，后将无法更改，服务器将通过该接口发送相关数据（题目，算法）进行动态转调（测试中），或者从中获得服务器的相关数据（插件列表，在线玩家uuid，等） | http(s)://127.0.0.1:2333/router/

返回`data`: 检索到的服务器相关信息
带\*的参数传入时将覆盖原来的内容，没必要参数可以不传。


## 发送邮件

请求参数：

参数 | 类型 | 备注 | 示例
 --- | --- | --- | ---
 t | String | 目标 | mail
 to | String | 发送的目标邮箱 | ikers@foxmail.com
 subject | String | 标题 | 你的服务器负载过高
 content | String | 详细内容 | 你的服务器负载高达xxx，时间：xxx，...
 
 无`data`返回。
