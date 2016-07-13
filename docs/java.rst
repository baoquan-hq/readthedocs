Java
=================

如果使用maven，可以加入如下依赖::

	<dependency>
	    <groupId>com.baoquan</groupId>
	    <artifactId>eagle-sdk</artifactId>
	    <version>1.0.4</version>
	</dependency>

如果使用gradle，可以加入如下依赖::
	
	compile group: 'com.baoquan', name: 'eagle-sdk', version: '1.0.4'

初始化客户端
------------------

::

	BaoquanClient client = new BaoquanClient();
	// 设置api地址，比如保全网的测试环境地址
	client.setHost("https://stg.baoquan.com"); 
	// 设置access key
	client.setAccessKey("fsBswNzfECKZH9aWyh47fc"); 
	// 设置rsa私钥文件的绝对路径
	client.setPemPath("path/to/rsa_private.pem"); 

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

其它初始化设置
^^^^^^^^^^^^^^^

还有些其它可选的初始化设置，比如设置api版本，设置request id生成器，默认情况下你无需进行这些设置::
	
	// 设置api版本
	client.setVersion("v1") 
	// 设置request id生成器，生成器需要实现RequestIdGenerator接口中的createRequestId方法
	client.setRequestIdGenerator(CustomRequestGenerator) 

	// sdk中默认的request id生成器
	public class DefaultRequestIdGenerator implements RequestIdGenerator {

		@Override
		public String createRequestId() {
			return UUID.randomUUID().toString();
		}
	}

客户端初始化完成后即可调用客户端中的方法发送请求

创建保全
------------------

::

	CreateAttestationPayload payload = new CreateAttestationPayload();
	// 设置模板id
	payload.setTemplateId("5Yhus2mVSMnQRXobRJCYgt"); 
	// 设置陈述是否上传完成，如果设置成true，则后续不能继续追加陈述
	payload.setCompleted(false); 
	// 设置保全所有者的身份标识，标识类型定义在IdentityType中
	Map<IdentityType, String> identities = new HashMap<>();
	identities.put(IdentityType.ID, "42012319800127691X");
	identities.put(IdentityType.MO, "15857112383");
	payload.setIdentities(identities);
	// 添加陈述对象列表
	List<Factoid> factoids = new ArrayList<>();
	// 添加product陈述
	Factoid factoid = new Factoid();
	Product product = new Product();
	product.setName("xxx科技有限公司");
	product.setDescription("p2g理财平台");
	factoid.setType("product");
	factoid.setData(product);
	factoids.add(factoid);
	// 添加user陈述
	factoid = new Factoid();
	User user = new User();
	user.setName("张三");
	user.setRegistered_at("1466674609");
	user.setUsername("tom");
	user.setPhone_number("13452345987");
	factoid.setType("user");
	factoid.setData(user);
	factoids.add(factoid);
	payload.setFactoids(factoids);
	// 调用创建保全接口，如果成功则返回保全号，如果失败则返回失败消息
	try {
		CreateAttestationResponse response = client.createAttestation(payload);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

如果创建保全时需要给陈述上传对应的附件::

	// 创建3个附件，每个附件都是ByteArrayBody实例，ContentType必须为DEFAULT_BINARY，并且需要设置filename
	InputStream inputStream0 = getClass().getClassLoader().getResourceAsStream("seal.png");
	ByteArrayBody byteArrayBody0 = new ByteArrayBody(IOUtils.toByteArray(inputStream0), ContentType.DEFAULT_BINARY, "seal.png");
	InputStream inputStream1 = getClass().getClassLoader().getResourceAsStream("seal.png");
	ByteArrayBody byteArrayBody1 = new ByteArrayBody(IOUtils.toByteArray(inputStream1), ContentType.DEFAULT_BINARY, "seal.png");
	InputStream inputStream2 = getClass().getClassLoader().getResourceAsStream("contract.pdf");
	ByteArrayBody byteArrayBody2 = new ByteArrayBody(IOUtils.toByteArray(inputStream2), ContentType.DEFAULT_BINARY, "contract.pdf");
	// 创建附件map，key为factoids中的角标，此处设置factoids中第1个factoid有1个附件，第2个factoid有2两个附件
	Map<String, List<ByteArrayBody>> attachments = new HashMap<>();
	attachments.put("0", Collections.singletonList(byteArrayBody0));
	attachments.put("1", Arrays.asList(byteArrayBody1, byteArrayBody2));
	// 此处省略payload的创建
	try {
		CreateAttestationResponse response = client.createAttestation(payload, attachments);
		System.out.println(response.getData().getNo());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

追加陈述
------------------

::

	AddFactoidsPayload addFactoidsPayload = new AddFactoidsPayload();
	// 设置保全号
	addFactoidsPayload.setAno("7F189BBB5FA1451EA8601D0693E36FE7");
	// 添加陈述对象
	factoids = new ArrayList<>();
	factoid = new Factoid();
	User user = new User();
	user.setName("张三");
	user.setRegistered_at("1466674609");
	user.setUsername("tom");
	user.setPhone_number("13452345987");
	factoid.setType("user");
	factoid.setData(user);
	factoids.add(factoid);
	addFactoidsPayload.setFactoids(factoids);
	// 调用追加陈述接口，如果成功则返回的success为true，如果失败则返回失败消息
	try {
		AddFactoidsResponse response = client.addFactoids(addFactoidsPayload);
		System.out.println(response.getData().isSuccess());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}

追加陈述的时候同样能为陈述上传附件，跟创建保全为陈述上传附件一样。

获取保全数据
------------------

::

	try {
		GetAttestationResponse response = client.getAttestation("DB0C8DB14E3C44C7B9FBBE30EB179241", null);
		System.out.println(response.getData());
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}	

getAttestation有两个参数，第1个参数ano是保全号，第二个参数fields是一个数组用于设置可选的返回字段

下载保全文件
------------------

::

	try {
		DownloadFile downloadFile = client.downloadAttestation("7FF4E8F6A6764CD0895146581B2B28AA");

		FileOutputStream fileOutputStream = new FileOutputStream(downloadFile.getFileName());
		IOUtils.copy(downloadFile.getFile(), fileOutputStream);
		fileOutputStream.close();
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}
