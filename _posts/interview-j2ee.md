面试-网易

- 列举Java和Python的一些重要特性： 

  Java：同步互斥、内存模型、NIO 

  Python：generator, yield

- Java: finally 块是否一定被执行，（不抛异常、抛异常、中间return）

- 进程间同步的手段？

- Spring： IoC概念、思想，AOP思想、原理；Spring MVC主要原理和特性

- Hibernate 和 Mybaits的异同和优缺点， ORM？

- 说明事务隔离的4种级别

- 列举常用的MySQL SQL和表优化方法

- 使用Redis进行缓存的经验（结合项目）

- 怎么解决更新丢失的问题（一个计算器，每次加1，到10的倍数时发起抽奖活动业务，如何避免多个线程（更进一步，多个进程）下，抽奖号出现重复的问题。









Java EE

## Servlet





### Spring -- DispatcherServlet

- MVC中C的主要实现，统一接受所有请求（在Web.xml中定义的路径，一般是\）
- 功能：通过HandlerMapping，将请求映射到处理器
- 通过ViewResolver解析逻辑视图名到具体视图实现
- 渲染具体的视图



## JDBC

### JDBC编程的步骤 

1. 加载数据库驱动 : 每种数据库的驱动程序都不一致，在JDBC规范中明确要求Driver(数据库驱动)类必须向DriverManager注册，故需要动态加载。
2. 创建并获取数据库链接 
3. 创建jdbc statement对象 
4. 设置sql语句 
5. 设置sql语句中的参数(使用preparedStatement) 
6. 通过statement执行sql并获取结果 
7. 对sql执行结果进行解析处理 
8. 释放资源(resultSet、preparedstatement、connection)



## Prepared Statement

- 性能：能够减少执行时间，会被预先编译，（能生成语法树，树的结构和节点的具体值无关）
- 资源：复用SQL语句
- 安全：代替字符串相加操作，减小SQL注入攻击
- 实现：在MySQL5.7 中，Prepared Statement 会保存在Query缓存中，提高性能。在缓存时，必须保证所用的query和缓存中存在字符上完全一致的语句才能被命中，**也即不忽略大小写**。

[Reference]

- https://dev.mysql.com/doc/refman/5.7/en/query-cache-operation.html



## JDBC编程问题

缺点：

- 每次创建连接消耗资源，连接是一个tcp连接。
- SQL硬编码，以String表示，不易维护
- 重复的创建链接、准备语句、封装结果、关闭连接的代码逻辑片段。
- prepareStatement占位符不安全，难以检查。
- 结果集为原始类型没有进行业务封装。
- 编程式的事务不易维护。



## 数据库持久层框架特性综述

### 线程池





JavaBean封装







## 自我介绍

- 我叫xxx，来自xxx，本科毕业于xxxx，现也于xxx大学xxx专业研究生就读，。

- 自大三开始，在企业中实习，

- 第一份实习工作 ： 原型？ 历史期货，在线策略，效益。负责整个原型，从架构到设计、编码，可实际运行的原型。

  策略代码：Classloader

- 第二份工作，时：大四，地点：在上海百度，部门：搜索测试部，职位：测试开发，负责JavaEE，推广业务系统测试，项目管理、迭代进度、功能测试、（业务、代码）评估风险、业务、性能监控，保证上线效果。而且得到了转正的机会。

  利用Spring 扩展，

  自定义注解，json测试数据注入，开发出基于Spring的命令行程序框架

  给予次框架的物料自动上线验证工具，方便人工测试

- 目前SAP，Node 南京交通大数据可视化平台。

- 在大学期间还参加过团队。

- 平时会写博客，关注一些信息、技术，最近研究 leveldb实现、和Node的核心功能。

  ​