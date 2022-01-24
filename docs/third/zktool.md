# Curator

```
Curator 是 Netflix 公司开源的一套 Zookeeper 客户端框架,Curator 提供了一套易用性和可读性更强的 Fluent 风格的客户端 API ，还提供了 Zookeeper 各种应用场景的抽象封装，比如：分布式锁服务、集群领导选举、共享计数器、缓存机制、分布式队列等
```

## POM配置

```xml
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>5.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>5.1.0</version>
        </dependency>   
```

## properties配置

```properties
# Zookeeper 地址
#curator.connectString=192.168.0.77:2181,192.168.0.88:2181,192.168.0.88:2181
curator.connectString=127.0.0.1:2181
# 会话超时时间
curator.sessionTimeoutMs=5000
# 命名空间，当前客户端的父节点,若没有imooc这个Node，则本项目启动时会自动创建这个Node
curator.namespace=imooc
```

## 在CommandLineRunner中打印imooc这个Node下的所有结点

```java
package com.example.test22;

import java.util.List;
import org.apache.curator.framework.CuratorFramework;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
public class TestRunner implements CommandLineRunner {

    @Autowired
    private CuratorService curatorService;

    @Override
    public void run(String... args) throws Exception {
        // 获取客户端
        CuratorFramework curatorClient = curatorService.getCuratorClient();
        // 开启会话
        curatorClient.start();
        // 查询命名空间下的子节点
        List<String> strings = curatorClient.getChildren().forPath("/");
        log.warn(String.valueOf(strings));
        curatorClient.close();
    }
}

```
