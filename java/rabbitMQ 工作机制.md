
# RabbitMQ 工作机制 #

## RabbitMQ简介 ##

**通信协议**

AMQP（Advanced Message Queuing Protocol），即高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。

AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。RabbitMQ是一个开源的AMQP实现，服务器端用Erlang语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持AJAX。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

**架构**

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-arch.png)

概念说明: 

Broker:它提供一种传输服务,它的角色就是维护一条从生产者到消费者的路线，保证数据能按照指定的方式进行传输

Exchange：消息交换机,它指定消息按什么规则,路由到哪个队列

Queue:消息的载体,每个消息都会被投到一个或多个队列

Binding:绑定，它的作用就是把exchange和queue按照路由规则绑定起来

Routing Key:路由关键字,exchange根据这个关键字进行消息投递

vhost:虚拟主机,一个broker里可以有多个vhost，用作不同用户的权限分离

Producer:消息生产者,就是投递消息的程序

Consumer:消息消费者,就是接受消息的程序
 
Channel:消息通道,在客户端的每个连接里,可建立多个channel


## 队列--消息分发机制 ##

**轮询分发（Round-robin dispathching）**

RabbbitMQ的轮询分发机制非常适合扩展,而且它是专门为并发程序设计的,如果现在负载加重,那么只需要创建更多的Consumer来进行消息处理。

**消息确认（Message acknowledgment）**

为了保证数据不被丢失,RabbitMQ支持消息确认机制,为了保证数据能被正确处理，而不仅仅是被Consumer收到（即有可能Consumer消费失败）,那么我们应该是在处理完数据之后发送ack，而不能采用autoAck。

在正确处理完数据之后发送ack,就是告诉RabbitMQ数据已经被成功消费,RabbitMQ可以安全的删除它。 如果Consumer退出了但是没有发送ack,那么RabbitMQ就会把这个Message发送到下一个Consumer，这样就保证在Consumer异常退出情况下数据也不会丢失。

RabbitMQ它没有用到超时机制。RabbitMQ仅仅通过Consumer的连接中断来确认该Message并没有正确处理，也就是说RabbitMQ给了Consumer足够长的时间做数据处理。 如果忘记ack,那么当Consumer退出时,Mesage会重新分发,然后RabbitMQ会占用越来越多的内存.

## 队列--消息均衡机制 ##

Prefetch count

前面我们讲到如果有多个消费者同时订阅同一个Queue中的消息，Queue中的消息会被平摊给多个消费者。这时如果每个消息的处理时间不同，就有可能会导致某些消费者一直在忙，而另外一些消费者很快就处理完手头工作并一直空闲的情况。我们可以通过设置prefetchCount来限制Queue每次发送给每个消费者的消息数，比如我们设置prefetchCount=1，则Queue每次给某个消费者发送一条消息，消费者处理完这条消息后Queue才会再给该消费者发送下一条消息。

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-prefetch.png)

通过basic.qos方法设置prefetch_count=1，这样RabbitMQ就会使得每个Consumer在同一个时间点最多处理一个Message，换句话说，在接收到该Consumer的ack前，它不会将新的Message分发给它。

	channel.basic_qos(prefetch_count=1) 

注意，这种方法可能会导致queue满。当然，这种情况下你可能需要添加更多的Consumer，或者创建更多的virtualHost来细化你的设计。

## 队列--消息持久化（Message durability） ##

要持久化队列queue的持久化需要在声明时指定durable=True; 这里要注意,队列的名字一定要是Broker中不存在的,否则是不能改变此队列的任何属性。

另外，需要强调说明一下，队列和交换机有一个创建时候指定的标志durable,durable的唯一含义就是具有这个标志的队列和交换机会在重启之后重新建立,它不表示说在队列中的消息会在重启后恢复。

消息持久化包括3部分

1、exchange持久化，在声明时指定durable => true

	hannel.ExchangeDeclare(ExchangeName, "direct", durable: true, autoDelete: false, arguments: null);//声明消息队列，且为可持久化的

2、queue持久化，在声明时指定durable => true

	channel.QueueDeclare(QueueName, durable: true, exclusive: false, autoDelete: false, arguments: null);//声明消息队列，且为可持久化的

3、消息持久化，在投递时指定delivery_mode => 2(1是非持久化)

	channel.basicPublish("", queueName, MessageProperties.PERSISTENT_TEXT_PLAIN, msg.getBytes());  

如果exchange和queue都是持久化的，那么它们之间的binding也是持久化的，如果exchange和queue两者之间有一个持久化，一个非持久化，则不允许建立绑定。

注意：一旦创建了队列和交换机，就不能修改其标志了。例如，创建了一个non-durable的队列，然后想把它改变成durable的，唯一的办法就是删除这个队列然后重现创建。

## 消息路由（Exchange） ##

生产者将消息发送到Exchange（交换器，下图中的X），再由Exchange将消息路由到一个或多个Queue中（或者丢弃）。

Exchange是按照什么逻辑将消息路由到Queue的？

生产者在将消息发送给Exchange的时候，需要指定一个routing key，通过指定消息的路由规则来投递消息到队列中。路由规则需要routing key与Exchange Type及binding key联合使用才能最终生效。RabbitMQ为routing key设定的长度限制为255 bytes。

在Exchange Type与binding key固定的情况下（在正常使用时一般这些内容都是固定配置好的），我们的生产者就可以在发送消息给Exchange时，通过指定routing key来决定消息流向对应的队列。Exchange Type与binding key的关联是通过binding来完成的。

在绑定（Binding）Exchange与Queue的同时，需要指定一个binding key；消费者将消息发送给Exchange时，一般会指定一个routing key；当binding key与routing key相匹配时，消息将会被路由到对应的Queue中。这个将在Exchange Types章节会列举实际的例子加以说明。
在绑定多个Queue到同一个Exchange的时候，这些Binding允许使用相同的binding key。

binding key 并不是在所有情况下都生效，它依赖于Exchange Type，比如fanout类型的Exchange就会无视binding key，而是将消息路由到所有绑定到该Exchange的Queue。

RabbitMQ常用的Exchange Type有fanout、direct、topic、headers这四种（AMQP规范里还提到两种Exchange Type，分别为system与自定义，这里不予以描述），下面分别进行介绍。

**fanout**

fanout类型的Exchange路由规则非常简单，它会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中。

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-exchange-fanout.png)

上图中，生产者（P）发送到Exchange（X）的所有消息都会路由到图中的两个Queue，并最终被两个消费者（C1与C2）消费。

**direct**

direct类型的Exchange路由规则也很简单，它会把消息路由到那些binding key与routing key完全匹配的Queue中。

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-exchange-direct.png)

以上图的配置为例，我们以routingKey=”error”发送消息到Exchange，则消息会路由到Queue1（amqp.gen-S9b…，这是由RabbitMQ自动生成的Queue名称）和Queue2（amqp.gen-Agl…）；如果我们以routingKey=”info”或routingKey=”warning”来发送消息，则消息只会路由到Queue2。如果我们以其他routingKey发送消息，则消息不会路由到这两个Queue中。

**topic**

前面讲到direct类型的Exchange路由规则是完全匹配binding key与routing key，但这种严格的匹配方式在很多情况下不能满足实际业务需求。topic类型的Exchange在匹配规则上进行了扩展，它与direct类型的Exchage相似，也是将消息路由到binding key与routing key相匹配的Queue中，但这里的匹配规则有些不同，它约定：
	
1、routing key为一个句点号“. ”分隔的字符串（我们将被句点号“. ”分隔开的每一段独立的字符串称为一个单词），如“stock.usd.nyse”、“nyse.vmw”、“quick.orange.rabbit”

2、binding key与routing key一样也是句点号“. ”分隔的字符串

3、binding key中可以存在两种特殊字符“*”与“#”，用于做模糊匹配，其中“*”用于匹配一个单词，“#”用于匹配多个单词（可以是零个）

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-exchange-topic.png)

以上图中的配置为例，routingKey=”quick.orange.rabbit”的消息会同时路由到Q1与Q2，routingKey=”lazy.orange.fox”的消息会路由到Q1与Q2，routingKey=”lazy.brown.fox”的消息会路由到Q2，routingKey=”lazy.pink.rabbit”的消息会路由到Q2（只会投递给Q2一次，虽然这个routingKey与Q2的两个bindingKey都匹配）；routingKey=”quick.brown.fox”、routingKey=”orange”、routingKey=”quick.orange.male.rabbit”的消息将会被丢弃，因为它们没有匹配任何bindingKey。

**headers**

headers类型的Exchange不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配。
在绑定Queue与Exchange时指定一组键值对；当消息发送到Exchange时，RabbitMQ会取到该消息的headers（也是一个键值对的形式），对比其中的键值对是否完全匹配Queue与Exchange绑定时指定的键值对；如果完全匹配则消息会路由到该Queue，否则不会路由到该Queue。

Headers类型的exchange使用的比较少，它也是忽略routingKey的一种路由方式。是使用Headers来匹配的。Headers是一个键值对，可以定义成Hashtable。发送者在发送的时候定义一些键值对，接收者也可以再绑定时候传入一些键值对，两者匹配的话，则对应的队列就可以收到消息。匹配有两种方式all和any。这两种方式是在接收端必须要用键值"x-mactch"来定义。all代表定义的多个键值对都要满足，而any则代码只要满足一个就可以了。

## 源码分析 ##

接下来以header类型举例

	public class Producer {  
	    private final static String EXCHANGE_NAME = "header-exchange";  
	      
	    @SuppressWarnings("deprecation")  
	    public static void main(String[] args) throws Exception {  
	        // 创建连接和频道  
	        ConnectionFactory factory = new ConnectionFactory();  
	        factory.setHost("192.168.36.102");  
	        // 指定用户 密码  
	        factory.setUsername("admin");  
	        factory.setPassword("admin");  
	        // 指定端口  
	        factory.setPort(AMQP.PROTOCOL.PORT);  
	        Connection connection = factory.newConnection();  
	        Channel channel = connection.createChannel();  
	          
	        //声明转发器和类型headers  
	        channel.exchangeDeclare(EXCHANGE_NAME, ExchangeTypes.HEADERS,false,true,null);  
	        String message = new Date().toLocaleString() + " : log something";  
	          
	        Map<String,Object> headers =  new Hashtable<String, Object>();  
	        headers.put("aaa", "01234");  
	        Builder properties = new BasicProperties.Builder();  
	        properties.headers(headers);  
	          
	        // 指定消息发送到的转发器,绑定键值对headers键值对  
	        channel.basicPublish(EXCHANGE_NAME, "",properties.build(),message.getBytes());  
	          
	        System.out.println("Sent message :'" + message + "'");  
	        channel.close();  
	        connection.close();  
	    }  
	}  
	
	public class Consumer {  
	    private final static String EXCHANGE_NAME = "header-exchange";  
	    private final static String QUEUE_NAME = "header-queue";  
	      
	    public static void main(String[] args) throws Exception {  
	        // 创建连接和频道  
	        ConnectionFactory factory = new ConnectionFactory();  
	        factory.setHost("192.168.36.102");  
	        // 指定用户 密码  
	        factory.setUsername("admin");  
	        factory.setPassword("admin");  
	        // 指定端口  
	        factory.setPort(AMQP.PROTOCOL.PORT);  
	        Connection connection = factory.newConnection();  
	        Channel channel = connection.createChannel();  
	          
	        //声明转发器和类型headers  
	        channel.exchangeDeclare(EXCHANGE_NAME, ExchangeTypes.HEADERS,false,true,null);  
	        channel.queueDeclare(QUEUE_NAME,false, false, true,null);  
	          
	        Map<String, Object> headers = new Hashtable<String, Object>();  
	        headers.put("x-match", "any");//all any  
	        headers.put("aaa", "01234");  
	        headers.put("bbb", "56789");  
	        // 为转发器指定队列，设置binding 绑定header键值对  
	        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME,"", headers);  
	        QueueingConsumer consumer = new QueueingConsumer(channel);  
	        // 指定接收者，第二个参数为自动应答，无需手动应答  
	        channel.basicConsume(QUEUE_NAME, true, consumer);  
	        while (true) {  
	            QueueingConsumer.Delivery delivery = consumer.nextDelivery();  
	            String message = new String(delivery.getBody());  
	            System.out.println(message);  
	        }   
	    }  
	}  

从示例代码中我们知道3个重要类：ConnectionFactory、Connection、Channel。ConnectionFactory、Connection、Channel都是RabbitMQ对外提供的API中最基本的对象。

Connection是RabbitMQ的socket链接，它封装了socket协议相关部分逻辑。

ConnectionFactory为Connection的制造工厂。

Channel是我们与RabbitMQ打交道的最重要的一个接口，我们大部分的业务操作是在Channel这个接口中完成的，包括定义Queue、定Exchange、绑定Queue与Exchange、发布消息等。

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-uml.png)

**SimpleMessageListenerContainer**

SimpleMessageListenerContainer是一个容器，我们在Spring中配置RabbitMQ的消费者就是对应的该类。该类的父类AbstractMessageListenerContainer实现了Lifecycle接口，因此该类的生命周期是托管给Spring容器，即初始化会由Spring启动。我们来看该类在初始化涉及到的关键信息：

SimpleMessageListenerContainer类invokeListener()方法，该方法在初始化就会被调用，方法的作用是完成消息的监听。监听ChannelAwareMessageListener和MessageListener接口的实现。

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-code-container.png)

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-code-container-invoke.png)

SimpleMessageListenerContainer类的doStart()方法，该方法会被父类AbstractMessageListenerContainer的start()方法调用。

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-code-container-start.png)

**AsyncMessageProcessingConsumer**

AsyncMessageProcessingConsumer类实现了Runnable接口，该类的作用是异步去创建处理消息的消费者。

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-code-consumer-run.png)

**BlockingQueueConsumer**

BlockingQueueConsumer类的作用就是将队列数组循环完成创建，并与Channel完成绑定。感兴趣的请查看源码。

**RabbitTemplate**

RabbitTemplate是用于生产者发送消息的模板类，调用该类的send()方法发送消息。比较简单，感兴趣的请查看源码。

**ConnectionFactory**

ConnectionFactory接口存在两个实现，CachingConnectionFactory和SimpleRoutingConnectionFactory。CachingConnectionFactory很好理解，是提供缓存支持的;SimpleRoutingConnectionFactory是从当前线程中获取ConnectionFactory。AbstractRoutingConnectionFactory类提供了配置许多的不同的Connection Factory的映射，并且能够根据运行时的lookupKey(通过绑定线程上下文的方式) 来决定使用哪个具体的ConnectionFactory。

**Channel**

Channel接口主要是定义Queue、定Exchange、绑定Queue与Exchange、发布消息等，该接口的实现有ChannelN、AutorecoveringChannel、PublisherCallbackChannelImpl。AutorecoveringChannel、PublisherCallbackChannelImpl都是通过静态代理的方式，最终都是调用的ChannelN类中的方法。

RabbitMQ消息最终发送出去是通过AMQCommand类的transmit()方法来完成。接下来我们来看一下：

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-code-command.png)

**AMQConnection**

AMQConnection类实现Connection接口，是每次消息处理的载体。该类的创建由ConnectionFactory的newConnection()发起。

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-code-factory.png)

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-code-connection.png)

MainLoop是AMQConnection的私有类，MainLoop类实现了Runnable接口，该类的主要作用是开启一个线程，用于接收或者发送消息。

![](https://github.com/longtian2/cc3/blob/master/images/rabbitMQ/rabbitmq-code-mainloop.png)

参考文献：

   https://blog.csdn.net/whycold/article/details/41119807


联系方式：

https://github.com/longtian2

**如有用请不吝打赏**

![](https://github.com/longtian2/cc3/blob/master/images/wechat_pay.png)