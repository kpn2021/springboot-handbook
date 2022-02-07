# springboot读取配置项

## 添加POM依赖
```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```
## 使用@Value读取

1.在application.properties中配置

```
testcfg.param=123456

```
2.在某service类中读取

```
    @Value("${testcfg.param:123}")
    private String testcfg;
```


> 语法1: ${配置项:配置项的默认值}

说明:当application.properties中找不到这个配置时，取这里的默认值



## 使用自定义配置类读取

1.在application.properties中配置

```
my.name=wefwf
my.age=27
```

2.编写对应的配置类

```java
@Data
@Component
@ConfigurationProperties(prefix = "my")
public class TestBean {
    private String name;
    private int    age;
}
```

3.在某类中读取这些配置

```java     
    @Autowired
    private TestBean testBean;

    @GetMapping("/testBean")
    public String testBean() {
        return testBean.toString() + "!!!";
    }
```

## yml与properties配置文件互转

```
 SpringBoot的配置文件都放在src/main/resources目录。
 有两种格式：一种是properties结尾的，一种是yaml或者yml文件结尾的，例application.properties，application.yml
```

> properties文件的编写格式

```
# 注释写法
key=val
key.a=213  # 读出来的是字符串
key.b='qwer'
# 数组写法 ，key.c读出来是一个字符串数组
key.c[0]=werq
key.c[1]=dfefg
```

> yml文件编写格式

```yml
# 以空格的缩进程度来控制层级关系，空格个数不重要

server:
  port: 8080
spring:
  application:
    name: 'helloworld'  

```

### 在线互转yml与properties配置

https://www.toyaml.com/index.html
