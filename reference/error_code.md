# 错误码规范

## 错误代码说明

```json
{
	"code" : "20502",
	"msg" : "Need you follow uid."
}
```
#### 错误代码说明
##### 20502

| 码 | 说明 |
| ------ | ------ | 
|2|错误级别：1 服务级 2 系统级|
|5|服务模块代码|
|2|具体错误代码||

#### 系统错误码

| 错误代码 | 错误信息 | 参数描述|
| ------ | ------ |------ |
|20001|	IDs is null|	IDs参数为空|
20002|	Uid parameter is null|	Uid参数为空|
20003|	User does not exists|	用户不存在|
20004	|Unsupported image type, only suport JPG, GIF, PNG	|不支持的图片类型，仅仅支持JPG、GIF、PNG|
20005|	Image size too large|	图片太大|
20006|	Content is null	|内容为空|
20007|	Text too long, please input text less than 140 characters|	输入文字太长，请确认不超过140个字符|
20008|	Test and verify	|需要验证码|
20009	|Update success, while server slow now, please wait 1-2 minutes	|发布成功，目前服务器可能会有延迟，请耐心等待1-2分钟|
20010|	Auth faild	|认证失败|
20011|	Username or password error	|用户名或密码不正确|
20012|	Username and pwd auth out of rate limit	|用户名密码认证超过请求限制|
20013|	Token expired	|Token已经过期|
20014|	Token revoked	|Token不合法|
20015|	Accessor was revoked|	授权关系已经被解除|
20016|	Geo code input error|	地理信息输入错误||



### XME


| 错误代码 | 错误信息 | 参数描述|
| ------ | ------ |------ |
|10001|	System error|	系统错误|
|10002|	Service unavailable|	服务暂停|
|10003|	Param error，see doc for more info 	参数错误，请参考|API文档|
|10004|	Too many pending tasks, system is busy	任务过多|，系统繁忙|
|10005|	Job expired|	任务超时|
|10006|	RPC error	|RPC错误|
|10007|	Illegal request|	非法请求|
|10008|	Invalid user	|不合法用户|
|10009|	Insufficient app permissions|	 应用的接口访问权限受限|
|10010|	Miss required parameter (%s) , see doc for more info|	 缺失必选参数 (%s)，请参考API文档|
|10011|	Parameter (%s)'s value invalid, expect (%s) , but get (%s) , see doc for more info	|参数值非法，需为 (%s)，实际为 (%s)，请参考API文档|
|10012|	Request body length over limit	|请求长度超过限制|
|10013|	Request api not found	|接口不存在|
|10014|	HTTP method is not suported for this request|	请求的HTTP METHOD不支持，请检查是否选择了正确的POST/GET方式||


### ETL