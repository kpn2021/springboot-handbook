# zookeeper

> 介绍

```
Zookeeper 是一个 C/S 架构的服务，也就是 Client — Server 的形式。
在我们使用 Zookeeper 时，都是使用 Zookeeper 的客户端向服务端发送请求，然后由服务端做出响应返回到客户端。
在这个过程中，Zookeeper 的客户端需要与 Zookeeper 服务端建立连接，
建立一个连接就是新建一个会话，那么会话的状态也就是 Zookeeper 客户端与 Zookeeper 服务端的连接状态
```

> 搭建zookeeper服务器

建议使用Docker安装
```
# 拉取 zookeeper 镜像
docker pull zookeeper

# run 启动，-d 后台运行，--name 别名，-p 端口映射（可以写多个）， 容器名称:版本（不写默认latest）
docker run -d --name=zookeeper -p 2181:2181 zookeeper


```

> zk可视化管理工具

建议使用PrettyZoo，[下载地址](https://www.oschina.net/p/prettyzoo)

![prettyZoo](https://s4.ax1x.com/2022/02/07/HMnopn.png)

## Curator
```
Curator 是 Netflix 公司开源的一套 Zookeeper 客户端框架,
Curator 提供了一套易用性和可读性更强的 Fluent 风格的客户端 API ，
还提供了 Zookeeper 各种应用场景的抽象封装，
比如：分布式锁服务、集群领导选举、共享计数器、缓存机制、分布式队列等
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
# 命名空间，当前客户端的父节点,若zkServer没有imooc这个Node，则本项目启动时会创建这个Node,这个Node就是namespace
curator.namespace=imooc
# Tips： 使用 curator 时，我们需要注意是否配置 namespace ，
# 如果没有配置 namespace 的话，我们使用 curator 进行操作时，path 参数需要填写全路径。
# 如果配置了 namespace ，我们使用 curator 时，Curator 会自动帮我们在 path 前加上 namespace
```

## 让ZkClient随着SpringBoot启动而启动的用法

```java
package com.example.test22;

import java.nio.charset.StandardCharsets;
import java.util.List;
import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.imps.CuratorFrameworkState;
import org.apache.curator.framework.recipes.cache.CuratorCache;
import org.apache.curator.framework.recipes.cache.CuratorCacheListener;
import org.apache.curator.retry.*;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
public class MyCuratorUtil implements CommandLineRunner {
    // Zookeeper 服务器地址
    @Value("${curator.connectString}")
    private String           connectString;
    // session 会话超时时间
    @Value("${curator.sessionTimeoutMs}")
    private int              sessionTimeoutMs;
    // 名称空间：在操作节点时，会以 namespace 为父节点
    @Value("${curator.namespace}")
    private String           namespace;

    /**
     * session 重连策略，使用其中一种即可
     */
    // RetryForever：间隔{参数1}毫秒后重连，永远重试
    private RetryPolicy      retryForever      = new RetryForever(3000);

    // RetryOneTime：{参数1}毫秒后重连，只重连一次
    private RetryPolicy      retryOneTime      = new RetryOneTime(3000);

    // RetryNTimes： {参数2}毫秒后重连，重连{参数1}次
    private RetryPolicy      retryNTimes       = new RetryNTimes(3, 3000);

    // RetryUntilElapsed：每{参数2}毫秒重连一次，总等待时间超过{参数1}毫秒后停止重连
    private RetryPolicy      retryUntilElapsed = new RetryUntilElapsed(10000, 3000);

    // ExponentialBackoffRetry：可重连{参数2}次，并增加每次重连之间的睡眠时间，增加公式如下：
    // {参数1} * Math.max(1,random.nextInt(1 << ({参数2：maxRetries} + 1)))
    private RetryPolicy      exponential       = new ExponentialBackoffRetry(1000, 3);

    private CuratorFramework client            = null;

    private CuratorCache     cache3            = null;

    private void initZk() {
        // 使用 CuratorFrameworkFactory 来构建 CuratorFramework
        client = CuratorFrameworkFactory.builder()
            // Zookeeper 服务器地址字符串
            .connectString(connectString)
            // session 会话超时时间
            .sessionTimeoutMs(sessionTimeoutMs)
            // 使用哪种重连策略
            .retryPolicy(retryOneTime)
            // 配置父节点
            .namespace(namespace).build();

        // 开启会话
        client.start();
        log.warn("zk Client已启动!");
        // 构建 CuratorCache 实例
        cache3 = CuratorCache.build(client, "/");
        // 使用 Fluent 风格和 lambda 表达式来构建 CuratorCacheListener 的事件监听
        CuratorCacheListener listener = CuratorCacheListener.builder()
            // 开启对所有事件的监听
            // type 事件类型：NODE_CREATED, NODE_CHANGED, NODE_DELETED;
            // oldNode 原节点：ChildData 类，包括节点路径，节点状态 Stat，节点 data
            // newNode 新节点：同上
            .forAll((type, oldNode, newNode) -> {
                log.warn(" 事件类型：{}", type);
                log.warn(" 原节点：{}", oldNode);
                log.warn(" 新节点：{}", newNode);
            })
            //            // 开启对节点创建事件的监听
            //            .forCreates(childData -> {
            //
            //                log.warn("创建了新节点：{}", childData);
            //
            //            })
            //            //             开启对节点更新事件的监听
            //            .forChanges((oldNode, newNode) -> {
            //                log.warn("forChanges 节点原始内容：{}", oldNode);
            //                log.warn("forChanges 节点更新后：{}", newNode);
            //            })
            //            // 开启对节点删除事件的监听
            //            .forDeletes(oldNode -> {
            //                log.warn("删除了结点:{}", oldNode);
            //            })
            // 初始化
            .forInitialized(() -> {
                log.warn("CuratorCacheListener初始化完毕");
            })
            // 构建
            .build();

        // 注册 CuratorCacheListener 到 CuratorCache
        cache3.listenable().addListener(listener);
        // CuratorCache 开启缓存
        cache3.start();
        log.warn("zk CuratorCache已启动!");
        log.warn("zk state:{}", client.getState());

    }

    @Override
    public void run(String... args) throws Exception {
        initZk();
    }

    /**
     * 父节点自动掉了，要在下次操作时恢复，即重新建立带zWatch 的 zkClient
     */
    private void refreshZkClient() {
        if (client.getState() != CuratorFrameworkState.STOPPED) {
            cache3.close();
            log.warn("zk CuratorCache 已关闭!");
            client.close();
            log.warn("zk Client 已关闭!");
        }
        initZk();
        log.warn("refreshZkClient---Completed");
    }

    public List<String> getAllNodeByPath(String path) throws Exception {
        refreshZkClient();
        return client.getChildren().forPath(path);
    }

    public void createNormalNode(String path) throws Exception {
        refreshZkClient();
        client.create().forPath(path);
    }

    public void createNodeAndData(String path, String dataStr) throws Exception {
        refreshZkClient();
        client.create().forPath(path, dataStr.getBytes(StandardCharsets.UTF_8));
    }

    public void updateNodeData(String path, String dataStr) throws Exception {
        refreshZkClient();
        client.setData().forPath(path, dataStr.getBytes(StandardCharsets.UTF_8));
    }

    public void deleteNode(String path) throws Exception {
        refreshZkClient();
        client.delete().forPath(path);
    }
}

```

## 测试zk的使用

```java
    @Autowired
    private MyCuratorUtil myCuratorUtil;

    @GetMapping("/testCreate")
    public String testApi() throws Exception {
        myCuratorUtil.createNormalNode("/mooc");
        return testcfg;
    }

    @GetMapping("/findNodeList")
    public String findNodeList() throws Exception {
        // 查询命名空间下的子节点
        List<String> strings = myCuratorUtil.getAllNodeByPath("/t1");
        return String.valueOf(strings);
    }

    @PostMapping("/testCreate2")
    public String testCreate2() throws Exception {
        myCuratorUtil.createNodeAndData("/mc3", "hello123");

        return testcfg;
    }

    @PutMapping("/testUpdate")
    public String testUpdate() throws Exception {
        myCuratorUtil.updateNodeData("/mooc", "Wiki");
        return testcfg;
    }

    @DeleteMapping("/testDel")
    public String testDel() throws Exception {

        //父节点的最后一个子节点-delete后，zk会自动把父节点也删掉
        myCuratorUtil.deleteNode("/mooc");
        return testcfg;
    }
```