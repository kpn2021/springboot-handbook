# SpringBoot默认日志框架logback常用配置

logback-spring.xml方式配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!--为了防止进程退出时，内存中的数据丢失，请加上此选项-->
    <shutdownHook class="ch.qos.logback.core.hook.DelayingShutdownHook"/>
    <!-- 引入 Spring Boot 默认的 logback XML 配置文件  -->
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <!-- 控制台 Appender -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 日志的格式化 -->
        <encoder>
            <!--    CONSOLE_LOG_PATTERN 是  Spring Boot 默认的 logback  配置 defaults.xml里定义的日志格式     -->
            <!--            <pattern>${CONSOLE_LOG_PATTERN}</pattern>-->
            <pattern>
                %clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta}
                %clr(---){faint} %clr([%thread]){faint} %clr(%-40.40logger{39}){cyan} : %line - %clr(:){faint}
                %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}
            </pattern>

            <charset>utf8</charset>
        </encoder>
    </appender>

    <!-- 从 Spring Boot 配置文件中，读取 spring.application.name 应用名 -->
    <springProperty name="applicationName" scope="context" source="spring.application.name"/>
    <!-- 日志文件的路径 -->
    <property name="LOG_FILE" value="logs/${applicationName}.log"/>
    <!-- 日志文件 Appender -->
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <!--滚动策略，基于时间 + 大小的分包策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>%d{yyyy-MM-dd}.%i.${LOG_FILE}.gz</fileNamePattern>
            <!--日志文件保留天数-->
            <maxHistory>2</maxHistory>
            <!--  单个日志文件最大限定100MB  -->
            <maxFileSize>50MB</maxFileSize>
        </rollingPolicy>
        <!-- 日志的格式化 -->
        <encoder>
            <!--  <pattern>${FILE_LOG_PATTERN}</pattern>-->
            <!--    FILE_LOG_PATTERN 是  Spring Boot 默认的 logback  配置 defaults.xml里定义的日志格式     -->
            <!--    默认的 pattern格式不会显示代码行号,这里使用自定义的pattern   -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%thread] %-40.40logger{39} :
                %line - : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}
            </pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>


    <!-- 设置 Appender ,日志的输出位置 -->
    <root level="info">
        <!--  控制台输出      -->
        <appender-ref ref="console"/>
        <!--  *.log文件输出      -->
        <appender-ref ref="file"/>

    </root>

</configuration>

```