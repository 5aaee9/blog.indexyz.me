---
title: 在 Spring Boot 中使用 Dubbo 以及 Kotlin 的坑
date: '2019-08-21T03:51:00+08:00'
categories:
- Java
---
最近写了点服务, 因为想尝试一下微服务的构架, 在项目中使用了 Dubbo 来做 RPC, 同时使用了 Kotlin 来简化语法(语法糖真甜)

<!--more-->
## Dubbo 序列化 Kotlin 的 Data Class 时需求 `java.io.Serializable`
这个坑比较简单, 首先我安装了 [kotlinx.serialization][1] 来序列化 kotlin 的 data class

将 Data Class 改成这样

```kotlin
import kotlinx.serialization.Serializable
import kotlinx.serialization.Polymorphic

@Serializable
data class Testing {
  val value: Map<String, @Polymorphic Any>
} : java.io.Serializable {}
```

这样 Dubbo 就可以序列化这个 Data Class 了

## Dubbo 反序列化 Data Class 回实例时报错
在默认情况下, Dubbo 使用修改过的 `hessian` 作为 `Serialization`

但是在反序列化 Data Class 时会报错 `com.alibaba.com.caucho.hessian.io.HessianProtocolException: 'com.example.TestingClass' could not be instantiated`

这个问题主要是由于 Data Class 没有无参数的构造器, 同时 Kotlin 在构造类时会对数据进行校验, 此时如果一个字段不能为 null 会丢错误出来. Hessian 会使用 `newInstance` 直接创建实例, 这时候校验就出错了.

解决方法也很简单, 将 `Serialization` 修改为不使用 `Hessian`, 此处修改成了 `fst`

在依赖中添加 `org.apache.dubbo:dubbo-serialization-fst:2.7.3`

然后在 `application.properties` 中添加 `dubbo.protocol.serialization=fst`

此时 Dubbo 传 Data Class 就能正确的反实例化了
## 传递懒加载的 Data Class 时报错 `class not found CLASSNAME`
例如在 MongoDB 的 Object 中开启 `@DBRef(lazy = true)`, 会创建一个代理实例, 只有在用到的时候才加载

如果直接输出对应的 class 的话 会看到此时的类后有 `EnhancerByCGLIB` 的标志, 我们只需要在返回时复制一个对象返回, 例如 `return t.obj.copy()` 就会复制一个没有代理过的实例返回, 就解决了这个问题

  [1]: https://github.com/Kotlin/kotlinx.serialization
