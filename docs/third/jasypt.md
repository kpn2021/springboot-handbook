# jasypt

## 需要的POM依赖

```xml
<dependency>
            <groupId>com.github.ulisesbocchio</groupId>
            <artifactId>jasypt-spring-boot-starter</artifactId>
            <version>2.0.0</version>
        </dependency>
```

## 配置jasypt

一、在application.properties里配置jasypt

```properties

#jasypt的加密配置
#密钥,
# 不建议将jasypt.encryptor.password直接配置在属性文件中，
# 可以通过java -jar -Djasypt.encryptor.password=atpingan  启动程序，作为系统属性、命令行参数或环境变量传递
jasypt.encryptor.password=atpingan
#算法
jasypt.encryptor.algorithm=PBEWithMD5AndDES
#加密器配置
jasypt.encryptor.iv-generator-classname=org.jasypt.iv.NoIvGenerator

```
二、在启动类上添加@EnableEncryptableProperties注解

三、在测试类里把明文加密为密文

```java
@Test
    void contextLoads() {

        BasicTextEncryptor textEncryptor = new BasicTextEncryptor();
        textEncryptor.setPassword("atpingan");

        String userNameS = textEncryptor.encrypt("root");
        String passWordS = textEncryptor.encrypt("123456");

        System.out.println("userName-->  " + userNameS);
        System.out.println("passWord-->  " + passWordS);
        //————————————————
        //        版权声明：本文为CSDN博主「神雕大侠mu」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
        //        原文链接：https://blog.csdn.net/m0_37635053/article/details/118256179
        System.err.println(textEncryptor.decrypt(userNameS));
        System.err.println(textEncryptor.decrypt(passWordS));
    }
```

四、在properties里配置自定义的加密配置项

```

# 其中ENC()是默认加密法的固定配置,ENC()内部的字符串为加密后的密文
test.app.param=ENC(8wGAMAxhagkUtGmTbfqq/A==)

```

五、测试读取自定义的加密配置项

```java
@RestController
public class TestController {

    @Value("${test.app.param:hello}")
    private String testAppParam;

    @GetMapping("/testJasypt")
    public String testJasypt() {
        return testAppParam + "!!!";
    }
}
```


