# 一、ActiveMQ简介

-   ActiveMQ 是Apache出品，最流行的，能力强劲的开源消息总线。ActiveMQ 是一个完全支持JMS1.1和J2EE 1.4规范的 JMS Provider实现，尽管JMS规范出台已经是很久的事情了,但是JMS在当今的J2EE应用中间仍然扮演着特殊的地位。
-   **主要特点**
    -   多种语言和协议编写客户端。语言: Java, C, C++, C#, Ruby, Perl, Python, PHP。应用协议: OpenWire,Stomp REST,WS Notification,XMPP,AMQP
    -   完全支持JMS1.1和J2EE 1.4规范 (持久化,XA消息,事务)
    -   对Spring的支持,ActiveMQ可以很容易内嵌到使用Spring的系统里面去,而且也支持Spring2.0的特性
    -   通过了常见J2EE服务器(如 Geronimo,JBoss 4, GlassFish,WebLogic)的测试,其中通过JCA 1.5 resource adaptors的配置,可以让ActiveMQ可以自动的部署到任何兼容J2EE 1.4 商业服务器上
    -   支持多种传送协议:in-VM,TCP,SSL,NIO,UDP,JGroups,JXTA
    -   支持通过JDBC和journal提供高速的消息持久化
    -   从设计上保证了高性能的集群,客户端-服务器,点对点
    -   支持Ajax
    -   支持与Axis的整合
    -   可以很容易得调用内嵌JMS provider,进行测试

# 二、ActiveMQ的消息形式

## 2.1 消息类型

对于消息的传递有两种类型

-   **一种是点对点的**，及一个生产者和一个消费者一一对应；
-   **另一种是发布/订阅模式**，即一个生产者产生消息并进行发送后，可以由多个消费者进行接收。

JMS（Java Message Service）定义了五种不同的消息正文格式，以及调用的消息类型，允许你发送并接收以一些不同形式的数据，根据现有消息格式的一些级别的兼容性。

-   StreamMessage   Java原始值的数据流
-   MapMessage      一套名称-值对
-   TextMessage     一个字符串对象
-   ObjectMessage   一个序列化的Java对象
-   ByteMessage     一个字节的数据流

## 2.2 消息确认
&emsp;&emsp;JMS 消息只有在被确认之后，才认为已经被成功地消费了。消息的成功消费通常包含三个阶段：客户接收消息、客户处理消息和消息被确认。<br>
&emsp;&emsp;在事务性会话中，当一个事务被提交的时候，确认自动发生。在非事务性会话中，消息何时被确认取决于创建会话时的应答模式（acknowledgementmode）。该参数有以下三个可选值：
-   **Session.AUTO_ACKNOWLEDGE**：当客户成功的从receive方法返回的时候，或者从MessageListener.onMessage 方法成功返回的时候，会话自动确认客户收到的消息。
-   **Session.CLIENT_ACKNOWLEDGE**：客户通过消息的acknowledge 方法确认消息。需要注意的是，在这种模式中，确认是在会话层上进行：确认一个被消费的消息将自动确认所有已被会话消费的消息。例如，如果一个消息消费者消费了10 个消息，然后确认第5 个消息，那么所有10 个消息都被确认。
-   **Session.DUPS_ACKNOWLEDGE**：该选择只是会话迟钝的确认消息的提交。如果JMS provider 失败，那么可能会导致一些重复的消息。如果是重复的消息，那么JMS provider 必须把消息头的JMSRedelivered 字段设置为true。

## 2.3 消息持久化
### 2.3.1 AMQ Message Store
&emsp;&emsp;AMQ Message Store 是ActiveMQ5.0缺省的持久化存储。Message commands 被保存到transactional journal（由rolling data logs 组成）。Messages 被保存到data logs 中，同时被reference store 进行索引以提高存取速度。Date logs由一些单独的data log 文件组成，缺省的文件大小是32M，如果某个消息的大小超过了data log 文件的大小，那么可以修改配置以增加data log 文件的大小。<br>
&emsp;&emsp;如果某个data log 文件中所有的消息都被成功消费了，那么这个data log 文件将会被标记，以便在下一轮的清理中被删除或者归档。以下是其配置的一个例子：
```xml
<brokerbrokerName="broker" persistent="true"useShutdownHook="false">
  <persistenceAdapter>
    <amqPersistenceAdapterdirectory="${activemq.base}/data" maxFileLength="32mb"/>
  </persistenceAdapter>
</broker>
```
### 2.3.2 Kaha Persistence
&emsp;&emsp;Kaha Persistence 是一个专门针对消息持久化的解决方案。它对典型的消息使用模式进行了优化。在Kaha 中，数据被追加到data logs 中。当不再需要log文件中的数据的时候，log 文件会被丢弃。以下是其配置的一个例子：
```xml
<brokerbrokerName="broker" persistent="true"useShutdownHook="false">
  <persistenceAdapter>
    <kahaPersistenceAdapterdirectory="activemq-data" maxDataFileLength="33554432"/>
  </persistenceAdapter>
</broker>
```
### 2.3.3 JDBC Persistence
&emsp;&emsp;目前支持的数据库有Apache Derby,Axion, DB2, HSQL, Informix, MaxDB, MySQL, Oracle,Postgresql, SQLServer, Sybase。如果你使用的数据库不被支持，那么可以调整StatementProvider来保证使用正确的SQL 方言（flavour of SQL）。通常绝大多数数据库支持以下adaptor：
* org.activemq.store.jdbc.adapter.BlobJDBCAdapter
* org.activemq.store.jdbc.adapter.BytesJDBCAdapter
* org.activemq.store.jdbc.adapter.DefaultJDBCAdapter
* org.activemq.store.jdbc.adapter.ImageJDBCAdapter

也可以在配置文件中直接指定JDBC adaptor，例如：
```xml
<jdbcPersistenceAdapter adapterClass="org.apache.activemq.store.jdbc.adapter.ImageBasedJDBCAdaptor"/>
```
```xml
<persistence>
  <jdbcPersistence dataSourceRef="mysql-ds"/>
</persistence>
<bean id="mysql-ds"class="org.apache.commons.dbcp.BasicDataSource"destroy-method="close">
  <property name="driverClassName"value="com.mysql.jdbc.Driver"/>
  <property name="url" value="jdbc:mysql://localhost/activemq?relaxAutoCommit=true"/>
  <propertyname="username" value="activemq"/>
  <property name="password"value="activemq"/>
  <propertyname="poolPreparedStatements" value="true"/>
</bean>
```
需要注意的是，如果使用MySQL，那么需要设置relaxAutoCommit 标志为true。<br>
**查看原文详细信息**：https://blog.csdn.net/boonya/article/details/51074478

# 三、ActiveMQ的安装

## 3.1 下载

进入<http://activemq.apache.org/> 下载ActiveMQ
<img alt="ActiveMq的简介及使用-a3d12501.png" src="assets/ActiveMq的简介及使用-a3d12501.png" width="" height="" >

## 3.2 安装

1.  由于ActiveMq为java代码编写，需要安装jdk
2.  上传到linux系统
3.  解压
4.  启动：使用bin目录下activeMq命令启动：./activemq start
5.  关闭：./activemq stop
6.  查看状态：./activemq status
7.  注意：如果ActiveMQ整合spring使用不要使用activemq-all-5.12.0.jar包。建议使用5.11.2
8.  进入管理后台：<http://192.168.1.1:8161/admin> 用户名：admin 密码：admin
    <img alt="ActiveMq的简介及使用-6188c05b.png" src="assets/ActiveMq的简介及使用-6188c05b.png" width="" height="" >

# 四、ActiveMQ的使用

## 4.1 Queue

### 4.1.1 添加jar包至工程中

```html
<properties>
<activemq.version>5.11.2</activemq.version>
</properties>
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.apache.activemq</groupId>
      <artifactId>activemq-all</artifactId>
      <version>${activemq.version}</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

### 4.1.2 Producer端测试代码

-   生产消息，发送端。

1.  创建一个ConnectionFactory对象
2.  从ConnectionFactory对象中获取Connection对象。
3.  开启连接，调用Connection对象的start方法。
4.  从Connection对象中获取Session对象
5.  使用Session对象创建一个Destination对象（topic、queue），此处创建一个Queue对象。
6.  使用Session对象创建一个Producer对象。
7.  创建一个Message对象，创建一个TextMessage对象
8.  使用Producer对象发送消息。
9.  关闭资源

```java
@Test
public void testQueueProducer() throws Exception {
  // 第一步：创建ConnectionFactory对象，需要指定服务端ip及端口号。
  //brokerURL服务器的ip及端口号
  ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.1.168:61616");
  // 第二步：使用ConnectionFactory对象创建一个Connection对象。
  Connection connection = connectionFactory.createConnection();
  // 第三步：开启连接，调用Connection对象的start方法。
  connection.start();
  // 第四步：使用Connection对象创建一个Session对象。
  //第一个参数：是否开启事务。true：开启事务，第二个参数忽略。
  //第二个参数：当第一个参数为false时，才有意义。消息的应答模式。1、自动应答2、手动应答。
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  // 第五步：使用Session对象创建一个Destination对象（topic、queue），此处创建一个Queue对象。
  //参数：队列的名称。
  Queue queue = session.createQueue("test-queue");
  // 第六步：使用Session对象创建一个Producer对象。
  MessageProducer producer = session.createProducer(queue);
  // 第七步：创建一个Message对象，创建一个TextMessage对象。
  /*TextMessage message = new ActiveMQTextMessage();
  message.setText("hello activeMq,this is my first test.");*/
  TextMessage textMessage = session.createTextMessage("hello activeMq,this is my first test.");
  // 第八步：使用Producer对象发送消息。
  producer.send(textMessage);
  // 第九步：关闭资源。
  producer.close();
  session.close();
  connection.close();
}
```

### 4.1.3 Comsumer

-   消费者：接收消息。

1.  创建一个ConnectionFactory对象
2.  从ConnectionFactory对象中获取Connection对象。
3.  开启连接，调用Connection对象的start方法。
4.  从Connection对象中获取Session对象
5.  使用Session对象创建一个Destination对象。和发送端保持一致queue，并且队列的名称一致。
6.  使用Session对象创建一个Consumer对象。
7.  接收消息。
8.  关闭资源

```java
@Test
public void testQueueConsumer() throws Exception {
  // 第一步：创建一个ConnectionFactory对象。
  ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.1.168:61616");
  // 第二步：从ConnectionFactory对象中获得一个Connection对象。
  Connection connection = connectionFactory.createConnection();
  // 第三步：开启连接。调用Connection对象的start方法。
  connection.start();
  // 第四步：使用Connection对象创建一个Session对象。
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  // 第五步：使用Session对象创建一个Destination对象。和发送端保持一致queue，并且队列的名称一致。
  Queue queue = session.createQueue("test-queue");
  // 第六步：使用Session对象创建一个Consumer对象。
  MessageConsumer consumer = session.createConsumer(queue);
  // 第七步：接收消息。
  consumer.setMessageListener(new MessageListener() {

  @Override
  public void onMessage(Message message) {
    try {
      TextMessage textMessage = (TextMessage) message;
      String text = null;
      //取消息的内容
      text = textMessage.getText();
      // 第八步：打印消息。
      System.out.println(text);
      } catch (JMSException e) {
        e.printStackTrace();
      }
    }
  });
  //等待键盘输入
  System.in.read();
  // 第九步：关闭资源
  consumer.close();
  session.close();
  connection.close();
}
```

## 4.2 Topic

### 4.2.1 Producer

-   生产消息，发送端。

1.  创建一个ConnectionFactory对象
2.  从ConnectionFactory对象中获取Connection对象。
3.  开启连接，调用Connection对象的start方法。
4.  从Connection对象中获取Session对象
5.  使用Session对象创建一个Destination对象（topic、queue），此处创建一个Topic对象。
6.  使用Session对象创建一个Producer对象。
7.  创建一个Message对象，创建一个TextMessage对象
8.  使用Producer对象发送消息。
9.  关闭资源

```java
@Test
public void testTopicProducer() throws Exception {
  // 第一步：创建ConnectionFactory对象，需要指定服务端ip及端口号。
  // brokerURL服务器的ip及端口号
  ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.1.168:61616");
  // 第二步：使用ConnectionFactory对象创建一个Connection对象。
  Connection connection = connectionFactory.createConnection();
  // 第三步：开启连接，调用Connection对象的start方法。
  connection.start();
  // 第四步：使用Connection对象创建一个Session对象。
  // 第一个参数：是否开启事务。true：开启事务，第二个参数忽略。
  // 第二个参数：当第一个参数为false时，才有意义。消息的应答模式。1、自动应答2、手动应答。
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  // 第五步：使用Session对象创建一个Destination对象（topic、queue），此处创建一个topic对象。
  // 参数：话题的名称。
  Topic topic = session.createTopic("test-topic");
  // 第六步：使用Session对象创建一个Producer对象。
  MessageProducer producer = session.createProducer(topic);
  // 第七步：创建一个Message对象，创建一个TextMessage对象。
  /*
  * TextMessage message = new ActiveMQTextMessage();  message.setText(
  * "hello activeMq,this is my first test.");
  */
  TextMessage textMessage = session.createTextMessage("hello activeMq,this is my topic test");
  // 第八步：使用Producer对象发送消息。
  producer.send(textMessage);
  // 第九步：关闭资源。
  producer.close();
  session.close();
  connection.close();
}
```

### 4.2.2 Producer

-   消费者：接收消息。

1.  创建一个ConnectionFactory对象
2.  从ConnectionFactory对象中获取Connection对象。
3.  开启连接，调用Connection对象的start方法。
4.  从Connection对象中获取Session对象
5.  使用Session对象创建一个Destination对象。和发送端保持一致topic，并且话题的名称一致。
6.  使用Session对象创建一个Consumer对象。
7.  接收消息。
8.  关闭资源

```java
@Test
public void testTopicConsumer() throws Exception {
  // 第一步：创建一个ConnectionFactory对象。
  ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.25.168:61616");
  // 第二步：从ConnectionFactory对象中获得一个Connection对象。
  Connection connection = connectionFactory.createConnection();
  // 第三步：开启连接。调用Connection对象的start方法。
  connection.start();
  // 第四步：使用Connection对象创建一个Session对象。
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  // 第五步：使用Session对象创建一个Destination对象。和发送端保持一致topic，并且话题的名称一致。
  Topic topic = session.createTopic("test-topic");
  // 第六步：使用Session对象创建一个Consumer对象。
  MessageConsumer consumer = session.createConsumer(topic);
  // 第七步：接收消息。
  consumer.setMessageListener(new MessageListener() {

    @Override
    public void onMessage(Message message) {
      try {
        TextMessage textMessage = (TextMessage) message;
        String text = null;
        // 取消息的内容
        text = textMessage.getText();
        // 第八步：打印消息。
        System.out.println(text);
      } catch (JMSException e) {
        e.printStackTrace();
      }
    }
  });
  System.out.println("topic的消费端03。。。。。");
  // 等待键盘输入
  System.in.read();
  // 第九步：关闭资源
  consumer.close();
  session.close();
  connection.close();
}
```

# 五、ActiveMq整合Spring

## 5.1 添加maven依赖

```xml
<properties>
  <activemq.version>5.11.2</activemq.version>
  <spring.version>4.2.4.RELEASE</spring.version>
</properties>
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.apache.activemq</groupId>
      <artifactId>activemq-all</artifactId>
      <version>${activemq.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jms</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context-support</artifactId>
      <version>${spring.version}</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

## 5.2 ActiveMq整合Spring发送端的xml文件配置

```xml
<!--配置ConnectionFactory-->
<!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供 -->
<bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
	<property name="brokerURL" value="tcp://192.168.1.168:61616" />
</bean>
<!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
<bean id="connectionFactory"
	class="org.springframework.jms.connection.SingleConnectionFactory">
	<!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
	<property name="targetConnectionFactory" ref="targetConnectionFactory" />
</bean>

<!-- 配置生产者，使用Spring提供的工具类 -->
<bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
<!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->
	<property name="connectionFactory" ref="connectionFactory" />
</bean>
<!--目的地可以有多个-->
<!--这个是队列目的地，点对点的，注入时使用@Resource来注入 -->
<bean id="queueDestination" class="org.apache.activemq.command.ActiveMQQueue">
	<constructor-arg>
		<value>spring-queue</value><!--此处是目的地的名称-->
	</constructor-arg>
</bean>
<!--这个是主题目的地，一对多的，注入时使用@Resource来注入 -->
<bean id="topicDestination" class="org.apache.activemq.command.ActiveMQTopic">
	<constructor-arg value="topic" />
</bean>
```

## 5.3 发送消息

```java
//注入模板
@Autowired
private JmsTemplate jmsTemplate;
//从spring中获取发消息的类型，这里根据id来获取，所以用@Resource较好
@Resource
private Destination topicDestination;

@Override
public void methodName() {
  // 。。。业务逻辑
  //发送消息代码
  jmsTemplate.send(topicDestination, new MessageCreator(){
    @Override
      public Message createMessage(Session session) throws JMSException {
        TextMessage textMessage = session.createTextMessage("要发送的消息！");
      return textMessage;
    }
  });
}
```

## 5.4 ActiveMq整合Spring接收端的xml文件配置

```xml
<bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
	<property name="brokerURL" value="tcp://192.168.1.168:61616" />
</bean>
<!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
<bean id="connectionFactory"
	class="org.springframework.jms.connection.SingleConnectionFactory">
	<!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
	<property name="targetConnectionFactory" ref="targetConnectionFactory" />
</bean>

<bean id="topicDestination" class="org.apache.activemq.command.ActiveMQTopic">
	<constructor-arg value="topic" />
</bean>
<!-- 接收消息 可配置多个-->
<!-- 配置监听器，此类监听到消息后处理业务 -->
<bean id="changeListener" class="com.didoumi.listener.ChangeListener" />
<!-- 消息监听容器 -->
<bean class="org.springframework.jms.listener.DefaultMessageListenerContainer">
	<property name="connectionFactory" ref="connectionFactory" />
	<property name="destination" ref="topicDestination" /><!--监听广播形式-->
	<property name="messageListener" ref="changeListener" /><!--监听广播形式-->
</bean>
```

## 5.5 接收消息并处理

```java
public class ChangeListener implements MessageListener {

  @Override
  public void onMessage(Message message) {
    try {
      TextMessage textMessage = null;
      if (message instanceof TextMessage) {
        textMessage = (TextMessage) message;
        String str = textMessage.getText();
        System.out.println(str);//这里打印出：要发送的消息！
      }
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}
```
