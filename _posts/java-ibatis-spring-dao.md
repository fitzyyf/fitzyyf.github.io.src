title: "ibatis+spring集合后，在DAO中使用SqlMapClient无效的原因"
date: 2008-04-16 23:17
tags: 
- ibatis
- spring
categories: 
- 编程语言
---

最近公司因为项目需要，使用了 webwork+spring+ibatis作用框架应用，做为数据库DAO，感觉ibatis很好，至少对我而言是很不错的，呵呵。
昨天因为需要在DAO层使用ibatis中的SQLMapClient，发现无法读取SQLMapConfig.xml文件，后来查了很多，都无果，后来我就想是不是我们这个项目SqlMapConfig.xml文件中缺少了什么，因为我们将原本需要ibatis连接数据库的操作交给了Spring来管理，所以在SqlMapConfig.xml文件中只加载了ibatis的映射文件。所以我又改回将数据库连接由ibatis来管理。在使用SqlMapClient就可以了。

{% codeblock SqlMapConfig.xml原始文件 lang:xml %}
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE sqlMapConfig PUBLIC "-//ibatis.apache.org//DTD SQL Map Config 2.0//EN"
"http://ibatis.apache.org/dtd/sql-map-config-2.dtd">
<sqlMapConfig>
    <properties resource="config/jdbc.properties" />
    
    <sqlMap resource="com/hfjh/domain/User.xml"/>

</sqlMapConfig>

{% endcodeblock %}

{% codeblock SqlMapConfig.xml修改 lang:xml %}
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE sqlMapConfig PUBLIC "-//ibatis.apache.org//DTD SQL Map Config 2.0//EN"
"http://ibatis.apache.org/dtd/sql-map-config-2.dtd">
<sqlMapConfig>
    <properties resource="config/jdbc.properties" />
    
    <!--
    <settings
            cacheModelsEnabled="true"
            enhancementEnabled="false"
            lazyLoadingEnabled="false"
            maxRequests="512"
            maxSessions="128"
            maxTransactions="32"
            useStatementNamespaces="true"
            errorTracingEnabled="true"
            /> -->
    <transactionManager type="JDBC">
		<dataSource type="SIMPLE">
			<property name="JDBC.Driver" value="${jdbc.driverClass}" />
			<property name="JDBC.ConnectionURL" value="${jdbc.url}" />
			<property name="JDBC.Username" value="${jdbc.user}" />
			<property name="JDBC.Password" value="${jdbc.password}" />
		</dataSource>
	</transactionManager>
    
    <sqlMap resource="com/hfjh/domain/User.xml"/>
</sqlMapConfig>
{% endcodeblock %}

以前的数据库层是由这个spring配置文件管理的

{% codeblock applicationContext-ibatis.xml lang:xml%}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd"
       default-autowire="autodetect">
    <!--  <bean id="lazyDataSource" class="org.springframework.jdbc.datasource.LazyConnectionDataSourceProxy">
    <property name="targetDataSource">
        <ref bean="dataSource"/>
    </property>
  </bean>-->
    <bean id="dataSource"
          class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
        <property name="driverClass" value="${jdbc.driverClass}" />
        <property name="jdbcUrl" value="${jdbc.url}" />
        <property name="user" value="${jdbc.user}" />
        <property name="password" value="${jdbc.password}" />
        <property name="initialPoolSize" value="2" />
        <property name="minPoolSize" value="2" />
        <property name="maxPoolSize" value="10" />
        <property name="acquireIncrement" value="2" />
        <property name="maxIdleTime" value="2" />
        <property name="maxStatements" value="10" />
        <property name="autoCommitOnClose" value="false" />
    </bean>
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource">
            <ref local="dataSource"></ref>
        </property>
    </bean>
    <bean id="sqlMapClient"
          class="org.springframework.orm.ibatis.SqlMapClientFactoryBean">
        <property name="configLocation">
            <value>classpath:/config/SqlMapConfig.xml</value>
        </property>
    </bean>
</beans>
{% endcodeblock %}

取得SqlMapClinet

{% codeblock JhSQLMapConfig.java lang:java %}
package com.hfjh.common;

import java.io.Reader;

import com.ibatis.common.resources.Resources;
import com.ibatis.sqlmap.client.SqlMapClient;
import com.ibatis.sqlmap.client.SqlMapClientBuilder;

public class JhSQLMapConfig {
	private static final SqlMapClient sqlMap;
	// 在静态区块中初试化返回

	static {
		try {

			// 声明配置文件的名称（映射文件被定义在其中）

			String resource = "config/SqlMapConfig.xml";

			// 利用工具类Resources来读取到配置文件

			Reader reader = Resources.getResourceAsReader(resource);

			// 创建SqlMapClient接口的变量实例

			sqlMap = SqlMapClientBuilder.buildSqlMapClient(reader);

		} catch (Exception e) {

			e.printStackTrace();

			throw new RuntimeException(
			"Error initializing MyAppSqlConfig class. Cause: " + e);
		}

	}

	public static SqlMapClient getSqlMapInstance() {
		// 提供静态方法返回静态区块中得到的SqlMapClient
		return sqlMap;

	}
}
{% endcodeblock %}

使用 SqlMapClient

{% codeblock use JhSQLMapConfig lang:java %}
private SqlMapClient sqlMap = JhSQLMapConfig.getSqlMapInstance();

/**
 * 取得所有的基本信息，做为导出之用
 * @return
 */
public List selectJbxx() {
    List jbxxList = null;
    try {
        jbxxList = sqlMap.queryForList("");
    } catch (Exception e) {
        e.printStackTrace();
    }
    return jbxxList;
}
{% endcodeblock %}
