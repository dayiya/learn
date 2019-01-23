# 一、Redis与spring的集成
## 1.1 maven引入Redis客户端的依赖
```xml
<properties>
  <jedis.version>2.7.2</jedis.version>
</properties>
<dependencyManagement>
  <dependencies>
    <!-- Redis客户端 -->
		<dependency>
			<groupId>redis.clients</groupId>
			<artifactId>jedis</artifactId>
			<version>${jedis.version}</version>
		</dependency>
  </dependencies>
</dependencyManagement>
```
## 1.2 定义连接口，封装常用的redis操作
```java
public interface JedisClient {

	String set(String key, String value);//设置key和String类型的value
	String get(String key);//获取String类型的value
	Boolean exists(String key);//判断是否存在key
	Long expire(String key, int seconds);//给指定的key设置过期时间，单位为s
	Long ttl(String key);//获取指定的key的剩余的超时时间
	Long incr(String key);//给指定的key自增1
	Long hset(String key, String field, String value);//设置key和hash类型的value
	String hget(String key, String field);//根据key和field获取hash类型的值
	Long hdel(String key, String... field);//根据key和field（可多个）删除hash类型的value
}
```
### 1.2.1 单机版实现类
```java
public class JedisClientPool implements JedisClient {

	@Autowired
	private JedisPool jedisPool;

	@Override
	public String set(String key, String value) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.set(key, value);
		jedis.close();
		return result;
	}

	@Override
	public String get(String key) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.get(key);
		jedis.close();
		return result;
	}

	@Override
	public Boolean exists(String key) {
		Jedis jedis = jedisPool.getResource();
		Boolean result = jedis.exists(key);
		jedis.close();
		return result;
	}

	@Override
	public Long expire(String key, int seconds) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.expire(key, seconds);
		jedis.close();
		return result;
	}

	@Override
	public Long ttl(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.ttl(key);
		jedis.close();
		return result;
	}

	@Override
	public Long incr(String key) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.incr(key);
		jedis.close();
		return result;
	}

	@Override
	public Long hset(String key, String field, String value) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.hset(key, field, value);
		jedis.close();
		return result;
	}

	@Override
	public String hget(String key, String field) {
		Jedis jedis = jedisPool.getResource();
		String result = jedis.hget(key, field);
		jedis.close();
		return result;
	}

	@Override
	public Long hdel(String key, String... field) {
		Jedis jedis = jedisPool.getResource();
		Long result = jedis.hdel(key, field);
		jedis.close();
		return result;
	}

}
```
### 1.2.2 redis集群版实现类
```java
package com.didoumi.jedis;

import org.springframework.beans.factory.annotation.Autowired;

import redis.clients.jedis.JedisCluster;

public class JedisClientCluster implements JedisClient {

	@Autowired
	private JedisCluster jedisCluster;

	@Override
	public String set(String key, String value) {
		return jedisCluster.set(key, value);
	}

	@Override
	public String get(String key) {
		return jedisCluster.get(key);
	}

	@Override
	public Boolean exists(String key) {
		return jedisCluster.exists(key);
	}

	@Override
	public Long expire(String key, int seconds) {
		return jedisCluster.expire(key, seconds);
	}

	@Override
	public Long ttl(String key) {
		return jedisCluster.ttl(key);
	}

	@Override
	public Long incr(String key) {
		return jedisCluster.incr(key);
	}

	@Override
	public Long hset(String key, String field, String value) {
		return jedisCluster.hset(key, field, value);
	}

	@Override
	public String hget(String key, String field) {
		return jedisCluster.hget(key, field);
	}

	@Override
	public Long hdel(String key, String... field) {
		return jedisCluster.hdel(key, field);
	}

}

```
## 1.3 配置xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans4.2.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context4.2.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop4.2.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx4.2.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util4.2.xsd">
  <!-- 单机版和集群版不能共存，使用单机版时注释集群版的配置。使用集群版，把单机版注释。-->
	<!-- 配置单机版的连接 -->
	<bean id="jedisPool" class="redis.clients.jedis.JedisPool">
		<constructor-arg name="host" value="192.168.1.1"></constructor-arg>
		<constructor-arg name="port" value="6379"></constructor-arg>
	</bean>
	<bean id="jedisClientPool" class="com.didoumi.jedis.JedisClientPool"/>

  <!-- 集群版的配置 -->
  	<bean id="jedisCluster" class="redis.clients.jedis.JedisCluster">
  		<constructor-arg>
  			<set>
  				<bean class="redis.clients.jedis.HostAndPort">
  					<constructor-arg name="host" value="192.168.1.1"></constructor-arg>
  					<constructor-arg name="port" value="7001"></constructor-arg>
  				</bean>
  				<bean class="redis.clients.jedis.HostAndPort">
  					<constructor-arg name="host" value="192.168.1.1"></constructor-arg>
  					<constructor-arg name="port" value="7002"></constructor-arg>
  				</bean>
  				<bean class="redis.clients.jedis.HostAndPort">
  					<constructor-arg name="host" value="192.168.1.1"></constructor-arg>
  					<constructor-arg name="port" value="7003"></constructor-arg>
  				</bean>
  				<bean class="redis.clients.jedis.HostAndPort">
  					<constructor-arg name="host" value="192.168.1.1"></constructor-arg>
  					<constructor-arg name="port" value="7004"></constructor-arg>
  				</bean>
  				<bean class="redis.clients.jedis.HostAndPort">
  					<constructor-arg name="host" value="192.168.1.1"></constructor-arg>
  					<constructor-arg name="port" value="7005"></constructor-arg>
  				</bean>
  				<bean class="redis.clients.jedis.HostAndPort">
  					<constructor-arg name="host" value="192.168.1.1"></constructor-arg>
  					<constructor-arg name="port" value="7006"></constructor-arg>
  				</bean>
  			</set>
  		</constructor-arg>
  	</bean>
  	<bean id="jedisClientCluster" class="com.didoumi.jedis.JedisClientCluster"/>
</beans>
```
## 1.4 单元测试
```java
  @Test
	public void testJedisClient() throws Exception {
		//初始化Spring容器
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/applicationContext-redis.xml");
		//从容器中获得JedisClient对象
		JedisClient jedisClient = applicationContext.getBean(JedisClient.class);
		jedisClient.set("first", "100");
		String result = jedisClient.get("first");
		System.out.println(result);
	}
```
