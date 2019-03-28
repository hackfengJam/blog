# Eureka Server 和 Client 之间的信息维护（注册和续约） 


### Eureka Server 维护了系统中服务的元信息，这些元信息包含什么你知道吗 ？

#### Eureka Server 维护了 Client 的哪些信息呢？

如果我们启动 Client （简单认为就是一个微服务）的时候，如果此时配置了错误的 Eureka Server 发现地址或 Eureka Server 没有启动，那么会发现，Client 可以正常启动，但不能完成注册，会抛出如下异常信息。

```
com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server
  ......
    at com.netflix.discovery.DiscoveryClient.register(DiscoveryClient.java:829) ~[eureka-client-1.9.2.jar:1.9.2]
  ......
```

这里的异常信息，其实就告诉了我们，Client 向 Eureka Server 发起注册的过程。我们来看一看 DiscoveryClient.register 里面做了什么（eureka-client-1.9.2-sources.jar!/com/netflix/discovery/DiscoveryClient.java）:

```java
/**
 * 通过 HTTP （eurekaTransport） 请求将 Eureka Client 注册到 Eureka Server 上
 */
boolean register() throws Throwable {
    logger.info(PREFIX + "{}: registering service...", appPathIdentifier);
    EurekaHttpResponse<Void> httpResponse;
    try {
        // 很显然，这里的 instanceInfo 就是 Client 的元数据
        httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
    } catch (Exception e) {
        logger.warn(PREFIX + "{} - registration failed {}", appPathIdentifier, e.getMessage(), e);
        throw e;
    }
    if (logger.isInfoEnabled()) {
        logger.info(PREFIX + "{} - registration status: {}", appPathIdentifier, httpResponse.getStatusCode());
    }
    return httpResponse.getStatusCode() == 204;
}
```

instanceInfo 就是 com.netflix.appinfo.InstanceInfo 的对象实例，它被称作是"应用实例信息"。
所以，这个对象中包含的属性，最终会投递到 Eureka Server 中，也就是说，Eureka Server 维护的元信息就是 InstanceInfo（各位可以看一看 InstanceInfo 中包含了什么，同时，还可以继续跟踪源代码，看一看 InstanceInfo 的初始化过程）。


### 元信息又是怎么存储的呢 ？

#### 猜想，Eureka Server 和 Client 之间的信息维护（注册和续约） 

元信息的保存，可能是Eureka Server内部维护着一张表来记录这些元信息，Eureka服务器没有后端存储，但注册表中的服务实例必须发送心跳信号以保持其注册是最新的，所以这可能使内存中完成。客户端可能还拥有一个eureka注册的内存缓存，这样，client不必为每个服务请求都去注册表。

#### Eureka Server 是怎么存储 Client 的元信息的呢 ？

这个问题与第一个问题是有关联性的，因为 Client 向 Server 发起注册（HTTP 请求），那么 Server 必须对外提供了这个接口才可以。

经过追踪 Server 端的源码（eureka-core-1.9.2-sources.jar）,很容易可以找到 com/netflix/eureka/resources/ApplicationResource.java，
其中 addInstance 方法即实现了服务注册的功能（Server 端）。源码如下：

```java
@POST
@Consumes({"application/json", "application/xml"})
public Response addInstance(InstanceInfo info,
                            @HeaderParam(PeerEurekaNode.HEADER_REPLICATION) String isReplication) {
    // 校验 instanceinfo 中是否包含必须的属性
    if (isBlank(info.getId())) {
        return Response.status(400).entity("Missing instanceId").build();
    } else if (isBlank(info.getHostName())) {
        return Response.status(400).entity("Missing hostname").build();
    }
    ......
 
    // 注册信息校验
    ......
 
    // 通过 PeerAwareInstanceRegistry 的 register 方法完成 Client 的注册
    registry.register(info, "true".equals(isReplication));
    return Response.status(204).build();
}
```

我们再去看 PeerAwareInstanceRegistry 的实现类：org.springframework.cloud.netflix.eureka.server.InstanceRegistry 的 register 方法的实现：

```java
public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
    try {
        read.lock();
        //  registry 存储了注册信息, 它是一个两层的 Map 结构
        // ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry
        Map<String, Lease<InstanceInfo>> gMap = registry.get(registrant.getAppName());
        REGISTER.increment(isReplication);
        if (gMap == null) {
            final ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap<String, Lease<InstanceInfo>>();
            gMap = registry.putIfAbsent(registrant.getAppName(), gNewMap);
            if (gMap == null) {
                gMap = gNewMap;
            }
        }
        Lease<InstanceInfo> existingLease = gMap.get(registrant.getId());
        ......
        Lease<InstanceInfo> lease = new Lease<InstanceInfo>(registrant, leaseDuration);
        if (existingLease != null) {
            lease.setServiceUpTimestamp(existingLease.getServiceUpTimestamp());
        }
        gMap.put(registrant.getId(), lease);
        ......
    } finally {
        read.unlock();
    }
}
```

InstanceInfo 中的元数据信息存储在一个 ConcurrentHashMap 对象中。Eureka Server 使用了两层 Map 结构做存储，第一层的 key 存储服务名：
InstanceInfo 中的 appName 属性，第二层 key 存储实例名：InstanceInfo 中的 instanceId 属性。


## 感谢

[张勤一 - 百度 · 高级Java工程师](https://www.zhihu.com/people/zhang-qin-yi-51-72/activities)

