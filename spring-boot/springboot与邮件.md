# 邮件任务

- 引入spring-boot-starter-mail
- SpringBoot自动配置MailSenderAutoConfiguration
- 定义MailProperties内容，配置在application.yml中
- 自动装配JavaMailSender

**自动配置类**

**MailSenderAutoConfiguration**

~~~java
@Configuration
@ConditionalOnClass({ MimeMessage.class, MimeType.class, MailSender.class })
@ConditionalOnMissingBean(MailSender.class)
@Conditional(MailSenderCondition.class)
@EnableConfigurationProperties(MailProperties.class)
@Import({ MailSenderJndiConfiguration.class, MailSenderPropertiesConfiguration.class })
public class MailSenderAutoConfiguration {

~~~

**MailProperties**

~~~java
@ConfigurationProperties(prefix = "spring.mail")
public class MailProperties {

	private static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

	/**
	 * SMTP server host.
	 */
	private String host;

	/**
	 * SMTP server port.
	 */
	private Integer port;
~~~

**application.yml**

~~~properties
spring.mail.username=87621996@qq.com
#是授权码
spring.mail.password= xxxxx
#smtp服务器的地址
spring.mail.host=smtp.qq.com
##qq邮箱需要配置ssl
spring.mail.properties.mail.smtp.ssl.enable=true;
~~~

**简单使用**

~~~java
@Autowired
JavaMailSendImpl mailSender;

//简单的邮件
public void sendMail(){
    SimpleMailMessage message = new SimpleMailMessage();
	//设置
    message.setSubject("这个是标题");
    message.setText("这是邮件内容");
    //发给谁
    message.setTo("pengyand@dingding.com");
	//从哪里发
    message.setFrom("87621996@qq.com");
    mailSender.send(message);
}

//复杂邮件
public void sendFzMail(){
    //创建一个复杂的消息邮件
    MimeMessage mimeMessage = mailSender.createMimeMessage();
	//创建一个可以上传文件的helper
    MimeMessageHelper helper =  MimeMessageHelper(mimeMessage,true);
    
    helper.setSubject("这个是标题");
    helper.setText("<h1>支持html</h1>",true);
    //发给谁
    helper.setTo("pengyand@dingding.com");
	//从哪里发
    helper.setFrom("87621996@qq.com");
    
    //上传附件，可上传多个
    helper.addAttachment("1.jpg",new File("/User/xxx.jpg"));
    helper.addAttachment("1.jpg",new File("/User/xxx.jpg"));
	
    mailSender.send(mimeMessage);
    
}

~~~

