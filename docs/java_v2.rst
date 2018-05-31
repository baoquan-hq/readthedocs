Java_v2
=================

如果使用maven，可以加入如下依赖::

	<dependency>
	    <groupId>com.baoquan</groupId>
	    <artifactId>eagle-sdk</artifactId>
	    <version>2.0.0</version>
	</dependency>

如果使用gradle，可以加入如下依赖::
	
	compile group: 'com.baoquan', name: 'eagle-sdk', version: '2.0.0'

初始化客户端
------------------

::

	BaoquanClient client = new BaoquanClient();
	// 设置api地址，比如保全网的测试环境地址
	client.setHost("https://baoquan.com"); 
	// 设置access key
	client.setAccessKey("fsBswNzfECKZH9aWyh47fc"); 
	// 设置rsa私钥文件的绝对路径
	client.setPemPath("path/to/rsa_private.pem");
	client.setVersion("v2");

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


用户认证信息同步
------------------

::

    try {
        UserKycResponse response = client.userKyc("15822222224", "用户一", "42012319800127691X");
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }

企业认证信息同步
------------------

::

	try {
		 KycEnterprisePayload payload = new KycEnterprisePayload();
        payload.setAccountName("潇潇公司");
        payload.setBank("中国银行");
        payload.setBankAccount("111111111111");
        payload.setName("这是我的公司");
        payload.setOrgcode("123456");
        payload.setPhone("17696526111");
        InputStream businessInputStream = getClass().getClassLoader().getResourceAsStream("licence.jpg");
        ByteArrayBody businessFile = new ByteArrayBody(IOUtils.toByteArray(businessInputStream), ContentType.DEFAULT_BINARY, "licence.jpg");
        kycEnterpriseResponse response = client.kycEnterprise(payload, businessFile);
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}



上传签章图片
------------------

::

    try {
         ContractPayload payload = new ContractPayload();
         InputStream inputStream = getClass().getClassLoader().getResourceAsStream("seal.png");
         ByteArrayBody byteArrayBody = new ByteArrayBody(IOUtils.toByteArray(inputStream), ContentType.DEFAULT_BINARY, "seal.png");
         Map<String, List<ByteArrayBody>> byteStreamBodyMap = new HashMap<String, List<ByteArrayBody>>();
         byteStreamBodyMap.put("0", Collections.singletonList(byteArrayBody));
         UploadSignatureResponse u=client.uploadSignature(payload, byteStreamBodyMap);
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }

设置默认签章图片
------------------

::

    try {
         SignaturePayload payload = new SignaturePayload();
         payload.setSignature_id("cey4FBLpqbsUNaLp3SENdp");
         client.setSignatureDefaultId(payload);
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }

签章图片列表
------------------

::

    try {
          client.listSignature();
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }

上传合同
------------------

::

    try {
          ContractPayload payload = new ContractPayload();
          Calendar calendar = Calendar.getInstance();
          Date date = new Date(System.currentTimeMillis());
          calendar.setTime(date);
          calendar.add(Calendar.YEAR, +1);
          date = calendar.getTime();
          System.out.println(date);
          payload.setEnd_at(date);
          payload.setRemark("sas");
          payload.setTitle("ssss合同");
          InputStream inputStream = getClass().getClassLoader().getResourceAsStream("contract.pdf");
          ByteArrayBody byteArrayBody = new ByteArrayBody(IOUtils.toByteArray(inputStream), ContentType.DEFAULT_BINARY, "contract.pdf");
          Map<String, List<ByteArrayBody>> byteStreamBodyMap = new HashMap<String, List<ByteArrayBody>>();
          byteStreamBodyMap.put("0", Collections.singletonList(byteArrayBody));
          UploadContractResponse u = client.uploadContract(payload, byteStreamBodyMap);
          System.out.println(u.getContractId());
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }


发送验证码
------------------

::

    try {
          client.sendVerifyCode("hspH56P7nZU4XSJRWWGvpy", "15811111111","personal");
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }

签署合同或设置合同状态
------------------

::

    try {
        Map<String, String> identitiesMap = new HashMap<String, String>();
        List<PayloadFactoid> list = new ArrayList<PayloadFactoid>();
        PayloadFactoid payloadFactoid = new PayloadFactoid();
        LinkedHashMap<String , Object> linkedHashMap = new LinkedHashMap<String, Object>();
        linkedHashMap.put("userTruename","张三");
        linkedHashMap.put("address", "hangzhou");
        payloadFactoid.setType("product");
        payloadFactoid.setData(linkedHashMap);
        list.add(payloadFactoid);
        identitiesMap.put("MO", "15611111111");
        identitiesMap.put("ID", "430426198401361452");
        client.signContractV2("2B5KcmMKg195rHhLBuNbZB", "15611111111", "5755", "DONE", "4", "400", "550","_priv_template_2", identitiesMap, list,false,"","enterprise","");
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }
	
合同列表
------------------

::

    try {
       ContractListPayload payload = new ContractListPayload();
        payload.setStatus("DONE");
        client.queryList(payload);
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }
	
合同签署详情信息
------------------

::

    try {
       client.getDetail("uqg9hB2JQg61g22ma2bFY2");
    } catch (ServerException e) {
        System.out.println(e.getMessage());
    }
    
签署合同下载
------------------
::

	try {
		DownloadFile downloadFile = client.downloadContract("jVef7CWtiFTvGRZ9ZG6ndD");

		FileOutputStream fileOutputStream = new FileOutputStream(downloadFile.getFileName());
		IOUtils.copy(downloadFile.getFile(), fileOutputStream);
		fileOutputStream.close();
	} catch (ServerException e) {
		System.out.println(e.getMessage());
	}
