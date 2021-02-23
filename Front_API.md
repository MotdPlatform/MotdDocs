# 前端接口

## 拉取服务器数据

请求参数：

参数 | 类型 | 备注
--- | --- | ---
d | String | list

返回参数：

参数 | 类型 | 备注
--- | --- | ---
id | Number | 唯一标识（键）
status | String | online/offline
motd | String | 服务器MOTD信息（已过滤）
delay | Number | 延迟
gamemode | String | 游戏模式
ip | String | 查询时使用的地址
port | Number | 查询时用的端口
real | String | 真实的IP地址
location | String | 服务器位置
online | Number | 当前在线玩家
max | Number | 最大玩家数
agreement | Number | 协议版本
version | String | 游戏客户端的适应版本
timestamp | Number | 添加时的时间戳
tip | String | 前置的服务器名片提示
badges | Json | 前置标签组
query | Number | 在平台总查询的次数
queried | Number | 在平台成功被查询（在线）的次数
fast | Number | 延迟达标次数
score | Float | 在平台的分数

## 拉取标签

请求参数：

参数 | 类型 | 备注
--- | --- | ---
d | String | badges

返回参数：

参数 | 类型 | 备注
--- | --- | ---
bid | Number | 唯一标识（键）
type | String | 标签类型
tag | String | 标签内容
filter | Json | 条件

## 拉取加载提示

请求参数：

参数 | 类型 | 备注
--- | --- | ---
d | String | tips

返回参数：

参数 | 类型 | 备注
--- | --- | ---
tid | Number | 唯一标识（键）
content | String | 提示内容
