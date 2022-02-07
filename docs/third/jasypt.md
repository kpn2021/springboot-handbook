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
# 其中ENC()是默认加密法的固定配置,ENC()内部的字符串为加密后的密文
test.app.param=ENC(8wGAMAxhagkUtGmTbfqq/A==)

```
二、在启动类上添加@EnableEncryptableProperties注解