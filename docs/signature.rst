签名
=================

通过API请求保全网时，每一个请求的payload都要用RSA私钥进行签名，这样保全网也无法伪造您的数据。

**第一步：注册保全网账号，在个人中心新增组织并认证，等待审核通过。**

**第二步：进入组织的控制面板，点开密钥管理。**

.. image:: images/open_key_manager.png 

**第三步：上传RSA公钥**

.. image:: images/create_key.png 

可以参考如下shell命令制作自己的证书，openssl版本要求>=1.0.2::

	openssl req -x509 -newkey rsa:1024 -nodes -keyout key.pem -out cert.pem

将cert.pem中的内容粘贴到文本框中，然后选择是沙盒还是正式。
在正式环境下编辑好的模板可以在沙盒环境下使用，但沙盒环境下的数据会被定期清除。所以在您的开发环境和测试环境下请使用沙盒的access key，在生产环境使用正式的access key。

.. note:: 请保管好自己的私钥，另外强烈推荐在正式环境下使用CA机构签发的证书作为公钥。

**第四步：构造签名**

RSA公钥上传成功后会得到一个Access Key，如图所示：

.. image:: images/use_key.png

假定待签名数据为::

	{
		"request_id": "2XiTgZ2oVrBgGqKQ1ruCKh",
		"access_key": "2y7cg8kmoGDrDBXJLaizoD",
		"tonce": 1464594744,
		"payload": {
			"template_id": "2hSWTZ4oqVEJKAmK2RiyT4",
		}
	}

签名过程用Java代码描述如下::

	// RSA私钥文件路径
	String keyFile = "/path/to/rsa_key.pem";
	
	// 请求数据
	String requestId = "2XiTgZ2oVrBgGqKQ1ruCKh";
	String accessKey = "2y7cg8kmoGDrDBXJLaizoD";
	String tonce = 1464594744;
	String payload = "{\"template_id\": \"2hSWTZ4oqVEJKAmK2RiyT4\"}";

	// API path
	String apiVersion = "v1";
	String apiName = "attestations";
	String path = String.format("/api/%s/%s", apiVersion, apiName);

	// 待签名数据 = 请求方法+API路径+requestId+accessKey+tonce+payload
	String data = "".concat("POST").concat(path).concat(requestId).concat(accessKey).concat(tonce).concat(payload);

	// 构造签名
	PEMReader pemReader = new PEMReader(new InputStreamReader(new FileInputStream(keyFile)));
	PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(pemReader.readPemObject().getContent());
	pemReader.close();
	KeyFactory keyFactory = KeyFactory.getInstance("RSA");
	PrivateKey privateKey = keyFactory.generatePrivate(pkcs8EncodedKeySpec);
	Signature signature = Signature.getInstance("SHA256WithRSA");
	signature.initSign(privateKey);
	signature.update(data.getBytes("UTF-8"));

	// 签名使用Base64编码后得到的值即为请求数据中signature字段的值
	String signatureEncoded = Base64.getEncoder().encodeToString(signature.sign());

.. note:: 签名所用的方法是SHA256WithRSA，签名数据字符串转换成bytes时要用UTF-8编码格式









