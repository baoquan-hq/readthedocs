接口
===============

保全 - attestations
----------------------

客户在保全网站上建好模板之后通过该接口传输模板渲染需要的数据。

payload
^^^^^^^^^^^^^^^

=================  ================================ ================
参数名 				描述                             是否可选
=================  ================================ ================
template_id        String字符串，模板id               必选
identities         Object对象，身份事项               必选
factoids           数组对象，陈述集                   必选
completed          Boolean值，是否完成陈述集的上传     可选，默认为true
attachments        数组对象，附件的校验码，可选         可选
=================  ================================ ================

**陈述** 是一个Object对象，包含type和data两个字段，例如::

	{
		"type": "hash",
		"data": {
			"userName": "张三",
			"idCard": "42012319800127691X"
		}
	}

假定客户制作了一份如图所示的模板：

.. image:: images/use_template.png

模板中包含common和hash两个陈述。common中的attestation_no(保全号)、attestation_at(保全时间)、product_name(产品名称)、organization_name(组织名称)在模板渲染时由保全网提供。hash这个陈述是客户自己定义的，所以需要客户通过API上传。

模板中可能包含多个客户自定义的陈述，比如factoA和factoB，此时客户可以选择分两次上传，第一次上传factoA，并设置completed为false，第二次上传factoB，并设置completed为true。

.. note:: 一旦completed设置为true，则不再接受陈述上传。

假定payload如下所示::

	{
		"template_id": "2hSWTZ4oqVEJKAmK2RiyT4",
		"identities": {
			"MO": "15857112383",
			"ID": "42012319800127691X"
		},
		"factoids": [
			{
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
		"template_id": "...",
		"identities": {...},
		"factoids": [
			{
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
		"template_id": "...",
		"identities": {...},
		"factoids": [
			{
				"type": "...",
				"data": {...}
			},
			{
				"type": "...",
				"data": {...}
			}
		],
		"completed": true,
		"attachments": {
			"0": ["checkSum1", "checkSum2"],
			"1": ["checkSum3"]
		}
	}

attachments中的key对应的是factoids数组的角标。

附件的checkSum是对文件进行SHA256产生的，以Java为例::

	String file = "/path/to/file";
	InputStream in = new FileInputStream(new File(file));

	// 使用SHA256对文件进行hash
	bytes[] digestBytes = DigestUtils.getDigest("SHA256").digest(StreamUtils.copyToByteArray(in));

	// 将bytes转换成16进制
	String checkSum = Hex.encodeHexString(digestBytes);

返回的data
^^^^^^^^^^^^^^

调用保全接口成功后会返回保全号

=================  ================================
字段名 				描述                            
=================  ================================
no                 String字符串，保全号              
template_id        String字符串，模板id             
identities         Object对象，身份标识                             
=================  ================================

比如::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"data": {
			"no": "rBgGqKQ1ruCKhXiTgZ2oVr",
			"template_id": "2hSWTZ4oqVEJKAmK2RiyT4",
			"identities": {
				"ID": "42012319800127691X"
			}
		}
	}

追加陈述 - factoids
----------------------

客户可以使用追加陈述接口上传陈述集

payload
^^^^^^^^^^^^^^^

=================  ================================ ================
参数名 				描述                             是否可选
=================  ================================ ================
ano                String字符串，保全号               必选
factoids           数组对象，陈述集                   必选
completed          Boolean值，是否完成陈述集的上传     可选，默认为true
attachments        数组对象，附件的校验码，可选         可选
=================  ================================ ================

返回的data
^^^^^^^^^^^^^^

=================  ================================
字段名 				描述                            
=================  ================================
no                 String字符串，保全号              
template_id        String字符串，模板id             
identities         Object对象，身份标识                             
=================  ================================