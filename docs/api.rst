接口
===============

初始化客户端
------------------
::
	BaoquanClient client = new BaoquanClient();
    //设置api地址，比如保全网的测试环境地址
	client.setHost("https://www.baoquan.com");
    // 设置access key
	client.setAccessKey("fsBswNzfECKZH9aWyh47fc");
    // 设置rsa私钥文件的绝对路径
	client.setPemPath("path/to/rsa_private.pem");
    // 设置版本
	client.setVersion("v2");

保全 - /attestations
----------------------

客户在保全网站上建好模板之后通过该接口传输模板渲染需要的数据。

payload
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
unique_id          String字符串，不超过255位，保全唯一码          必选
template_id        String字符串，模板id                       必选
identities         Object对象，身份事项                        必选
factoids           数组对象，陈述集                           必选
completed          Boolean值，是否完成陈述集的上传            可选，默认为true
attachments        附件信息数组对象                           可选
=================  ======================================= ================

unique_id是保全唯一码，这个唯一码的作用是避免在网络超时或者其它异常的情况下接入方重复上传相同内容的保全数据。如果同样unique_id的保全内容多次上传，保全网只进行1次保全，并返回相同的保全号。

**陈述** 是一个Object对象，包含unique_id,type和data三个字段，例如::

	{
		"unique_id": "9de7be94-a697-4398-945a-678d3f916b9f",
		"type": "hash",
		"data": {
			"userName": "张三",
			"idCard": "42012319800127691X"
		}
	}

陈述的unique_id的作用跟保全的unique_id类似，如果某次保全过程中同样unique_id的陈述内容多次上传到保全网，保全网只处理1次。

type是客户定义的陈述名称，data是陈述的字段值，如下图所示：

.. image:: images/use_template.png

模板中包含common和hash两个陈述。common中的attestation_no(保全号)、attestation_at(保全时间)、product_name(产品名称)、organization_name(组织名称)在模板渲染时由保全网提供。hash这个陈述是客户自己定义的，所以需要客户通过API上传。

.. note::
	在添加陈述对象是要保证陈述对象跟编辑模板时的要求一致。以下几种情况会导致保全失败：

	- 上传了模板中没有的陈述对象，比如模板中没有type为product的对象却上传了。
	- 模板中有字段是必要的，但是完成陈述上传时并没有上传该字段，比如user.name需要上传且不能为空，
	  但是没有上传type为user的data或者data中没有name这个字段。
	- 上传的字段的格式不符，比如模板中要求user.money是int型，但是上传的user.money值是"$100"

模板中可能包含多个客户自定义的陈述，比如factoA和factoB，此时客户可以选择分两次上传，第一次上传factoA，并设置completed为false，第二次上传factoB，并设置completed为true。

.. note:: 一旦completed设置为true，则不再接受陈述上传。

假定payload如下所示::

	{
		"unique_id": "acafa00d-5579-4fe5-93c1-de89ec82006e",
		"template_id": "2hSWTZ4oqVEJKAmK2RiyT4",
		"identities": {
			"MO": "15857112383",
			"ID": "42012319800127691X"
		},
		"factoids": [
			{
				"unique_id": "9de7be94-a697-4398-945a-678d3f916b9f",
				"type": "hash",
				"data": {
					"userName": "李三",
					"idCard": "330124199501017791",
					"buyAmount": 0.3,
					"incomeStartTime": "2015-12-02",
					"incomeEndTime": "2016-01-01",
					"createTime": "2015-12-01 14:33:44",
					"payTime": "2015-12-01 14:33:59",
					"payAmount": 600
				}
			}
		],
		"completed": true
	}

经过保全后，在保全网上可以通过保全号查看经过渲染后的内容，类似下图所示：

.. image:: images/render_template.png

附件
^^^^^^^^^^^^^^^

在上传陈述数据的时候可以同时上传跟该陈述相关的附件，在payload中 **attachments** 存放的是附件的校验码。

form表单形式上传单个附件::

	<form method='post' enctype='multipart/form-data'>
	  ...
	  <input type=file name="attachments[0][]">
	</form>

	payload = {
		"unique_id": "...",
		"template_id": "...",
		"identities": {...},
		"factoids": [
			{
				"unique_id": "...",
				"type": "...",
				"data": {...}
			}
		],
		"completed": true,
		"attachments": {
			"0": ["checkSum"]
		}
	}

form表单形式上传多个附件::

	<form method='post' enctype='multipart/form-data'>
	  ...
	  <input type=file name="attachments[0][]">
	  <input type=file name="attachments[0][]">
	  <input type=file name="attachments[1][]">
	</form>

	payload = {
		"unique_id": "...",
		"template_id": "...",
		"identities": {...},
		"factoids": [
			{
				"unique_id": "...",
				"type": "...",
				"data": {...}
			},
			{
				"unique_id": "...",
				"type": "...",
				"data": {...}
			}
		],
		"completed": true,
		"attachments": {
			"0": [
				"checkSum1",
				{
					"checksum": "checkSum2",
					"sign": {
						"F98F99A554E944B6996882E8A68C60B2": ["甲方（签章）", "甲方法人（签章）"],
						"0A68783469E04CAC95ADEAE995A92E65": ["乙方（签章）"]
					}
				}
			],
			"1": ["checkSum3"]
		}
	}

attachments中的key对应的是factoids数组的下标，比如"0"对应的是factoid为factoids[0]。attachments中的value是一个数组，每个数组元素表示对应附件的附件信息。

附件信息有两种：校验码和电子签名信息，其中校验码是必须提供。当附件信息只有校验码时可以用字符串对象，当包含电子签名信息时需要使用object对象。

.. note:: 只有pdf附件才能进行电子签名。

校验码（checksum）是对文件进行SHA256产生的，以Java为例::

	String file = "/path/to/file";
	InputStream in = new FileInputStream(new File(file));

	// 使用SHA256对文件进行hash
	bytes[] digestBytes = DigestUtils.getDigest("SHA256").digest(StreamUtils.copyToByteArray(in));

	// 将bytes转换成16进制
	String checkSum = Hex.encodeHexString(digestBytes);

电子签名信息（sign）是一个object对象，key值是caId（客户调用申请ca证书接口时会返回caId），value值是签名关键字数组。比如“张三”和“李四”需要在“xxx合同.pdf”附件上进行电子签名，调用ca证书申请接口为“张三”申请得到的caId是"F98F99A554E944B6996882E8A68C60B2"，为“李四”申请得到的caId是"0A68783469E04CAC95ADEAE995A92E65"，其中“张三”需要在"甲方（签章）", "甲方法人（签章）"两个位置进行电子签名，”李四“只需要在"乙方（签章）"进行电子签名，那么sign对象可以表示为::

	"sign": {
		"F98F99A554E944B6996882E8A68C60B2": ["甲方（签章）", "甲方法人（签章）"],
		"0A68783469E04CAC95ADEAE995A92E65": ["乙方（签章）"]
	}

.. note:: 同一个用户可以在多处进行电子签名，但关键字要保证唯一，不能跟正文内容重复。


返回的data
^^^^^^^^^^^^^^

调用保全接口成功后会返回保全号

=================  ================================
字段名 				描述
=================  ================================
no                 String字符串，保全号
=================  ================================

例如::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"data": {
			"no": "rBgGqKQ1ruCKhXiTgZ2oVr",
		}
	}

保全书凭证
^^^^^^^^^^^^^^^

如果是新闻、文章、资讯类等内容保全完成后，需要打上保全网已保全的LOGO，点击查看证书即可跳转到对应保全书上链页面。
如下图所示:

.. image:: images/pingzheng.png

查看保全所对应的JS脚本代码如下::

    <!-- 增加已保全标签链接  title所对应的保全号需根据实际保全内容入参。 -->
    <div id="baoquan-evidence" title="90458CE5C19B402B9BA46905736DF819"></div>
    <!-- 建议将JS放在底部导入，否则可能会影响显示效果  -->
    <script src="https://eagle-p1.oss-cn-szfinance.aliyuncs.com/baoquan.js"></script>

追加陈述 - /factoids
----------------------

客户可以使用追加陈述接口上传陈述集

当上传保全时completed参数为false（未设置默认为true）可追加陈述，陈述接口不产生新的保全号。

payload
^^^^^^^^^^^^^^^

=================  ================================ ================
参数名 				描述                             是否可选
=================  ================================ ================
ano                String字符串，保全号               必选
factoids           数组对象，陈述集                   必选
completed          Boolean值，是否完成陈述集的上传     可选，默认为true
attachments        数组对象，附件的校验码，可选         可选
=================  ================================ ================
ano需要追加陈述的保全号，上传保全成功后会得到应答保全号（请注意上传保全时completed参数的设置，已完成后不可追加陈述）。
factoids具体描述请参考保全接口，
例如::

	{
		"ano": "2hSWTZ4oqVEJKAmK2RiyT4",
		"factoids": [
			{
				"unique_id": "9de7be94-a697-4398-945a-678d3f916b9f",
				"type": "hash",
				"data": {
					"userName": "李三",
					"idCard": "330124199501017791",
					"buyAmount": 0.3,
					"incomeStartTime": "2015-12-02",
					"incomeEndTime": "2016-01-01",
					"createTime": "2015-12-01 14:33:44",
					"payTime": "2015-12-01 14:33:59",
					"payAmount": 600
				}
			}
		],
		"completed": false
	}

返回的data
^^^^^^^^^^^^^^

=================  ================================
字段名 				描述
=================  ================================
success            是否成功，布尔值
=================  ================================

例如::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"data": {
			"success": true,
		}
	}

保全（sha256） - /attestation/hash
------------------------------------
客户在保全网站上建好模板（文件HASH上传）之后通过该接口传输模板渲染需要的数据。

payload
^^^^^^^^^^^^^^^
=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
unique_id          String字符串，不超过255位，保全唯一码      必选
template_id        String字符串，模板（文件HASH模板）id       必选
identities         Object对象，身份事项                       必选
factoids           数组对象，陈述集                           必选
sha256             文件的sha256值                             必选
=================  ======================================= ================

unique_id是保全唯一码，这个唯一码的作用是避免在网络超时或者其它异常的情况下接入方重复上传相同内容的保全数据。如果同样unique_id的保全内容多次上传，保全网只进行1次保全，并返回相同的保全号。

sha256的算法为SHA256WithRSA。

**陈述** 是一个Object对象，包含unique_id,type和data三个字段，例如::

	{
		"unique_id": "9de7be94-a697-4398-333a-678d3f916b9f",
		"type": "file",
		"data": {
			"owner_name": "张三",
			"owner_id": "330124199501017791"
		}
	}

陈述的unique_id的作用跟保全的unique_id类似，如果某次保全过程中同样unique_id的陈述内容多次上传到保全网，保全网只处理1次。

type建议采用系统模板提供的默认值file，data是陈述的字段值，如下图所示：

.. image:: images/use_template_hash.png

模板中包含common和file两个陈述。common中的attestation_no(保全号)、attestation_at(保全时间)、product_name(产品名称)、organization_name(组织名称)在模板渲染时由保全网提供。

file这个陈述为系统默认陈述，数据需要客户通过API上传。

.. note::
	- 文件HASH上传只允许上传一次，不可追加陈述。
	- 该接口不接收附件。
	- 调用该接口时，模板必须为系统提供的文件HASH模板的子模板。

.. note::
	在添加陈述对象是要保证陈述对象跟编辑模板时的要求一致。以下几种情况会导致保全失败：

	- 上传了模板中没有的陈述对象，比如模板中没有type为product的对象却上传了。
	- 模板必须为系统提供的文件HASH模板的子模板，否则，上传失败。

假定payload如下所示::

	{
		"unique_id": "acafa00d-5579-4fe5-93c1-de89ec82006e",
		"template_id": "2hSWTZ4oqVEJKAmK2RiyT4",
		"identities": {
			"MO": "15857112383",
			"ID": "42012319800127691X"
		},
		"factoids": [
			{
				"unique_id": "9de7be94-a697-4398-945a-678d3f916b9f",
				"type": "file",
				"data": {
					"owner_name": "李三",
					"owner_id": "330124199501017791"
				}
			}
		],
		"sha256": "654c71176b207401445fdd471f5e023f65af50d7361bf828e5b1c19c89b977b0"
	}

经过保全后，在保全网上可以通过保全号查看经过渲染后的内容，类似下图所示：

.. image:: images/render_template_hash.png


返回的data
^^^^^^^^^^^^^^

调用保全接口成功后会返回保全号

=================  ================================
字段名 				描述
=================  ================================
no                 String字符串，保全号
=================  ================================

例如::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"data": {
			"no": "rBgGqKQ1ruCKhXiTgZ2oVr",
		}
	}

网页取证 - /attestations/url/asy
------------------------------------
根据网页地址固定证据

payload
^^^^^^^^^^^^^^^
=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
unique_id          String字符串，不超过255位，保全唯一码          必选
template_id        String字符串，模板id                       必选
identities         Object对象，身份事项                        必选
factoids           数组对象，陈述集                           必选
completed          Boolean值，是否完成陈述集的上传            可选，默认为true
url                 String字符串，网页地址                        必选
webName               String字符串，网页名称                        必选
remark                 String字符串，网页备注                        可选
label                 String字符串，网页标签                        可选
=================  ======================================= ================

**陈述** 是一个Object对象，包含unique_id,type和data三个字段，例如::

	{
		"unique_id": "9de7be94-a697-4398-333a-678d3f916b9f",
		"type": "file",
		"data": {
			"owner_name": "张三",
			"owner_id": "330124199501017791"
		}
	}

陈述的unique_id的作用跟保全的unique_id类似，如果某次保全过程中同样unique_id的陈述内容多次上传到保全网，保全网只处理1次。

type建议采用系统模板提供的默认值file，data是陈述的字段值，如下图所示：

假定payload如下所示::

	{
		"unique_id": "acafa00d-5579-4fe5-93c1-de89ec82006e",
		"template_id": "2hSWTZ4oqVEJKAmK2RiyT4",
		"identities": {
			"MO": "15857112383",
			"ID": "42012319800127691X"
		},
		"factoids": [
			{
				"unique_id": "9de7be94-a697-4398-945a-678d3f916b9f",
				"type": "file",
				"data": {
					"owner_name": "李三",
					"owner_id": "330124199501017791"
				}
			}
		],
		"url": "https://www.simplechain.com",
		"webName": "简单上链"
		"remark": "简单上链"
		"label": "sipc"
	}


返回的data
^^^^^^^^^^^^^^

调用保全接口成功后会返回保全号

=================  ================================
字段名 				描述
=================  ================================
no                 String字符串，保全号
=================  ================================

例如::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"data": {
			"no": "rBgGqKQ1ruCKhXiTgZ2oVr",
		}
	}

网页取证状态查询 - /attestations/url/info
------------------------------------
网页取证后状态查询

payload
^^^^^^^^^^^^^^^
=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
ano          String字符串，保全号         必选
=================  ======================================= ================

假定payload如下所示::

	{
		"ano": "acafa00d-5579-4fe5-93c1-de89ec82006e"
	}


返回的data
^^^^^^^^^^^^^^

返回的相关信息

=================  ================================
字段名 				描述
=================  ================================
status                 String字符串，取证状态
message                 String字符串，描述信息
blockchain_hash          String字符串，保全链链上hash
hhf_hash                 String字符串，杭互法司法链链上hash
=================  ================================

例如::

	{
	"data": {
		"status": "SUCCESSED",
		"message": "取证成功",
		"blockchain_hash": "",
		"hhf_hash": null
	},
	"request_id": "c4e7240c-b1d2-4b0a-a73a-3cc1cc0d05f7"
	}



获取保全数据 - /attestation
-------------------------------

客户可以通过该接口获取上传的保全数据，比如身份标识、陈述列表等

payload
^^^^^^^^^^^^^^^

=================  ================================ ================
参数名 				描述                             是否可选
=================  ================================ ================
ano                String字符串，保全号               必选
fields             数组对象，希望返回的字段            可选，默认为null
=================  ================================ ================

由于获取identities、factoids、attachments等字段需要连接数据库、对数据进行解密，耗时较长，所以提供fields进行返回字段的设置。

返回的data
^^^^^^^^^^^^^^

=================  ================================================================
字段名 				描述
=================  ================================================================
no                 保全号
template_id        模板id
identities         身份标识
factoids           陈述列表
completed          陈述是否上传完成
attachments        附件列表
blockchain_hash    区块链hash，当尚未hash到区块链时为空
=================  ================================================================

attachments是一个数组，其中key是factoids中陈述的角标，value是一个附件id数组

（1）当fields为null时会获取所有的字段值，返回的结果例如::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"data": {
			"no": "DB0C8DB14E3C44C7B9FBBE30EB179241",
			"unique_id": "acafa00d-5579-4fe5-93c1-de89ec82006e",
			"template_id" : "5Yhus2mVSMnQRXobRJCYgt",
			"identities": {
				"ID": "42012319800127691X",
				"MO": "15857112383"
			},
			"factoids": [
				{
					"unique_id": "28fcdf56-bff3-4ed9-9f87-c8d35ad49e0c",
					"type": "product",
					"data": {
						"name:: "浙金网",
						"description": "p2g理财平台""
					}
				},
				{
					"unique_id": "e68eb8bc-3d7a-4e22-be47-d7999fb40c9a",
					"type": "user",
					"data": {
						"name": "张三",
						"phone_number": "13234568732",
						"registered_at": "1466674609",
						"username": "tom"
					}
				}
			],
			"completed": true,
			"attachments": {
				"1": [
					"2EHJQPs5j4SZpEKQXQ7r6C",
					"2F81ZJXosNjzrPJsXKywAu"
				]
			},
			"blockchain_hash": "s5j4SZpEKQXQ7r6C2F81ZJXosNjzrPJsXKywAu"
		}
	}

（2）当fields为一个空数组时不会获取identities、factoids和attachments的值，返回的结果例如::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"data": {
			"no": "DB0C8DB14E3C44C7B9FBBE30EB179241",
			"unique_id": "acafa00d-5579-4fe5-93c1-de89ec82006e",
			"template_id" : "5Yhus2mVSMnQRXobRJCYgt",
			"identities": null,
			"factoids": null,
			"completed": true,
			"attachments": null,
			"blockchain_hash": "s5j4SZpEKQXQ7r6C2F81ZJXosNjzrPJsXKywAu"
		}
	}

因此当需要快速获取blockchain_hash时可以设置fields为一个空数组。

（3）当fields为一个非空数组，比如["identities"]，返回的结果例如::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"data": {
			"no": "DB0C8DB14E3C44C7B9FBBE30EB179241",
			"unique_id": "acafa00d-5579-4fe5-93c1-de89ec82006e",
			"template_id" : "5Yhus2mVSMnQRXobRJCYgt",
			"identities": {
				"ID": "42012319800127691X",
				"MO": "15857112383"
			},
			"factoids": null,
			"completed": true,
			"attachments": null,
			"blockchain_hash": "s5j4SZpEKQXQ7r6C2F81ZJXosNjzrPJsXKywAu"
		}
	}

下载保全文件 - /attestation/download
--------------------------------------------------------------

客户上传到保全数据会经过一定的处理（比如模板渲染）生成一份保全文件，这份保全文件才是最终会hash到区块链上的数据，也是最终能通过公证处出公证书或者通过司法鉴定中心出司法鉴定书的数据。

payload
^^^^^^^^^^^^^^^

=================  ================================ ================
参数名 				描述                             是否可选
=================  ================================ ================
ano                String字符串，保全号               必选
=================  ================================ ================

返回的文件
^^^^^^^^^^^^^^^

该接口会返回保全文件以及文件名，文件就是http返回结果的body，文件名存放在http的header中，header的名称是Content-Disposition，header值形如::

	form-data; name=Content-Disposition; filename=5Yhus2mVSMnQRXobRJCYgt.zip

以java为例::

	// 此处省略使用apache http client构造http请求的过程
	// closeableHttpResponse是一个CloseableHttpResponse实例
	HttpEntity httpEntity = closeableHttpResponse.getEntity();
	Header header = closeableHttpResponse.getFirstHeader(MIME.CONTENT_DISPOSITION);
	Pattern pattern = Pattern.compile(".*filename=\"(.*)\".*");
	Matcher matcher = pattern.matcher(header.getValue());
	String fileName = "";
	if (matcher.matches()) {
		fileName = matcher.group(1);
	}
	FileOutputStream fileOutputStream = new FileOutputStream(fileName);
	IOUtils.copy(httpEntity.getContent(), fileOutputStream);
	fileOutputStream.close();


用户认证信息同步 - /users/kyc
-------------------------------

客户可以通过该接口同步实名认证信息到保全网，并自动生成一个用户，同步用户信息后，
创建保全数据的时候，identities 可以使用USERID

payload
^^^^^^^^^^^^^^^

=================  ================================ ================
参数名 				描述                             是否可选
=================  ================================ ================
name                用户姓名                             必选
phone               用户手机号                           必选
idCard              用户身份证号                         必选
=================  ================================ ================



返回的data
^^^^^^^^^^^^^^

=================  ================================================================
字段名 				描述
=================  ================================================================
userId             同步认证信息后，返回保全网自动注册的USERID
=================  ================================================================

例如::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"data": {
			"userId": "avbjetsfgyuyrryjetyDFs",
		}
	}


企业认证信息同步 - /organizations/kyc
-------------------------------

客户可以通过该接口同步企业认证信息到保全网，并自动生成一个用户与此企业关联

payload
^^^^^^^^^^^^^^^

=================  ================================ ================
参数名 				描述                             是否可选
=================  ================================ ================
name                企业名称                            必选
phone               企业关联用户手机号                  必选
accountName         企业开户名称                        必选
bank                企业开户银行                        必选
bankAccount         企业银行账号                        必选
orgcode             统一社会信用代码                    必选
contactCode         联系人身份证号码                    必选
contactName         联系人姓名                             必选
businessFile        营业执照                            必选
=================  ================================ ================
一个用户只能关联一个企业


返回的data
^^^^^^^^^^^^^^

=================  ================================================================
字段名 				描述
=================  ================================================================
kycEnterprise        企业认证信息的键值对
=================  ================================================================

例如::

	{
	    "kycEnterprise": {
		    "bankAccount": "111111111111",
		    "organizationId": "r7qyAncCDN1wDJJL6AotQb",
		    "bank": "中国银行",
		    "rejectReason": null,
		    "accountName": "潇潇公司",
		    "orgcode": "123456",
		    "name": "xxxxx公司",
            "businessFile": "https://baoquan-pub.oss-cn-hangzhou.aliyuncs.com/staging/trust/uploads/kycEnterprise/716d1ff2-e631-4c61-8ced-4553a8d58de4.png",
		    "status": "PASS"
	    }
    }


上传签章图片 - /contract/signature
----------------------

客户在保全网电子签章之前上传用来签章的签章图片。


附件
^^^^^^^^^^^^^^^
同保全附件上传，暂只支持单个签章图片，附件必须是png格式

form表单形式上传单个附件::

	<form method='post' enctype='multipart/form-data'>
	  ...
	  <input type=file name="attachments[0][]">
	</form>

返回的data
^^^^^^^^^^^^^^

调用接口成功后会返回签章图片id

=================  ================================
字段名 				描述
=================  ================================
signatureId         String字符串，签章图片id
=================  ================================

例如::

    {
	   "signatureId":"ejDVGiGeCQ5Ndn6dzsnWx9"
    }

设置默认签章图片 - /contract/signature/default
----------------------

客户在保全网电子签章之前设置用来签章的默认签章图片。

payload
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
signature_id        签章图片id                              必选
phone        用户手机号                             非必选,不填则绑定此accessKey绑定账户
type        用户类型                              非必选，phone不为空时必选
=================  ======================================= ================

假定payload如下所示::

	{
		 "signatureId":"ejDVGiGeCQ5Ndn6dzsnWx9",
		 "phone":"13011111111",
		 "type":"personal"
    }

返回的data
^^^^^^^^^^^^^^

调用接口成功后会返回是否成功

=================  ================================
字段名 				描述
=================  ================================
result             String字符串，设置的结果
=================  ================================

例如::

   {
        "result":"success"
    }


删除默认签章图片 - /contract/signature/delete/default
----------------------

客户在保全网删除用户默认签章图片。

payload
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
signature_id        签章图片id                              必选
phone        用户手机号                                    必选
type        用户类型                                       必选
=================  ======================================= ================

假定payload如下所示::

	{
		 "signatureId":"ejDVGiGeCQ5Ndn6dzsnWx9",
		 "phone":"13011111111",
		 "type":"personal"
    }

返回的data
^^^^^^^^^^^^^^

调用接口成功后会返回是否成功

=================  ================================
字段名 				描述
=================  ================================
result             String字符串，设置的结果
=================  ================================

例如::

   {
        "result":"success"
    }


列出签章图片 - /contract/signature/list
----------------------

客户在保全网电子签章时查看自己所有的签章图片。

返回的data
^^^^^^^^^^^^^^

调用接口成功后会返回是否成功

=================  ================================
字段类型 				描述
=================  ================================
Map                   key-value，key为签章图片id，value为签章图片地址
=================  ================================

例如::

   {
	    "ejDVGiGeCQ5Ndn6dzsnWx9": "https://eagle-p1.oss-cn-szfinance.aliyuncs.com/production/trust/uploads/userSignature/1b338bba-64c1-47d8-bb34-dcb2dbfd7e48.png",
	    "cey4FBLpqbsUNaLp3SENdp": "https://eagle-p1.oss-cn-szfinance.aliyuncs.com/production/trust/uploads/userSignature/5f80cd17-016e-4266-9c35-13266767edb7.png",
	    "gHuVuR2EfvJXAF6D1AqEix": "https://eagle-p1.oss-cn-szfinance.aliyuncs.com/production/trust/uploads/userSignature/fb4a28b2-0d1e-4a61-8913-6a259d06ca5a.png"
    }

上传合同 - /contract/uploadPdf
----------------------
电签接口调用步骤 ： 准备工作:1、个人信息同步（必选）2、企业信息同步（必选）3、上传签章图片（可选）4、设置默认签章图片（可选）
签章步骤：1、上传合同 2、发送验证码 3、签署合同（如有多个签署人则重复第二三步）

客户在保全网电子签章时上传用来签章合同pdf。


附件
^^^^^^^^^^^^^^^
同保全附件上传，暂只支持单个合同，附件必须是pdf格式

payload
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
title              String字符串，合同标题                    必选
end_at             Date类型，合同可以签署的截止时间          必选
remark             String字符串，合同备注                    必选
=================  ======================================= ================

例如::

    {
        "title": "这是xx合同的标题",
        "end_at": "TueAug1418: 08: 29CST2018",
        "remark": "这是xx合同的备注信息"
    }

form表单形式上传单个附件::

	<form method='post' enctype='multipart/form-data'>
	  ...
	  <input type=file name="attachments[0][]">
	</form>

返回的data
^^^^^^^^^^^^^^

调用接口成功后会返回签章图片id

=================  ================================
字段名 				描述
=================  ================================
contractId         String字符串，合同id
=================  ================================

例如::

    {
	   "contractId":"4imixswKjEUU2rzintD3Vx"
    }

发送验证码 - /contract/verifyCode
----------------------

客户在保全网电子签章时按顺序发送验证码。

payload
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
contract_id        String字符串，合同id                      必选
phone              String字符串，当前签署人手机号                   必选
type               String字符串，签署人类型                      必选，（可填"personal"，"enterprise"）
=================  ======================================= ================
type必须根据预申请证书类型填写，personal请事先完成个人实名，enterprise请事先完成企业实名

假定payload如下所示::

    {
        "phone": "15861111111",
        "contract_id": "4imixswKjEUU2rzintD3Vx",
         "type":"personal",
    }

返回的data
^^^^^^^^^^^^^^

=================  ================================
字段名 				描述
=================  ================================
result              String字符串，设置的结果
=================  ================================

例如::

    {
        "result": "success"
    }

签署合同和设置签署合同状态 - /contract/sign
----------------------

客户在保全网签署合同和设置签署合同状态。

payload
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
contract_id        String字符串，合同id                      必选
phone              String字符串，当前签署人手机号                   必选
verify_code        String字符串，收到的验证码                  必选
ecs_status             枚举值，合同状态                    必选（当前可选"DONE"签署）
page                String字符串，签署位置所在页码                    必选
posX                String字符串，签署横坐标位置               必选
posY                String字符串，签署纵坐标位置               必选
template_id        String字符串，模板id                        可选（completed为true必填，可登录保全网创建模板）
identities         Object对象，身份事项                        可选（completed为true必填）
factoids           数组对象，陈述集                            可选（completed为true必填）
completed          Boolean值，是否完成合同签署             必选，false或true
signature_id       String字符串，签章id                       可选，可不填
type               String字符串，签署类型                     必选，（"personal"，"enterprise"）
=================  ======================================= ================
template_id为生成的保全证书模板id（可到官网设置自己的模板）
signature_id为签章图片得id，设置则使用此签章图片签章，不设置则使用用户默认签章图片，如未设置默认签章图片则根据企业实名认证信息或个人实名认证信息生成签章图片
type为签署类型，现有"personal"个人签章，使用个人证书签名；"enterprise"企业签章，默认会使用用户上传的签章图片，如未上传签章图片则根据此账户企业认证名称生成签章图片，使用企业证书签名。
假定payload如下所示::

   {
    "phone": "15811111111",
    "verify_code": "1525",
    "ecs_status": "DONE",
    "contract_id": "4imixswKjEUU2rzintD3Vx",
    "page": "4",
    "posX": "400",
    "posY": "500",
    "template_id": "2hSWTZ4oqVEJKAmK2RiyT4",
    "identities": {
        "MO": "15857112383",
        "ID": "42012319800127691X"
    },
    "factoids": [
        {
            "unique_id": "9de7be94-a697-4398-945a-678d3f916b9f",
            "type": "hash",
            "data": {
                "userName": "李三",
                "idCard": "330124199501017791",
                "buyAmount": 0.3,
                "incomeStartTime": "2015-12-02",
                "incomeEndTime": "2016-01-01",
                "createTime": "2015-12-01 14:33:44",
                "payTime": "2015-12-01 14:33:59",
                "payAmount": 600
            }
        }
    ],
    "completed": false,
    "signature_id":"",
    "type":"",
}

返回的data
^^^^^^^^^^^^^^

=================  ================================
字段名 				描述
=================  ================================
result              String字符串，合同签署结果
=================  ================================

例如::

	{
    		"result": "success"
	}
	
获取合同列表 - /contract/list
----------------------

客户在保全网获取合同列表。

payload
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                           是否可选
=================  ======================================= ================
status              枚举值，合同状态                            可选
keyWord             String字符串，合同标题或签署方             可选
start		    Date类型，合同创建开始时间		    可选
end		    Date类型，合同创建结束时间		    可选
=================  ======================================= ================
假定payload如下所示::

   {
    "status": "DONE",
    "keyWord": "张三",
    "start": "TueAug1418: 08: 29CST2018",
    "end": "TueAug1418: 08: 29CST2018"
}

返回的data
^^^^^^^^^^^^^^

调用接口成功后会返回合同列表

=================  ================================
字段名 				描述
=================  ================================
Map                   key-value，value为数组集合
=================  ================================

例如::

	{
    	  "list": [
		       {
			"attestationId": "FDD989DBC9894C94B3AD26CE7D85FEA2",
			"signUser": "张三，李四",
		        "id": "5j1ugSoK5EzkTmkTypH58u",
			"title": "xxx合同",
			"endAt": "1534505604000",
			"userId": "isxaH5d3EAo3KkBWs1bCLC",
			"createAt": "1502969589000",
			"status": "DONE"
		     },{
			"attestationId": "EB56D19A331E48D78B37250B05563C60",
			"signUser": "张三，王五",
			"id": "vFBB2sXbZDeWVSd91sVUTk",
			"title": "xx合同",
			"endAt": "1534386299000",
			"userId": "isxaH5d3EAo3KkBWs1bCLC",
			"createAt": "1502850267000",
			"status": "DONE"
		        }			
		]
	}
	
获取合同签署详情信息 - /contract/detail
----------------------

客户在保全网获取合同签署详情信息。

payload
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                         是否可选
=================  ======================================= ================
contract_id            String字符串，合同id                      必选
=================  ======================================= ================
假定payload如下所示::

   {
    "contract_id": "jVef7CWtiFTvGRZ9ZG6ndD"
}

返回的data
^^^^^^^^^^^^^^

调用接口成功后会返回合同签署信息

=================  ================================
字段名 				描述
=================  ================================
Map                   key-value，value为合同详情
=================  ================================

例如::

	{
    	   "detail": {			   
		"signList": [
			       {
				"id": "isSpr9bHLZj6CuiZobYgvU",
				"eContractId": "jVef7CWtiFTvGRZ9ZG6ndD",
				"eContractSignId": "cf6cbNnZ4ZAP8stXD4NG5G",
				"userId": "isxaH5d3EAo3KkBWs1bCLC",
				"name": "张三",
				"phoneNumber": "18311111111",
				"signOrder": "1",
				"status": "DONE",
				"createdAt": "1505700143000"
			       },{
				"id": "we1FxES4e4YMNPMp7HZJEq",
				"eContractId": "jVef7CWtiFTvGRZ9ZG6ndD",
				"eContractSignId": "kKFr7E9hT88honVeXkLHLW",
				"userId": "48JGfksQ3LZATZs3TmPTeV",
				"name": "李四",
				"phoneNumber": "18322222222",
				"signOrder": "2",
				"status": "WAIT",
				"createdAt": "1505700140000"
				}
			    ],
				"endDate": "1537279284000",
				"id": "jVef7CWtiFTvGRZ9ZG6ndD",
				"attestationId": "EB56D19A331E48D78B37250B05563C60",
				"startDate": "1505700065000",
      				"isCreator": false,
   				"status": "WAIT_OTHERS",
    				"token": "aa5JMXFiv-6_upl81M8Xzp2cgENyf_HkVKVie40Ouw4plZAVRPpfxSjwF4PMLSTTn0qbsE9wuWsoDwkQ-4D1RAJ1-					     POJAs6hU8yCEufmj45j_SyO4zYcFW0kHPIMjWbJ"
		   }
	}
	
签署合同下载 - /contract/download
----------------------

客户在保全网下载签署合同文件。

payload
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                         是否可选
=================  ======================================= ================
contract_id            String字符串，合同id                      必选
=================  ======================================= ================
假定payload如下所示::

   {
    "contract_id": "jVef7CWtiFTvGRZ9ZG6ndD"
}

返回的文件
^^^^^^^^^^^^^^^

该接口会返回合同文件以及文件名，文件就是http返回结果的body，文件名存放在http的header中，header的名称是Content-Disposition，header值形如::
	
	form-data; name=Content-Disposition; filename=jVef7CWtiFTvGRZ9ZG6ndD.pdf

以java为例::

	// 此处省略使用apache http client构造http请求的过程
	// closeableHttpResponse是一个CloseableHttpResponse实例
	HttpEntity httpEntity = closeableHttpResponse.getEntity();
	Header header = closeableHttpResponse.getFirstHeader(MIME.CONTENT_DISPOSITION);
	Pattern pattern = Pattern.compile(".*filename=\"(.*)\".*");
	Matcher matcher = pattern.matcher(header.getValue());
	String fileName = "";
	if (matcher.matches()) {
		fileName = matcher.group(1);
	}
	FileOutputStream fileOutputStream = new FileOutputStream(fileName);
	IOUtils.copy(httpEntity.getContent(), fileOutputStream);
	fileOutputStream.close();

证据固定 - /copyright/fixedEvidence
------------------------------------
对原创文章和侵权文章进行证据固定。

payload
^^^^^^^^^^^^^^^
=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
unique_id          String字符串，不超过255位，保全唯一码      必选
template_id        String字符串，模板（文件HASH模板）id       必选
identities         Object对象，身份事项                       必选
factoids           数组对象，陈述集                           必选
=================  ======================================= ================


unique_id是保全唯一码，这个唯一码的作用是避免在网络超时或者其它异常的情况下接入方重复上传相同内容的保全数据。如果同样unique_id的保全内容多次上传，保全网只进行1次保全，并返回相同的保全号。

template_id为证据固定模板ID

**陈述** 是一个Object对象，包含unique_id,type和data三个字段，且必须包含一个type 为"qqxx" 的对象,data 中的字段皆为必填字段 如下::

	{
		"unique_id": "9de7be94-a697-4398-333a-678d3f916b9f",
		"type": "qqxx",
		"data": {
                "platFormId": "1",//绑定平台Id
                "ywlj": "https://www.baoquan.com/",
                "ywbt": "hahaha",
                "originalType": "1",
                "url": "https://baoquan.readthedocs.io/zh/latest/api.html#sha256-attestation-hash",
                "qqbt": "这是侵权问题",
                "qqwz": "这是侵权网站",
                "bqgs": "这是版权归属",
                "qqbh": "这是侵权编号",
                "qqzt": "这是侵权主体",
		"oriSubDate": "原文发布时间",//格式为yyyy-MM-dd HH:mm
		"pirSubDate": "侵权文章发布时间",//格式为yyyy-MM-dd HH:mm
                "matchNum": "侵权文章相似度"
		}
	}
**平台代码集**
^^^^^^^^^^^^^^^
=================  =======================================
key 				value
=================  =======================================
1                  微信公众号
2                  知乎
3                  简书
4                  豆瓣
5                  杭州日报报业集团
=================  =======================================

**原创文章类型代码集**
^^^^^^^^^^^^^^^
=================  =======================================
key 				value
=================  =======================================
1	                 体育
2	                 财经
3	                 娱乐
4	                 军事
5	                 电影
6	                 数码
7	                 科技
8	                 政治
9	                 小说
10	                 汽车
11	                 文学
12	                 教育
13	                 法律
14	                 时尚
15	                 艺术
16	                 女性
17	                 地理
18	                 星座
19	                 建筑
20	                 健康
21	                 能源
22	                 历史
23	                 房产
24	                 收藏
25	                 母婴
26	                 读书
27	                 游戏
28	                 旅游
29	                 情感
30	                 心理
31	                 美妆
32	                 家居
33	                 音乐
=================  =======================================
陈述的unique_id的作用跟保全的unique_id类似，如果某次证据固定过程中同样unique_id的陈述内容多次上传到保全网，保全网只处理1次。

陈述中必须包含一个type为qqxx的对象，data是陈述的字段值，如下图所示：

假定payload如下所示::

	{
		"unique_id": "acafa00d-5579-4fe5-93c1-de89ec82006e",
		"template_id": "2hSWTZ4oqVEJKAmK2RiyT4",
		"identities": {
			"MO": "15857112383",
			"ID": "42012319800127691X"
		},
		"factoids": [
			{
                    "unique_id": "9de7be94-a697-4398-945a-678d3f916b9f",
                    "type": "qqxx",
                    "data": {
                                "platFormId": "1",//绑定平台Id
                                "ywlj": "https://www.baoquan.com/",//原文链接
                                "ywbt": "hahaha", //原文标题
                                "originalType": "1", //原创文章标签类型
                                "url": "https://baoquan.readthedocs.io/zh/latest/api.html#sha256-attestation-hash",//侵权URL
                                "qqbt": "这是侵权标题",//侵权标题
                                "qqwz": "这是侵权网站", //侵权网站
                                "bqgs": "这是版权归属", //版权归属
                                "qqbh": "这是侵权编号",//侵权编号
                                "qqzt": "这是侵权主体", //侵权主体
				"oriSubDate": "2018-06-01 19:20",//格式为yyyy-MM-dd HH:mm
				"pirSubDate": "2018-06-02 19:20",//格式为yyyy-MM-dd HH:mm
                                "matchNum": "侵权文章相似度"  //侵权文章相似度
                    }
			}
		]

	}




返回的data
^^^^^^^^^^^^^^

调用证据固定接口成功后会返回证据固定保全号

=================  ================================
字段名 				描述
=================  ================================
no                 String字符串，保全号
=================  ================================

例如::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"data": {
			"no": "rBgGqKQ1ruCKhXiTgZ2oVr",
		}
	}


添加原创 - /copyright/createOriginalArticle
------------------------------------
将用户的原创文章添加到保全网，用户侵权文章全网扫描。

payload
^^^^^^^^^^^^^^^
=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
uniqueId          String字符串，不超过255位，原创文章唯一码      必选
linkUrl           原创文章链接                                必选
nickName          文章发布平台的用户昵称                       必选
originalType      原创文章类型代码                            必选
platformCode      平台代码                                   必选
subDate           文章发布时间                                必选
title             文章标题                                   必选
=================  ======================================= ================

假定payload如下所示::

    {
        "uniqueId": "5SiadRtV6ebSAThUEdKsUF",
        "linkUrl": "http://www.baoquan.com",
        "nickName": "平台昵称",
        "originalType": "1,2",//多种类型，则逗号分隔
        "platformCode": "1",
        "subDate":"2018-06-27 17:16"//格式为“YYYY-MM-DD HH:mm”
        "title":"文章标题"

    }

返回的data
^^^^^^^^^^^^^^

=================  ================================
字段名 				描述
=================  ================================
originalId              String字符串，设置的结果,文章的uniqueId
=================  ================================

例如::

    {
        "originalId": "5SiadRtV6ebSAThUEdKsUF"
    }

客户免验证码签约授权发送验证码 - /authorization/verifyCode
----------------------

客户在保全网电子签章时免发验证码授权验证码。

payload
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                         是否可选
=================  ======================================= ================
phone              String字符串，授权人手机号                    必选
type               String字符串，签署类型                     必选，（"personal"，"enterprise"）
=================  ======================================= ================
phone 为合同组的创建人
假定payload如下所示::

    {
        "phone": "15861111111",
        "type":"personal"
    }

返回的data
^^^^^^^^^^^^^^

=================  ================================
字段名 				描述
=================  ================================
result              String字符串，设置的结果
=================  ================================

例如::

    {
        "result": "success"
    }

客户免验证码签约授权确认 - /authorization
----------------------

客户在保全网电子签章时免发验证码授权确认。

payload
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                         是否可选
=================  ======================================= ================
phone              String字符串，授权人手机号                    必选
verfiy_code          String字符串，验证码                    必选
type               String字符串，签署类型                     必选，（"personal"，"enterprise"）
=================  ======================================= ================
phone 为合同组的创建人
假定payload如下所示::

    {
        "phone": "15861111111",
        "verfiy_code": "1111",
        "type":"personal"
    }

返回的data
^^^^^^^^^^^^^^

=================  ================================
字段名 				描述
=================  ================================
result              String字符串，设置的结果
=================  ================================

例如::

    {
        "result": "success"
    }

