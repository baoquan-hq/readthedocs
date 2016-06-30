PHP
=================

如果使用composer，可以加入如下依赖::

	{
	  "require": {
	    "baoquan/eagle-sdk": "1.0.3"
	  }
	}

初始化客户端
------------------

::

	$client = new BaoquanClient();
	// 设置api地址，比如保全网的测试环境地址
	$client->setHost('https://stg.baoquan.com'); 
	// 设置access key
	$client->setAccessKey('fsBswNzfECKZH9aWyh47fc'); 
	// 设置rsa私钥文件的绝对路径
	$client->setPemPath('path/to/rsa_private.pem'); 

rsa私钥文件应该以 **-----BEGIN PRIVATE KEY-----** 开头和 **-----END PRIVATE KEY-----** 结尾，比如::

	-----BEGIN PRIVATE KEY-----
	MIICeAIBADANBgkqhkiG9w0BAQEFAASCAmIwggJeAgEAAoGBALS8adG98pHSLEq6
	kOT6PG25GMBzpiSs1oXwnPLTOVOYarffF0xSB7nk5yxbqx5BseJNz2NxyTpeJOk8
	FXEI7qTbS6oYAgyH/2HMr5Az3pKGLRdIjJQrpu3qpJkzRw82qGP2MkmVkUYeOl9B
	ZEUpk1GmziwrhbD0zcJITA0mnUqnAgMBAAECgYBnetUPjLTcvrwzURxyrb95hxff
	4JdIuljdOUVzVnKlJUg83JOHVBQuYBvn7thLq4uAqdJK+rQfIhX6IDeaj2WqsO7Y
	d4YoVxFAlfaHIICJKur15KOXuPMpdm3ilZ0c2yCTrJ0m3Xm6mpwd4blDDSupmlj4
	HEXXiInGZgwfTqONAQJBAOlX3EyvE2NvzYMh39wz11fmOi0UiyIvz0immjed4dhV
	0YvPjx8Gj7XGwCkzbuNwr7tlkMTaSiYR8cw1QzV4QoECQQDGSOlgAJC8oUP2+u4H
	+A83jfSLlhQ8XKAJn5Din9kBvs4eKMSjTpJiDBgA7NUAhUfCqS2/m5TiTiS3X3Ij
	ZKknAkEA2iaQCQks4SvnQI9s0FuPGdhdz0ODiCSWb9+CEjkCqdQhococje7+b/0u
	Ldat9uilAlfD7qX96HWiTz4EZXrXAQJBAJ+CbgMl0Ul9bcBUsoHEovEtCEn2TIcW
	eEPlkldNAfSuev+2CiHZhlbLpc+wtdU6YrUNBdl7HjVDabP+W0JvqscCQQDBoUR8
	Y3NUOdGRcaSgwT56tP5J1cZxg1b4vCyr+YfvcEGSBrEaxEugDUjxbON4etMVflh/
	H3QNSvRf4XQ44wQO
	-----END PRIVATE KEY-----

或者以 **-----BEGIN RSA PRIVATE KEY-----** 开头和 **-----END RSA PRIVATE KEY-----** 结尾，比如::

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
	$client->setVersion('v1');
	// 设置request id生成器，生成器需要实现RequestIdGenerator接口中的createRequestId方法
	$client->setRequestIdGenerator(CustomRequestGenerator);

	// sdk中默认的request id生成器
	class DefaultRequestIdGenerator implements RequestIdGenerator
	{
	    /**
	     * create request id
	     * @return string
	     */
	    function createRequestId()
	    {
	        return md5(uniqid("", true));
	    }
	}

客户端初始化完成后即可调用客户端中的方法发送请求

创建保全
------------------

::

	// 调用创建保全接口，如果成功则返回保全号，如果失败则返回失败消息
	try {
		$response = $client->createAttestation([
			// 设置模板id
			'template_id'=>'5Yhus2mVSMnQRXobRJCYgt', 
			// 设置保全所有者的身份标识
			'identities'=>[
			    'ID'=>'42012319800127691X',
			    'MO'=>'15857112383',
			],
			// 陈述对象列表
			'factoids'=>[
			    // product陈述
			    [
			        'type'=>'product',
			        'data'=>[
			            'name'=>'浙金网',
			            'description'=>'p2g理财平台'
			        ]
			    ],
			    // user陈述
			    [
			        'type'=>'user',
			        'data'=>[
			            'name'=>'张三',
			            'phone_number'=>'13234568732',
			            'registered_at'=>'1466674609',
			            'username'=>'tom'
			        ]
			    ]
			],
			// 设置陈述是否上传完成，如果设置成true，则后续不能继续追加陈述
			'completed'=>true
			]
		);
		echo $response['data']['no'];
	} catch (ServerException $e) {
		echo $e->getMessage();
	}

如果创建保全时需要给陈述上传对应的附件::

	// 创建3个附件，其中product陈述有1个附件，user陈述有两个附件，附件列表的key对应陈述在陈述列表中的角标
	// 每个附件由resource和resource_name组成，resource是一个php中的resource对象
	$attachments = [
		0=>[
		    [
		        'resource'=>fopen(__DIR__.'/resources/seal.png', 'r'),
		        'resource_name'=>'seal.png'
		    ]
		],
		1=>[
		    [
		        'resource'=>fopen(__DIR__.'/resources/seal.png', 'r'),
		        'resource_name'=>'seal.png'
		    ],
		    [
		        'resource'=>fopen(__DIR__.'/resources/contract.pdf', 'r'),
		        'resource_name'=>'contract.pdf'
		    ]
		]
	];

	// 调用创建保全接口，如果成功则返回保全号，如果失败则返回失败消息
	// 此处省略payload的创建
	try {
		$response = $client->createAttestation($payload, $attachments);
		echo $response['data']['no'];
	} catch (ServerException $e) {
		echo $e->getMessage();
	}

追加陈述
------------------

::

	try {
		$response = $this->client->addFactoids([
			// 设置保全号
			'ano'=>'7F189BBB5FA1451EA8601D0693E36FE7', 
			// 陈述对象列表
			'factoids'=>[
			    [
			        'type'=>'user',
			        'data'=>[
			            'name'=>'张三',
			            'phone_number'=>'13234568732',
			            'registered_at'=>'1466674609',
			            'username'=>'tom'
			        ]
			    ]
			],
			'completed'=>true
			]
		);
		echo $response['data']['success'];
	} catch (ServerException $e) {
		echo $e->getMessage();
	}

追加陈述的时候同样能为陈述上传附件，跟创建保全为陈述上传附件一样。