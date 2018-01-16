---
layout: post
title: Useful Utils and Tricks in Spring Framework
tags: 
- Java
- Spring
---





# Useful Utils and Tricks in Spring Framework

这边总结一下在Spring中比较有意思的工具或扩展，Spring并不只有Ioc框架和Web框架，除此之外，还有一些比较有用的工具或扩展，这些本身应该是实现Spring核心功能所设计和封装的类，但由于良好的抽象和封装设计，使得这些类也能在一些平时开发中单独使用上，或许这些微不足道的工具正好能够解决一些细微又苦于实现的痛点。

## `ConversionService`

`package org.springframework.core.convert.support`

这是一个类型转换的服务类，提供如下的接口：

```
<T> T convert(Object source, Class<T> targetType);
```

其实现是将向`ConversionService`注册多个一个个具体负责由类型A转换为类型B的`Converter`，最后汇总而成，因此，也可以自定义实现`Converter`并添加到`ConversionService`，Spring本身提供了默认实现`DefaultConversionService`，可以注册到容器中作为线程安全的单例使用，也可直接new出来

原本是用于解析spring-context.xml文件中bean的property部分，将property的value值字符串转换为注入目标的类型。因此，这个也可直接用在平时开发中的入参解析转换中，比如`String`转为`int`，`String`转为`Enum`，还有各种奇怪的`Array`转为`List`，`String`转为`Date`， `UUID`等等，几乎所有基本类型转换的情况都囊括进去，不用再自己去写XXXUtils之类了，业务搬砖开发遇到需要转换类型的时候不妨先考虑一下这个，不用太担心性能。

唯一的问题是，如果转换失败，会**直接抛异常**，实际使用时还是比较难看。

下面是使用实例

```java
public void example() {
        ConversionService  converter = new DefaultConversionService();
        int three = converter.convert("1", Integer.class) + converter.convert("2", Integer.class);
        List<String> stringList = converter.convert(new String[] {"a", "b"}, List.class);
    }
```



## `StopWatch`

`package org.springframework.util;`

这是一个计时器，一般用于平时开发中计算方法的调用耗时，相比于CSDN博客中烂大街的`System.currentTimeMillis()`，这个提供更OO的操作，可统计多个串行的任务时间，但不支持多线程并发的计算时间操作。

以下是使用实例

```java
public void example() throws InterruptedException {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start("task1");
        Thread.sleep(1000);
        // blablabla
        stopWatch.stop();
        stopWatch.start("task2");
        Thread.sleep(2000);
        // blabla
        stopWatch.stop();
        stopWatch.getTotalTimeMillis();
        System.out.println(stopWatch.prettyPrint());
}
```



## `UriTemplate`

`package org.springframework.web.util`

这个专门用于拼接url的辅助类，对于一些Resutful URL的接口，常常需要使用一些参数值根据url格式生成实际的url链接，并符合URL编码规范，使用这个可以避免繁杂的字符串拼接，更原始的URL拼接可以使用`UriComponentsBuilder`

以下是使用实例

```java
UriTemplate template = new UriTemplate("http://example.com/hotels/{hotel}/bookings/{booking}");
Map<String, String> uriVariables = new HashMap<String, String>();
uriVariables.put("booking", "42");
uriVariables.put("hotel", "Rest & Relax");
System.out.println(template.expand(uriVariables));
// Will print: http://example.com/hotels/Rest%20%26%20Relax/bookings/42
```



## `RestTemplate`

`package org.springframework.web.client`

这个相当于一个Restful HTTP Client，可以配置为bean使用，类似于Python中的[requests](http://docs.python-requests.org/en/master/)，但如果需要支持JSON数据消息读取的话还需要做一定配置(看到之前的Converter没有？)， 因此如果需要用的话，似乎[okHttp](http://square.github.io/okhttp/)是一个更好的选择，当然，Spring的设计者都考虑到了，通过方法`setRequestFactory`可以让该类使用okHttp的内核而保持原来的接口。异步调用还可以使用扩展类`AsyncRestTemplate`



## `BeanUtils`

`package org.springframework.beans;`

为Java Bean提供的反射编辑工具类，例如动态编辑bean属性，实例化bean之类的，相信很多都用过



## `AnnotationUtils`

`package org.springframework.core.annotation`

为Java注解解析提供的反射编辑类，例如获取类上的注解，获取父类已经标记的注解，获取被元注解标记的注解（例如@Service本身被@Component标记，当搜索扫描时也会搜索注解为@Service的bean），内容繁多，可到具体需要的时候再阅读doc。



## `HtmlUtils`

html字符串转义工具，根据文档描述，其实现完全符合HTML 4.0标准。



## `WebUtils`

针对Web 环境的工具，例如获取cookie，上下文路径等等。



## `AntPathMatcher`

匹配Ant风格路径工具类