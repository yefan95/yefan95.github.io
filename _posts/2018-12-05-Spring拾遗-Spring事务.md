---
layout:     post
title:      "Spring拾遗-Spring事务"
subtitle:   " \"Spring事务管理方式\""
date:       2018-12-05 14:19:00
author:     "Zero"
header-img: "img/post-spring-bg-2018.jpeg"
tags:
    - Spring
---

## 什么是事务?

事务指的是逻辑上的一组操作，这组操作要么全部成功，要么全部失败。

## 事务的四大特性

- 原子性 

  原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。

- 一致性

  一致性指事务前后数据的完整性必须保持一致。

- 隔离性

  隔离性指多个用户并发访问数据库时，一个用户的事务不能被其他用户的事务所干扰，多个并发事务之间数据要相互隔离。

- 持久性

  持久性是指一个事务一旦被提交提交，它对数据库中数据的改变就是永久性的，即使数据库发生故障也不应该对其有任何影响。



## Spring事务管理

### Spring事务接口API

Spring事务管理高层抽象主要包括3个接口

- PlatformTransactionManager

  事务管理器

  ![image-20181205112759849](https://yefan95.github.io/img/spring-transaction/image-20181205112759849.png)

- TransactionDefintion

  事务定义信息（隔离、传播、超时、只读）

  ![image-20181205112947504](https://yefan95.github.io/img/spring-transaction/image-20181205112947504.png)

- TransactionStatus

  事务具体运行状态

  ![image-20181205113244536](https://yefan95.github.io/img/spring-transaction/image-20181205113244536.png)



#### 事务管理器PlatformTransactionManager

##### Spring API提供的实现

Spring的api提供了一些对PlatformTransactionManager的实现，如下图所示：

![image-20181205113852723](https://yefan95.github.io/img/spring-transaction/image-20181205113735074.png)

Spring为不同的持久化框架提供了不同PlatformTransactionManager接口实现

| 事务                                                         | 说明                                                        |
| :----------------------------------------------------------- | ----------------------------------------------------------- |
| org.springframework.jdbc.datasource.DataSourceTransactionManager | 使用JDBC或IBatis进行持久化数据时使用                        |
| org.springframework.orm.hibernate3.HibernateTransactionManager | 使用Hibernate3.0版本持久化数据时使用                        |
| org.springframework.jpa.JpaTransactionManager                | 使用JPA持久化时使用                                         |
| org.springframework.jdo.JdoTransactionManager                | 使用持久化机制是Jdo时使用                                   |
| org.springframework.transaction.jta.JtaTransactionManager    | 使用一个JTA实现来管理事务，在一个事务跨越多个资源时必须使用 |
|                                                              |                                                             |

#### 事务定义TransactionDefintion

Spring的api提供了一些对TransactionDefintion的实现，如下图所示：

![image-20181205115441261](https://yefan95.github.io/img/spring-transaction/image-20181205113852723.png)

TransactionDefintion的隔离级别参数，如下图所示：

![image-20181205115550112](https://yefan95.github.io/img/spring-transaction/image-20181205115441261.png)



TransactionDefintion的方法，如下图所示：

![image-20181205115703531](https://yefan95.github.io/img/spring-transaction/image-20181205115703531.png)



#### 事务状态TransactionStatus

Spring的api提供了一些对TransactionStatus的实现，如下图所示：

![image-20181205120207869](https://yefan95.github.io/img/spring-transaction/image-20181205120207869.png)

TransactionStatus的方法，如下图所示：

![image-20181205120406267](https://yefan95.github.io/img/spring-transaction/image-20181205120406267.png)

### Spring事务的隔离级别

| 隔离级别        | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| DEFAULT         | 使用后端数据库默认的隔离级别(spring中的选择项)               |
| READ_UNCOMMITED | 允许你读取还未提交的改变了的数据。可能导致脏、幻、不可重复读 |
| REPEATABLE_READ | 允许在并发事务已经提交后读取，可防止脏读，但幻读、不可重复读仍发生 |
| SERIALIZABLE    | 完全服从ACID的隔离级别，确保不发生脏读、幻读、不可重复读。在所有的隔离级别中是最慢的，它是典型的通过完全锁定在事务中涉及的数据表来完成。 |
|                 |                                                              |

### Spring事务的传播行为

| 事务传播类型              | 说明                                           |
| ------------------------- | ---------------------------------------------- |
| PROPAGATION_REQUIRED      | 支持当前事务，如果不存在就新建一个             |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果不存在，就不使用事务         |
| PROPAGATION_MANDATORY     | 支持当前事务，如果不存在，就抛出异常           |
| PROPAGATION_REQUIRES_NEW  | 如果有事务存在，挂起当前事务，创建一个新的事务 |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式运行，如果有事务存在，挂起当前事务 |
| PROPAGATION_NEVER         | 以非事务方式运行，如果有事务，抛出异常         |
| PROPAGATION_NESTED        | 如果当前事务存在，则嵌套事务执行               |
|                           |                                                |

## Spring事务实现方式

### 业务场景

#### 业务描述

该例子主要简单的实现两个账户之间的转账流程，从张三的账户中转出200元到李四的账户中

#### 业务流程

- 开始转账

- 从张三的账户余额减少200
- 李四的账户余额增加200
- 转账完成

#### 环境准备

- 创建数据库

  `CREATE DATABASE spring-transaction`

- 创建数据表

  ```
  CREATE TABLE  `account`(
     `id` int(11) NOT NULL AUTO_INCREMENT,
     `name` VARCHAR(11) NOT NULL,
     `money` DOUBLE DEFAULT NULL,
      PRIMARY KEY (`id`)
  );
  
  INSERT INTO `account` VALUES ('1','aaa','1000');
  INSERT INTO `account` VALUES ('2','bbb','1000');
  INSERT INTO `account` VALUES ('3','ccc','1000');
  ```

- 创建Maven工程

  使用IDEA提供的maven工程模板

- 添加pom文件依赖

  ```
  <dependencies>
      <!-- 测试依赖 -->
      <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>4.12</version>
          <scope>test</scope>
      </dependency>
      <!-- Spring相关依赖 -->
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-core</artifactId>
          <version>4.3.16.RELEASE</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>4.3.16.RELEASE</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-aop</artifactId>
          <version>4.3.16.RELEASE</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-beans</artifactId>
          <version>4.3.16.RELEASE</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-aspects</artifactId>
          <version>4.3.16.RELEASE</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-jdbc</artifactId>
          <version>4.3.16.RELEASE</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-test</artifactId>
          <version>4.3.16.RELEASE</version>
      </dependency>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-tx</artifactId>
          <version>4.3.16.RELEASE</version>
      </dependency>
      <!-- 切面依赖 -->
      <dependency>
          <groupId>org.aspectj</groupId>
          <artifactId>aspectjweaver</artifactId>
          <version>1.8.13</version>
      </dependency>
      <!-- 日志相关依赖 -->
      <dependency>
          <groupId>org.apache.logging.log4j</groupId>
          <artifactId>log4j</artifactId>
          <version>2.11.1</version>
      </dependency>
      <!-- 连接池依赖 -->
      <dependency>
          <groupId>com.mchange</groupId>
          <artifactId>c3p0</artifactId>
          <version>0.9.5.2</version>
      </dependency>
      <!-- 数据库驱动依赖 -->
      <dependency>
          <groupId>mysql</groupId>
          <artifactId>mysql-connector-java</artifactId>
          <version>5.1.46</version>
      </dependency>
  </dependencies>
  ```

- JDBC配置

```
jdbc.driverClass=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/spring-transaction?useSSL=false
jdbc.username=root
jdbc.password=root
```

- log4j配置

```
log4j.rootLogger=INFO,logfile,stdout
#log4j.logger.org.springframework.web.servlet=INFO,db
#log4j.logger.org.springframework.beans.factory.xml=INFO
#log4j.logger.com.neam.stum.user=INFO,db
#log4j.appender.stdout=org.apache.log4j.ConsoleAppender
#log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
#log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH\:mm\:ss} %p [%c] %X{remoteAddr}  %X{remotePort}  %X{remoteHost}  %X{remoteUser} operator\:[\u59D3\u540D\:%X{userName} \u5DE5\u53F7\:%X{userId}] message\:<%m>%n
#write log into file
log4j.appender.logfile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.logfile.Threshold=warn
log4j.appender.logfile.File=${webapp.root}\\logs\\Spring-transaction.log
log4j.appender.logfile.DatePattern=.yyyy-MM-dd
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=[SpringTransaction] %d{yyyy-MM-dd HH\:mm\:ss} %X{remoteAddr} %X{remotePort} %m %n
#display in console
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Threshold=info
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=[SpringTransaction] %d{yyyy-MM-dd HH\:mm\:ss} %X{remoteAddr} %X{remotePort} %m %n
```

- 业务DAO层

转账案例DAO接口

```
/**
 * @author yefan
 * @date 2018/12/05
 * <p>
 * 转账案例的DAO层接口
 */
public interface AccountDao {
    /**
     * @param out   转出金额
     * @param money 转账金额
     */
    void outMoney(String out, Double money);

    /**
     * @param in    转入金额
     * @param money 转账金额
     */
    void intMoney(String in, Double money);

}
```

转账案例DAO接口实现类

```
/**
 * @author yefan
 * @date 2018/12/05
 * <p>
 * 转账案例的DAO层接口实现类
 */
public class AccountDaoImpl extends JdbcDaoSupport implements AccountDao {


    @Override
    public void outMoney(String out, Double money) {
        String sql = "update account set money = money - ? where name = ?";
        this.getJdbcTemplate().update(sql, money, out);
    }

    @Override
    public void intMoney(String in, Double money) {
        String sql = "update account set money = money + ? where name = ?";
        this.getJdbcTemplate().update(sql, money, in);
    }
}
```

- 业务Serviec层

  ```
  /**
   * @author yefan
   * @date 2018/12/05
   * <p>
   * 转账案例的业务接口
   */
  public interface AccountService {
  
      /**
       * @param out   转出账号
       * @param in    转入账号
       * @param money 转账金额
       */
      void transfer(String out, String in, Double money);
  
  }
  ```



### 编程式事务管理

- Service实现

```
/**
 * @author yefan
 * @date 2018/12/05
 * <p>
 * 转账案例的业务接口实现类
 */
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    private TransactionTemplate transactionTemplate;

    @Override
    public void transfer(String out, String in, Double money) {
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {
                accountDao.outMoney(out, money);
//                int a = 2 / 0;
                accountDao.intMoney(in, money);
            }
        });
    }

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    public void setTransactionTemplate(TransactionTemplate transactionTemplate) {
        this.transactionTemplate = transactionTemplate;
    }
}
```



- springApplication配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context-3.1.xsd">

    <!-- 引入外部的属性文件 -->
    <context:property-placeholder location="classpath:config/jdbc.properties"/>

    <!-- 配置c3p0连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driverClass}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!-- 配置业务层类 -->
    <bean id="accountService" class="com.yefan.study.transaction.demo1.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <!-- 注入事务管理的模板 -->
        <property name="transactionTemplate" ref="transactionTemplate"/>
    </bean>

    <!-- 配置DAO类(简化，会自动配置JdbcTemplate) -->
    <bean id="accountDao" class="com.yefan.study.transaction.demo1.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置DAO类(未简化) -->
    <!-- <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="accountDao" class="com.zs.spring.demo1.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate" />
    </bean> -->

    <!-- ==================================1.编程式的事务管理=============================================== -->
    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置事务管理的模板:Spring为了简化事务管理的代码而提供的类 -->
    <bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
        <property name="transactionManager" ref="transactionManager"/>
    </bean>

</beans>
```

### 声明式事务管理



#### 基于TransactionProxyFactoryBean的方式

- Service实现

```
/**
 * @author yefan
 * @date 2018/12/05
 */
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    @Override
    public void transfer(String out, String in, Double money) {
        accountDao.outMoney(out, money);
//                int a = 2 / 0;
        accountDao.intMoney(in, money);
    }

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }
}
```



- springApplication配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context-3.1.xsd">

    <!-- 引入外部的属性文件 -->
    <context:property-placeholder location="classpath:config/jdbc.properties"/>

    <!-- 配置c3p0连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driverClass}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!-- 配置业务层类 -->
    <bean id="accountService" class="com.yefan.study.transaction.demo2.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>

    <!-- 配置DAO类(简化，会自动配置JdbcTemplate) -->
    <bean id="accountDao" class="com.yefan.study.transaction.demo2.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置DAO类(未简化) -->
    <!-- <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="accountDao" class="com.zs.spring.demo1.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate" />
    </bean> -->

    <!-- ==================================2.使用XML配置声明式的事务管理(原始方式)=============================================== -->
    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置业务层的代理 -->
    <bean id="accountServiceProxy" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
        <!-- 配置目标对象 -->
        <property name="target" ref="accountService"/>
        <!-- 注入事务管理器 -->
        <property name="transactionManager" ref="transactionManager"/>
        <!-- 注入事务的属性 -->
        <property name="transactionAttributes">
            <props>
                <!--
                    prop的格式:
                  * PROPAGATION  :事务的传播行为
                  * ISOTATION       :事务的隔离级别
                  * readOnly    :只读
                  * -EXCEPTION   :发生哪些异常回滚事务
                  * +EXCEPTION   :发生哪些异常不回滚事务
                -->
                <prop key="transfer">PROPAGATION_REQUIRED</prop>
                <!-- <prop key="transfer">PROPAGATION_REQUIRED,readOnly</prop> -->
                <!-- <prop key="transfer">PROPAGATION_REQUIRED,+java.lang.ArithmeticException</prop> -->
            </props>
        </property>
    </bean>

</beans>
```

#### 基于AspectJ的XML方式

- Service实现

```
/**
 * @author yefan
 * @date 2018/12/05
 */
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    @Override
    public void transfer(String out, String in, Double money) {
        accountDao.outMoney(out, money);
//                int a = 2 / 0;
        accountDao.intMoney(in, money);
    }

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }
}
```

- springApplication配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context-3.1.xsd http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 引入外部的属性文件 -->
    <context:property-placeholder location="classpath:config/jdbc.properties"/>

    <!-- 配置c3p0连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driverClass}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!-- 配置业务层类 -->
    <bean id="accountService" class="com.yefan.study.transaction.demo3.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>

    <!-- 配置DAO类(简化，会自动配置JdbcTemplate) -->
    <bean id="accountDao" class="com.yefan.study.transaction.demo3.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置DAO类(未简化) -->
    <!-- <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="accountDao" class="com.zs.spring.demo1.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate" />
    </bean> -->

    <!-- ==================================3.使用XML配置声明式的事务管理,基于tx/aop=============================================== -->
    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置事务的通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!--
            propagation    :事务传播行为
            isolation  :事务的隔离级别
            read-only  :只读
            rollback-for:发生哪些异常回滚
            no-rollback-for    :发生哪些异常不回滚
            timeout       :过期信息
          -->
            <tx:method name="transfer" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <!-- 配置切面 -->
    <aop:config>
        <!-- 配置切入点 -->
        <aop:pointcut id="pointcut1" expression="execution(* com.yefan.study.transaction.demo3.AccountService+.*(..))"/>
        <!-- 配置切面 -->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut1"/>
    </aop:config>
</beans>
```

#### 基于注解方式

- Service实现

```
/**
 * @author yefan
 * @date 2018/12/05
 */
@Transactional(rollbackFor = Exception.class)
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    @Override
    public void transfer(String out, String in, Double money) {
        accountDao.outMoney(out, money);
//        int a = 2 / 0;
        accountDao.intMoney(in, money);
    }

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }
}
```

- springApplication配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context-3.1.xsd http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 引入外部的属性文件 -->
    <context:property-placeholder location="classpath:config/jdbc.properties"/>

    <!-- 配置c3p0连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driverClass}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!-- 配置业务层类 -->
    <bean id="accountService" class="com.yefan.study.transaction.demo4.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>

    <!-- 配置DAO类(简化，会自动配置JdbcTemplate) -->
    <bean id="accountDao" class="com.yefan.study.transaction.demo4.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置DAO类(未简化) -->
    <!-- <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="accountDao" class="com.zs.spring.demo1.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate" />
    </bean> -->

    <!-- ==================================4.使用注解配置声明式事务=============================================== -->
    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 开启注解事务 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

## 总结

Spring将事务管理分为两类：

- 编程式事务管理

  - 手动编写代码进行事务管理(很少使用)

    对业务代码的侵入性很强

- 声明式事务管理

  - 基于TransactionProxyFactoryBean的方式(很少使用)

    需要为每个进行事务管理的类，配置一个TransactionProxyFactoryBean进行增强

  - 基于AspectJ的XML方式(经常使用)

    一旦配置好之后，类上不需要添加任何东西

  - 基于注解方式(经常使用)

    配置简单，需要业务层类上添加一个@Transactional的注解





    