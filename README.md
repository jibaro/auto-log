# auto-log

[auto-log](https://github.com/houbb/auto-log) 是一款为 java 设计的自动日志监控框架。

[![Build Status](https://travis-ci.com/houbb/auto-log.svg?branch=master)](https://travis-ci.com/houbb/auto-log)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.houbb/auto-log/badge.svg)](http://mvnrepository.com/artifact/com.github.houbb/auto-log)
[![](https://img.shields.io/badge/license-Apache2-FF0080.svg)](https://github.com/houbb/auto-log/blob/master/LICENSE.txt)
[![Open Source Love](https://badges.frapsoft.com/os/v2/open-source.svg?v=103)](https://github.com/houbb/auto-log)

## 创作目的

经常会写一些工具，有时候手动加一些日志很麻烦，引入 spring 又过于大材小用。

所以希望从从简到繁实现一个工具，便于平时使用。

## 特性

- 基于注解+字节码，配置灵活

- 自动适配常见的日志框架

- 支持编程式的调用

- 支持注解式，完美整合 spring

- 支持整合 spring-boot

- 支持慢日志阈值指定，耗时，入参，出参，异常信息等常见属性指定

- 支持 traceId 特性

## 变更日志

- v0.0.9 变化

支持自定义日志输出策略

> [变更日志](https://github.com/houbb/auto-log/blob/master/CHANGELOG.md)

# 快速开始

## maven 引入

```xml
<dependency>
    <group>com.github.houbb</group>
    <artifact>auto-log-core</artifact>
    <version>${最新版本}</version>
</dependency>
```

## 入门案例

```java
UserService userService = AutoLogHelper.proxy(new UserServiceImpl());
userService.queryLog("1");
```

- 日志如下

```
[INFO] [2020-05-29 16:24:06.227] [main] [c.g.h.a.l.c.s.i.AutoLogMethodInterceptor.invoke] - public java.lang.String com.github.houbb.auto.log.test.service.impl.UserServiceImpl.queryLog(java.lang.String) param is [1]
[INFO] [2020-05-29 16:24:06.228] [main] [c.g.h.a.l.c.s.i.AutoLogMethodInterceptor.invoke] - public java.lang.String com.github.houbb.auto.log.test.service.impl.UserServiceImpl.queryLog(java.lang.String) result is result-1
```

### 代码

其中方法实现如下：

- UserService.java

```java
public interface UserService {

    String queryLog(final String id);

}
```

- UserServiceImpl.java

直接使用注解 `@AutoLog` 指定需要打日志的方法即可。

```java
public class UserServiceImpl implements UserService {

    @Override
    @AutoLog
    public String queryLog(String id) {
        return "result-"+id;
    }

}
```

## TraceId 的例子

### 代码

```java
UserService service =  AutoLogProxy.getProxy(new UserServiceImpl());
service.traceId("1");
```

其中 traceId 方法如下：

```java
@AutoLog
@TraceId
public String traceId(String id) {
    return id+"-1";
}
```

### 测试效果

```
信息: [ba7ddaded5a644e5a58fbd276b6657af] <traceId>入参: [1].
信息: [ba7ddaded5a644e5a58fbd276b6657af] <traceId>出参：1-1.
```

其中 ba7ddaded5a644e5a58fbd276b6657af 就是对应的 traceId，可以贯穿整个 thread 周期，便于我们日志查看。

# 注解说明

## @AutoLog

核心注解 `@AutoLog` 的属性说明如下：

| 属性 | 类型 | 默认值 | 说明 |
|:--|:--|:--|:--|
| param | boolean | true | 是否打印入参 |
| result | boolean | true | 是否打印出参 |
| costTime | boolean | false | 是否打印耗时 |
| exception | boolean | true | 是否打印异常 |
| slowThresholdMills | long | -1 | 当这个值大于等于 0 时，且耗时超过配置值，会输出慢日志 |
| description | string |"" | 方法描述，默认选择方法名称 |

## @TraceId

`@TraceId` 放在需要设置 traceId 的方法上，比如 Controller 层，mq 的消费者，rpc 请求的接受者等。

| 属性 | 类型 | 默认值 | 说明 |
|:--|:--|:--|:--|
| id | Class | 默认为 uuid | traceId 的实现策略 |
| putIfAbsent | boolean | false | 是否在当前线程没有值的时候才设置值 |

# spring 整合使用

完整示例参考 [SpringServiceTest](https://github.com/houbb/auto-log/tree/master/auto-log-test/src/test/java/com/github/houbb/auto/log/spring/SpringServiceTest.java)

## 注解声明

使用 `@EnableAutoLog` 启用自动日志输出

```java
@Configurable
@ComponentScan(basePackages = "com.github.houbb.auto.log.test.service")
@EnableAutoLog
public class SpringConfig {
}
```

## 测试代码

```java
@ContextConfiguration(classes = SpringConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringServiceTest {

    @Autowired
    private UserService userService;

    @Test
    public void queryLogTest() {
        userService.queryLog("1");
    }

}
```

- 输出结果

```
信息: public java.lang.String com.github.houbb.auto.log.test.service.impl.UserServiceImpl.queryLog(java.lang.String) param is [1]
五月 30, 2020 12:17:51 下午 com.github.houbb.auto.log.core.support.interceptor.AutoLogMethodInterceptor info
信息: public java.lang.String com.github.houbb.auto.log.test.service.impl.UserServiceImpl.queryLog(java.lang.String) result is result-1
五月 30, 2020 12:17:51 下午 org.springframework.context.support.GenericApplicationContext doClose
```

# springboot 整合使用

## maven 引入

```xml
<dependency>
    <groupId>com.github.houbb</groupId>
    <artifactId>auto-log-springboot-starter</artifactId>
    <version>最新版本</version>
</dependency>
```

只需要引入 jar 即可，其他的什么都不用配置。

使用方式和 spring 一致。

## 测试

```java
@Autowired
private UserService userService;

@Test
public void queryLogTest() {
    userService.query("spring-boot");
}
```

# 支持自定义日志输出策略

## 说明

有时候我们希望自定义日志策略，比如除了输出到日志文件，还可以保存到数据库做审计日志等。

## 说明

可以通过 `@EnableAutoLog` 中的 value() 属性指定多个实现策略的类，执行按照对应的指定类顺序。

最后方法的返回值以最后一个实现类为准。

## 案例

### 配置

```java
@Configurable
@ComponentScan(basePackages = "com.github.houbb.auto.log.test.service")
@EnableAutoLog(value = {MyAutoLog.class, SimpleAutoLog.class})
public class SpringConfig {
}
```

指定了 MyAutoLog（自定义策略） 和 SimpleAutoLog（默认策略） 策略。

### 自定义实现

```java
public class MyAutoLog implements IAutoLog {

    @Override
    public Object autoLog(IAutoLogContext context) throws Throwable {
        System.out.println("自定义参数：" + Arrays.toString(context.params()));
        Object result = context.process();
        System.out.println("自定义结果：" + result);
        return result;
    }

}
```

其他完全保持不变。

## 效果

```
自定义参数：[1]
自定义结果：result-1
九月 25, 2020 9:52:57 上午 com.github.houbb.auto.log.core.core.impl.SimpleAutoLog info
信息: <查询日志>入参: [1].
九月 25, 2020 9:52:57 上午 com.github.houbb.auto.log.core.core.impl.SimpleAutoLog info
信息: <查询日志>出参：result-1.
```

# 开源地址

> Github: [https://github.com/houbb/auto-log](https://github.com/houbb/auto-log)

> Gitee: [https://gitee.com/houbinbin/auto-log](https://gitee.com/houbinbin/auto-log)

# Road-Map

- [ ] 全局配置

比如全局的慢日志阈值设置等

- [ ] 支持类注解

获取方法时，获取对应的类。

- [ ] 注解特性拓展

是否可以支持自定义 handler

- [ ] jvm-sandbox 特性

- [ ] 编译时注解特性 