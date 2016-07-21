请求
==============
访问保全网的API需要使用https协议，并进行数字签名。


路径
--------------

保全网会提供向后兼容的API，不同的API版本会以比如v1、v2.1这样的字符串标记，当前的版本为v1。

API地址：`https://baoquan.com/api/v1`

请求路径=API地址+接口名称，比如保全接口的名称是/attestations，则请求路径为
`https://baoquan.com/api/v1/attestations`


方法
--------------

所有的请求均为HTTP POST方法

参数
--------------

.. _Unix Time: https://en.wikipedia.org/wiki/Unix_time>

=================  =====================================================================================
参数名 				描述 
=================  =====================================================================================
request_id         请求号，由接入客户创建唯一字符串，长度不超过32位
access_key         访问识别码，当您在保全网成功上传RSA公钥后会获得一个access_key
tonce              请求时间, 必须以 `Unix Time`_ 的格式发送, tonce与服务器时间不得超过正负300秒，否则请求将视为无效
payload            被签名的数据体部分，用JSON进行编码，不同的请求的payload数据一般不同
signature          使用你的RSA私钥进行签名后的字符串，具体签名的方法后面会进一步描述
=================  ===================================================================================== 

例如::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"access_key": "2y7cg8kmoGDrDBXJLaizoD",
		"tonce": 1464594744,
		"payload": {
			"template_id": "2hSWTZ4oqVEJKAmK2RiyT4",
		}
		"signature": "moGDrDBXJLaizoD2hSWTZ4oqVEJKAmK2RiyT4"
	}