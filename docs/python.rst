Python
=================

如果使用pip可以在requirements.txt中加入如下依赖::

	eagle-sdk==1.0.4

.. note:: 请在python 3环境下使用本sdk

初始化客户端
------------------

::

	from baoquan import BaoquanClient

	client = BaoquanClient()
	// 设置api地址，比如保全网的测试环境地址
	client.host = 'https://stg.baoquan.com'
	// 设置access key
	client.access_key = 'fsBswNzfECKZH9aWyh47fc'
	// 设置rsa私钥文件的绝对路径
	client.pem_path = 'path/to/rsa_private.pem'

rsa私钥文件应该以 **-----BEGIN RSA PRIVATE KEY-----** 开头和 **-----END RSA PRIVATE KEY-----** 结尾，比如::

	-----BEGIN RSA PRIVATE KEY-----
	MIICXgIBAAKBgQC0vGnRvfKR0ixKupDk+jxtuRjAc6YkrNaF8Jzy0zlTmGq33xdM
	Uge55OcsW6seQbHiTc9jcck6XiTpPBVxCO6k20uqGAIMh/9hzK+QM96Shi0XSIyU
	K6bt6qSZM0cPNqhj9jJJlZFGHjpfQWRFKZNRps4sK4Ww9M3CSEwNJp1KpwIDAQAB
	AoGAZ3rVD4y03L68M1Eccq2/eYcX3+CXSLpY3TlFc1ZypSVIPNyTh1QULmAb5+7Y
	S6uLgKnSSvq0HyIV+iA3mo9lqrDu2HeGKFcRQJX2hyCAiSrq9eSjl7jzKXZt4pWd
	HNsgk6ydJt15upqcHeG5Qw0rqZpY+BxF14iJxmYMH06jjQECQQDpV9xMrxNjb82D
	Id/cM9dX5jotFIsiL89Ippo3neHYVdGLz48fBo+1xsApM27jcK+7ZZDE2komEfHM
	NUM1eEKBAkEAxkjpYACQvKFD9vruB/gPN430i5YUPFygCZ+Q4p/ZAb7OHijEo06S
	YgwYAOzVAIVHwqktv5uU4k4kt19yI2SpJwJBANomkAkJLOEr50CPbNBbjxnYXc9D
	g4gklm/fghI5AqnUIaHKHI3u/m/9Li3WrfbopQJXw+6l/eh1ok8+BGV61wECQQCf
	gm4DJdFJfW3AVLKBxKLxLQhJ9kyHFnhD5ZJXTQH0rnr/tgoh2YZWy6XPsLXVOmK1
	DQXZex41Q2mz/ltCb6rHAkEAwaFEfGNzVDnRkXGkoME+erT+SdXGcYNW+Lwsq/mH
	73BBkgaxGsRLoA1I8WzjeHrTFX5Yfx90DUr0X+F0OOMEDg==
	-----END RSA PRIVATE KEY-----

其它初始化设置
^^^^^^^^^^^^^^^

还有些其它可选的初始化设置，比如设置api版本，设置request id生成器，默认情况下你无需进行这些设置::
	
	// 设置api版本
	client.version = 'v1' 
	// 设置request id生成器，生成器必须返回一个长度不超过256位的唯一字符串
	// sdk中默认的request id生成器是uuid.uuid4
	def custom_request_id_generator():
		pass 
	client.request_id_generator = custom_request_id_generator

客户端初始化完成后即可调用客户端中的方法发送请求

创建保全
------------------

::

	// 调用创建保全接口，如果成功则返回保全号，如果失败则返回失败消息
	try:
		response = client.create_attestation({
			// 设置模板id
			'template_id': '5Yhus2mVSMnQRXobRJCYgt',
			// 设置保全所有者的身份标识
			'identities': {
			    'ID': '42012319800127691X',
			    'MO': '15857112383'
			},
			// 陈述对象列表
			'factoids': [
			    {
			        'type': 'product',
			        'data': {
			            'name': '浙金网',
			            'description': 'p2g理财平台'
			        }
			    },
			    {
			        'type': 'user',
			        'data': {
			            'name': '张三',
			            'phone_number': '13234568732',
			            'registered_at': '1466674609',
			            'username': 'tom'
			        }
			    }
			],
			// 设置陈述是否上传完成，如果设置成true，则后续不能继续追加陈述
			'completed': true
		})
		print(response['data']['no'])
	except ServerException as e:
		print(e.message)

如果创建保全时需要给陈述上传对应的附件::

	// 创建3个附件，其中product陈述有1个附件，user陈述有两个附件，附件列表的key对应陈述在陈述列表中的角标
	// 每个附件由resource、resource_name和resource_content_type组成，resource是字节数组
	attachments = {
		0: [
			{
			    'resource': open(os.path.dirname(__file__) + '/resources/seal.png', 'rb').read(),
			    'resource_name': 'seal.png',
			    'resource_content_type': 'image/png'
			}
		],
		1: [
			{
			    'resource': open(os.path.dirname(__file__) + '/resources/seal.png', 'rb').read(),
			    'resource_name': 'seal.png',
			    'resource_content_type': 'image/png'
			},
			{
			    'resource': open(os.path.dirname(__file__) + '/resources/contract.pdf', 'rb').read(),
			    'resource_name': 'contract.pdf',
			    'resource_content_type': 'application/pdf'
			}
		]
	}
	// 调用创建保全接口，如果成功则返回保全号，如果失败则返回失败消息
	// 此处省略payload的创建
	try:
		response = client.create_attestation(payload, attachments)
		print(response['data']['no'])
	except ServerException as e:
		print(e.message)

追加陈述
------------------

::

	try:
		response = client.add_factoids({
			// 设置保全号
			'ano': '7F189BBB5FA1451EA8601D0693E36FE7',
			// 陈述对象列表
			'factoids': [
			    {
			        'type': 'user',
			        'data': {
			            'name': '张三',
			            'phone_number': '13234568732',
			            'registered_at': '1466674609',
			            'username': 'tom'
			        }
			    }
			]
			})
		print(response['data']['success'])
	except ServerException as e:
		print(e.message)	

追加陈述的时候同样能为陈述上传附件，跟创建保全为陈述上传附件一样。

获取保全数据
------------------

::

	try:
		response = client.get_attestation('DB0C8DB14E3C44C7B9FBBE30EB179241')
		print(response['data'])
	except ServerException as e:
		print(e.message)	

get_attestation有两个参数，第1个参数ano是保全号，第二个参数fields是一个数组用于设置可选的返回字段

下载保全文件
------------------

::

	try:
		response = client.download_attestation('DB0C8DB14E3C44C7B9FBBE30EB179241')
		with open(response['file_name'], 'wb') as f:
			f.write(response['file_content'])
	except ServerException as e:
		print(e.message)

返回的response有两个字段，file_name表示文件名，file_content是以字节形式表示的文件内容

