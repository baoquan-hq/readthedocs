PHP
=================

如果使用composer，可以加入如下依赖::

	{
	  "require": {
	    "baoquan/eagle-sdk": "1.1.1"
	  }
	}

初始化客户端
------------------

::

	$client = new BaoquanClient();
	// 设置api地址，比如保全网的测试环境地址
	$client->setHost('https://www.baoquan.com');
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
	$client->setVersion('v2');
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
			// 设置保全唯一码
			'unique_id'=>'5bf54bc4-ec69-4a5d-b6e4-a3f670f795f3',
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
			    	'unique_id'=>'9826dc2a-f5af-4433-ab8d-f515630677c2',
			        'type'=>'product',
			        'data'=>[
			            'name'=>'浙金网',
			            'description'=>'p2g理财平台'
			        ]
			    ],
			    // user陈述
			    [
			    	'unique_id'=>'c83d838e-3844-4689-addf-ca0f01171e7c',
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
		$response = $client->addFactoids([
			// 设置保全号
			'ano'=>'7F189BBB5FA1451EA8601D0693E36FE7',
			// 陈述对象列表
			'factoids'=>[
			    [
			    	'unique_id'=>'c83d838e-3844-4689-addf-ca0f01171e7c',
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

获取保全数据
------------------

::

	try {
		$response = $client->getAttestation('DB0C8DB14E3C44C7B9FBBE30EB179241');
		var_dump($response['data']);
	} catch (ServerException $e) {
		echo $e->getMessage();
	}

getAttestation有两个参数，第1个参数ano是保全号，第二个参数fields是一个数组用于设置可选的返回字段

下载保全文件
------------------

::

	try {
		$response = $client->downloadAttestation('DB0C8DB14E3C44C7B9FBBE30EB179241');
		$file = fopen($response['file_name'], 'w');
		fwrite($file, $response['file']->getContents());
		fclose($file);
	} catch (ServerException $e) {
		echo $e->getMessage();
	}

返回的response有两个字段，file_name表示文件名，file是一个\\Psr\\Http\\Message\\StreamInterface实例

申请ca证书
------------------

申请个人ca证书::

	try {
		$response = $client->applyCa([
			'type'=>'PERSONAL',
			'link_name'=>'张三',
			'link_id_card'=>'330184198501184115',
			'link_phone'=>'13378784545',
			'link_email'=>'123@qq.com',
		]);
		echo $response['data']['no'];
	} catch (ServerException $e) {
		echo $e->getMessage();
	}

三证合一情况，申请企业证书::

	try {
		$response = $client->applyCa([
			'type'=>'ENTERPRISE',
			'name'=>'xxx有限公司',
			'ic_code'=>'91332406MA27XMXJ27',
			'link_name'=>'张三',
			'link_id_card'=>'330184198501184115',
			'link_phone'=>'13378784545',
			'link_email'=>'123@qq.com',
		]);
		echo $response['data']['no'];
	} catch (ServerException $e) {
		echo $e->getMessage();
	}

非三证合一情况，申请企业证书::

	try {
		$response = $client->applyCa([
			'type'=>'ENTERPRISE',
			'name'=>'xxx有限公司',
			'ic_code'=>'419001000033792',
			'org_code'=>'177470403',
			'tax_code'=>'419001177470403',
			'link_name'=>'张三',
			'link_id_card'=>'330184198501184115',
			'link_phone'=>'13378784545',
			'link_email'=>'123@qq.com',
		]);
		echo $response['data']['no'];
	} catch (ServerException $e) {
		echo $e->getMessage();
	}

签章图片管理
------------------

上传签章图片::

	try {
	     $attachments = [
	            0=>[
	                [
	                    'resource'=>fopen(__DIR__.'/resources/seal.png', 'r'),
	                    'resource_name'=>'seal.png'
	                ]
	            ]
	    ];

	    $response = $client->uploadContractSignaturePng($attachments);
	    print_r($response['signatureId']);
	} catch (ServerException $e) {
	    echo $e->getMessage();
	}

列出签章图片::

	try {
	    $response = $client->listContractSignature();
	    print_r($response);
	} catch (ServerException $e) {
	    echo $e->getMessage();
	}

设置默认签章图片::

	try {
	    $response = $client->setDefaultContractSignatureId([
	        'signature_id'=>"dtodydoVEtvCVY6ee2RNco"
	        ]);
	    print_r($response);
	} catch (ServerException $e) {
	    echo $e->getMessage();
	}

删除签章图片::

	try {
	    $response = $client->deleteContractSignature([
	        'signature_id'=>"nSysHgKX1Z1sHd9eQGRAe8"
	        ]);

	    print_r($response);
	} catch (ServerException $e) {
	    echo $e->getMessage();
	}

合同管理
------------------

上传pdf合同文件::

	try {
	     $attachments = [
	            0=>[
	                [
	                    'resource'=>fopen(__DIR__.'/resources/contract.pdf', 'r'),
	                    'resource_name'=>'contract.pdf'
	                ]
	            ]
	    ];

	    $response = $client->uploadContractPdf($attachments);
	    print_r($response);
	} catch (ServerException $e) {
	    echo $e->getMessage();
	}

设置合同和内容，在上传合同之后请求::

	try {
	    $response = $client->setContractDetail([
	        "contract_id"=>"qCWgbg26B63SJEio8Wr2rf",
	        "title"=>"这是合同标题",
	        "end_at"=>"2017-12-31",
	        "userPhones"=>["13812345678","13712345678"],
	        "remark"=>"这是备注信息",
	        ]);
	    print_r($response);
	} catch (ServerException $e) {
	    echo $e->getMessage();
	}

请求短信验证码，在修改合同状态之前请求::

	try {
	    $response = $client->requireContractVerifyCode([
	        "contract_id"=>"qCWgbg26B63SJEio8Wr2rf",
	        "phone"=>"18167127094",
	        ]);
	    print_r($response);
	} catch (ServerException $e) {
	    echo $e->getMessage();
	}

修改合同状态 （签署|取消|拒绝）::

	try {
	    $response = $client->signContract([
	        "contract_id"=>"qCWgbg26B63SJEio8Wr2rf",
	        "phone"=>"13812345678",
	        "verify_code"=>123456,
	        "ecs_status"=>"DONE",
	        "page"=>"1",
	        "posX"=>"2",
	        "posY"=>"3",
	        ]);
	    print_r($response);
	} catch (ServerException $e) {
	    echo $e->getMessage();
	}
	
查询合同列表 ::

	try {
	    $response = $client->contractList([
		'status'=>'DONE',
		'keyWord'=>'李四',
		'start'=>'2017-8-30',
		'end'=>'2017-9-1 12:00:00'
		]);
	    print_r($response);
	} catch (ServerException $e) {
	    echo $e->getMessage();
	}
	
查询合同详情::

	try {
	    $response = $client->contractDetail([
		'contract_id'=>'uqg9hB2JQg61g22ma2bFY2'
		]);
	    print_r($response);
	} catch (ServerException $e) {
	    echo $e->getMessage();
	}
