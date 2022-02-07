# zookeeper

## 介绍

![zNode](https://s4.ax1x.com/2022/02/07/HMty1U.png)

```
  Zookeeper 叫做分布式协调服务
Zookeeper 是一个 C/S 架构的服务，也就是 Client — Server 的形式。
在我们使用 Zookeeper 时，都是使用 Zookeeper 的客户端向服务端发送请求，然后由服务端做出响应返回到客户端。

Zookeeper 数据模型的结构为树形节点。Znode 的元素组成有 4 种：data（用户数据）、ACL（权限信息）、child（子节点引用）、stat（元数据）。Znode 的类型有 4 种：持久节点、持久顺序节点、临时节点、临时顺序节点。我们可以根据 Znode 的特点来实现各种功能，比如服务注册与发现、配置中心、分布式锁等.
Zookeeper 是 CP 系统，在保证数据一致性的同时，不能保证服务注册的高可用。
```

> zookeeper特点

```
在分布式环境下，Zookeeper 的部署方式为一主（ Leader ）多从（ Follower ）的集群方式，只要半数以上的节点（包括 Leader 节点）存活，Zookeeper 集群就能正常服务。就算是 Leader 节点挂掉了，Zookeeper 也会进行崩溃恢复，所说 Zookeeper 集群本身是高可用的；

Zookeeper 集群的数据具有全局一致性。也就是说，无论客户端连接到 Zookeeper 集群的哪一个从节点，获取的数据都是一致的；

在 Zookeeper 集群节点进行数据同步更新时，要么全部成功，要么全部失败。所以 Zookeeper 的数据更新具有原子性；

在同一个客户端对 Zookeeper 节点进行更新请求操作时，会根据发送的顺序依次去执行；

由于 Zookeeper 能存储的数据量非常小，所以数据的同步更新也会非常快。也就可以说在一定时间段内，客户端获取的数据是实时的
```

> zookeeper使用场景

```
服务注册与发现

当我们的分布式系统增加了一个服务，我们只需要利用 Znode 和 Watcher，让它注册到 Zookeeper 中，我们就可以很方便的对这个服务进行管理；

分布式锁

为了防止在分布式环境下，服务中多个进程之间互相干扰，我们可以用 Zookeeper 的临时顺序节点实现分布式锁，对这些进程进行调度，让它们顺序执行；

配置管理

我们可以把核心的配置文件交给 Zookeeper 管理。当我们修改配置文件时，Zookeeper 就会把配置文件的信息同步到集群中的所有节点中去

```

## 搭建zookeeper服务器

建议使用Docker安装
```
# 拉取 zookeeper 镜像
docker pull zookeeper

# run 启动，-d 后台运行，--name 别名，-p 端口映射（可以写多个）， 容器名称:版本（不写默认latest）
docker run -d --name=zookeeper -p 2181:2181 zookeeper


```

## zk可视化管理工具

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

## zk在第三方框架中的应用

除了单独使用 Zookeeper 来实现分布式锁、配置中心等功能外，
在我们使用一些优秀的框架时，比如高性能的分布式流处理平台 Apache Kafka，高性能的 Java RPC 框架 Apache Dubbo，它们也不同程度的依赖了 Zookeeper 服务。

### zk在kafka中的应用

```
Topic 配置管理： Topic 的配置会注册到 Zookeeper 中 的 config 节点下，根据 config 节点来动态更新配置；

Broker 管理： 在每个 Broker 启动时，都会注册到 Zookeeper 的 brokers 节点下；

Topic 及 Partition 管理： Topic 会注册到 brokers 节点下的 topics 节点下，Partition 会注册到 Topic 的节点下；

Producer 负载均衡： Producer 将消息发布到 Topic 时，会根据 Zookeeper 的 brokers 节点下的 Broker 来进行动态的负载均衡；

Consumer 负载均衡： Consumer 从 Topic 拉取消息时，同样也需要根据 Zookeeper 的 brokers 节点下的 Broker 来进行动态的负载均衡；

消费管理： 每个 Partition 只能被 Consumer Group 中的一个 Consumer 进行消费，因此需要关联 Partition 与 Consumer 之间的关系，将 Consumer 的 Consumer ID 注册到相关联的 Partition 节点的临时节点上；

Offset 记录： 在 Consumer 对指定 Partition 进行消息消费的过程中，需要将 Partition 的消费数量记录到 Zookeeper 中。
```

### zk在Dubbo中的应用

```
根据 Zookeeper 的树状结构，Dubbo 把节点分为 4 层

第一层为 Root 节点，固定为 dubbo；

第二层为服务节点，我们可以根据不同的服务来注册不同的服务节点，同时 Monitor 会监听服务节点；

第三层为服务的类型，分别为服务提供者和服务消费者，固定为 providers 和 consumers ；

第四层为服务提供者和服务消费者的 URL，如果是 Provider 就会注册到 providers 节点下的临时节点。如果是 Consumer 就注册到 consumers 节点下的临时节点，并且对同一服务下的 providers 开启监听
```

> Zookeeper 作为注册中心的原理

```
服务提供者把 URL 注册到 providers 下的临时节点，
服务消费者从 providers 获取 URL 列表，并对 providers 开启子节点事件的监听。
根据 Zookeeper 临时节点的特性，服务提供者只要与 Zookeeper 断开连接，Zookeeper 服务就会把该临时节点移除，此时服务消费者就会收到 providers 的子节点移除事件，然后更新自己的服务提供者的 URL 列表
```

