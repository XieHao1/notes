# 1RabbitMQ

# 一.RabbitMQ 的概念

​	RabbitMQ 是一个消息中间件：它接收，存储和转发消息数据。



# 二.四大核心概念

## 1.生产者

产生数据发送消息的程序是生产者

## 2.交换机

交换机是 RabbitMQ 非常重要的一个部件，一方面它接收来自生产者的消息，另一方面它将消息推送到队列中。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推送到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定。

## 3.队列

队列是 RabbitMQ 内部使用的一种数据结构，尽管消息流经 RabbitMQ 和应用程序，但它们只能存储在队列中。队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。许多生产者可以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们使用队列的方式

## 4.消费者

​	消费与接收具有相似的含义。消费者大多时候是一个等待接收消息的程序。请注意生产者，消费者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。

![image-20220317193104586](img.assets\image-20220317193104586.png)



# 三.RabbitMQ的核心部分

![image-20220317193254992](img.assets\image-20220317193254992.png)

1.Hello World : 简单模式

2.Work queues: 工作模式

3.Pushlish/Subscribe: 发布订阅模式

4.Routing：路由模式

5.Topics：主题模式

6.Publisher Confirms：发布确认模式



# 四.RabbitMQ的工作原理

![image-20220317193649798](img.assets\image-20220317193649798.png)

**`Producer`**:生产者。



**`Connection`**：publisher／consumer 和 broker 之间的 TCP 连接。



**`Channel（信道）`**：如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCPConnection 的开销将是巨大的，效率也较低。Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。**Channel 作为轻量级的Connection 极大减少了操作系统建立 TCP connection 的开销** 



**`Broker`**：接收和分发消息的应用(MQ的服务器，消息实体)，RabbitMQ Server 就是 Message Broker.



**`Exchange(交换机)`**：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到 queue 中去。常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout(multicast)



**`Queue`**：消息最终被送到这里等待 consumer 取走



**`Binding`**：exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key，Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据



**`Consumer`：**消费者



# 五.简单模式

“ P”是我们的生产者，“ C”是我们的消费者。中间的框是一个队列-RabbitMQ 代表使用者保留的消息缓冲区

![image-20220318142320412](img.assets\image-20220318142320412.png)

## 1.相关依赖

```xml
<!--指定 jdk 编译版本-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
    <!--rabbitmq 依赖客户端-->
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.8.0</version>
        </dependency>
    <!--操作文件流的一个依赖-->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>
    </dependencies>
```

## 2.生产者

相关方法说明

**在服务器中开发5672端口**

创建一个队列

```java
queueDeclare()
//主动声明一个服务器命名的独占、自动删除、非持久队列。
//新队列的名称保存在 AMQP.Queue.DeclareOk 结果的“queue”字段中。
//返回：一个声明确认方法，表明队列已成功声明

queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete,
                                 Map<String, Object> arguments)
//queue - 队列的名称 
//Durable - 如果我们声明一个持久队列（该队列将在服务器重新启动后继续存在），则为 true
//autoDelete - 如果我们声明一个独占队列，则为真（仅限于此连接)
//exclusive-声明一个自动删除队列（服务器将在不再使用时将其删除）
//arguments——队列的其他属性（构造参数）
//返回：一个声明确认方法，表明队列已成功声明
```

发送消息方法

```java
basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body);
//exchange - 将消息发布到的交换机
//routingKey -路由键
//props - 消息的其他属性 - 路由标头等
//body -消息体

basicPublish(String exchange, String routingKey, boolean mandatory, BasicProperties 													props, byte[] body);
// mandatory-如果要设置“强制”标志，则为 true
 
basicPublish(String exchange, String routingKey, boolean mandatory, boolean immediate, 				  BasicProperties props, byte[] body);
//immediate-如果要设置“立即”标志，则为 true。请注意，RabbitMQ 服务器不支持此标志。
```

### 发送消息实现

```java
//生产者---发消息，将消息发送给队列
public class Producer {
    
    //队列名称
    private static final String QUEUE_NAME = "hello";

    //发消息
    public static void main(String[] args) throws TimeoutException, IOException {
        //创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置工厂ip 连接RabbitMQ队列 端口默认5672
        factory.setHost("121.41.112.246");
        //设置用户名
        factory.setUsername("root");
        //设置密码
        factory.setPassword("root");
        //创建连接
        Connection connection = factory.newConnection();
        //获取信道
        Channel channel = connection.createChannel();
        //创建一个队列
        //1.第一个参数：队列的名称
        //2.第二个参数：队列是否持久化，默认情况下存储在内存中,持久化储存在磁盘中
        // 如果我们声明一个持久队列，则为 true（该队列将在服务器重新启动后继续存在
        //3.第三个参数:声明一个独占队列，设置为true则只能使用该信道进行接收消息
        //4.第四个参数:声明一个自动删除队列，则为 true（服务器将在不再使用时将其删除）
        //5.第五个参数:队列的其他属性（构造参数）
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        //发消息
        String message = "hello world";
        //第一个参数：表示发送在哪个交换机 空串表示一个默认的交换机
        //第二个参数：路由的key值,本次是队列的名称
        //第三个参数:消息的其他属性 - 路由标头等
        //第四个参数:发送消息的消息体-使用二进制发送
        channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
        System.out.println("消息发送完毕");
    }
}
```

### 在客户端中查看

![image-20220318152227780](img.assets\image-20220318152227780.png)

![image-20220318152343123](img.assets\image-20220318152343123.png)

## 3.消费者

获取消息的方法

```java
basicConsume(String queue, boolean autoAck, DeliverCallback deliverCallback, CancelCallback cancelCallback)；
//queue – 队列的名称
//autoAck – 如果服务器应考虑消息在传递后确认，则为 true；如果服务器应该期待明确的确认，则返回 false
//deliverCallback – 消息传递时的回调（消息成功收到后将消息保存在此接口的回调中）
//cancelCallback – 消费者取消时的回调

```

 DeliverCallback接口  **message中储存接收到的消息 **,收到消息后用来处理消息的回调对象参数

```java
@FunctionalInterface
public interface DeliverCallback {
    void handle(String consumerTag, Delivery message) throws IOException;}
```

CancelCallback接口-消费者取消时的回调

```java
@FunctionalInterface
public interface CancelCallback {
    void handle(String consumerTag) throws IOException;}
```

### 接收消息的实现

```java
//消费者，接收消息的
public class Consumer {
    //队列名称
    private static final String QUEUE_NAME = "hello";
    public static void main(String[] args) throws TimeoutException, IOException {
        //创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置工厂的ip
        factory.setHost("121.41.112.246");
        //设置用户名和密码
        factory.setUsername("root");
        factory.setPassword("root");
        //创建连接
        Connection connection = factory.newConnection();
        //创建信道
        Channel channel = connection.createChannel();
        //使用信道来接收消息
        //参数信息：
        //第一个参数：消费哪个队列
        //第二个参数：消费成功后是否要自动应答
        //第三个参数：消费者为成功消费的回调,消费者对象的接口
        //          处理推送过来的消息的回调函数
        //第四个参数：消费者取消消费的回调
        DeliverCallback deliverCallback = (consumerTag, message) ->
                    System.out.println(new String(message.getBody()));
        //deliverCallback，推送的消息如何进行消费的接口回调
        CancelCallback cancelCallback = System.out::println;
        System.out.println("接收消息");
        channel.basicConsume(QUEUE_NAME, true, deliverCallback,cancelCallback);
    }
}
```



# 六.Work Queues

​	工作队列(又称任务队列)的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。相反我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行作业。当有多个工作线程时，这些工作线程将一起处理这些任务。

![image-20220405143751213](img.assets\image-20220405143751213.png)

## 1.轮询分发消息

### 1.创建工具类

```java
public class RabbitMQUtils {
    public static Channel getChannel() throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("124.221.229.229");
        connectionFactory.setUsername("root");
        connectionFactory.setPassword("root");
        Connection connection = connectionFactory.newConnection();
        return connection.createChannel();
    }
```

### 2.生产者

```java
public class Producer {
    //队列名称
    private static final String QUEUE_NAME = "HELLO";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        //创建队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        //返送消息
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String next = scanner.next();
            channel.basicPublish("",QUEUE_NAME,null,next.getBytes());
            System.out.println("发送的消息:"+next);
        }
    }
}
```

### 3.消费者

```java
//一个工作线程，相当于一个消费者
public class Work01 {

    //队列名称
    private static final String QUEUE_NAME = "HELLO";

    //接收消息
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        DeliverCallback deliverCallback = (consumerTag,message) -> System.out.println("接收到的消息"+new String(message.getBody()));
        CancelCallback cancelCallback = System.out::println;
        System.out.println("c2等待消息的接收");
        channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
    }
}
```

开启多工作线程

![image-20220405151153978](img.assets\image-20220405151153978.png)

### 4.结果

![image-20220405151337009](img.assets\image-20220405151337009.png)



## 2.消息应答

​	消息应答就是:**消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了。**保证消息不丢失



### 1.自动应答(尽量少使用)

​	消息发送后立即被认为已经传送成功，这种模式需要在**高吞吐量和数据传输安全性方面做权衡**,因为这种模式如果消息在接收到之前，消费者那边出现连接或者 channel 关闭，那么消息就丢失 了,当然另一方面这种模式消费者那边可以传递过载的消息，**没有对传递的消息数量进行限制**，当然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使得内存耗尽，最终这些消费者线程被操作系统杀死，**所以这种模式仅适用在消费者可以高效并以某种速率能够处理这些消息的情况下使用**。



### 2.手段应答

#### 1.手动应答的方法

```java
//1.deliveryTag 消息的标识 -- message.getEnvelope().getDeliveryTag()
//2.multiple 是否批量应答
void basicAck(long deliveryTag, boolean multiple) throws IOException;
Channel.basicAck(用于肯定确认)  //RabbitMQ 已知道该消息并且成功的处理消息，可以将其丢弃

void basicNack(long deliveryTag, boolean multiple, boolean requeue) throws IOException;
Channel.basicNack(用于否定确认) 

void basicReject(long deliveryTag, boolean requeue) throws IOException;
Channel.basicReject(用于否定确认) //与 Channel.basicNack 相比少一个参数,不处理该消息了直接拒绝，可以将其丢弃
```

#### 2.Multiple参数

**手动应答的好处是可以批量应答并且减少网络拥堵,建议不要批量处理，既将multiple=false**

<img src="img.assets\image-20220405155305116.png" alt="image-20220405155305116" style="zoom: 50%;" />

#### 3.消息自动重新入队

​		如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息未发送 ACK 确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息。

![image-20220405155845391](img.assets\image-20220405155845391.png)

#### 4.手动应答的实现

```java
public class Work01 {
    //队列名称
    private static final String QUEUE_NAME = "HELLO";
    //接收消息
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        DeliverCallback deliverCallback = (consumerTag,message) -> {
            System.out.println("睡眠一秒后应答");
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("接收到的消息:"+new String(message.getBody()));
            //1.deliveryTag 消息的标识
            //2.multiple 是否批量应答
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
        };
        CancelCallback cancelCallback = System.out::println;
        System.out.println("c1等待消息的接收");
        channel.basicConsume(QUEUE_NAME,false,deliverCallback,cancelCallback);
    }
}
```



## 3.**RabbitMQ** 持久化

​	将RabbitMQ的消息保存在磁盘中，防止因为重启RabbitMQ导致消息丢失



### 1.队列实现持久化

​	需要在声明队列的时候把 durable 参数设置为持久化

```java
//创建队列
channel.queueDeclare(QUEUE_NAME,durable:true,false,false,null);
```

但是需要注意的就是如果之前声明的队列不是持久化的，需要把原先队列先删除，或者重新创建一个持久化的队列，不然就会出现错误

![image-20220405164445790](img.assets\image-20220405164445790.png)

在控制台中可以显示队列是否持久化

<img src="img.assets\image-20220405164541609.png" alt="image-20220405164541609" style="zoom: 80%;" />



### 2.消息实现持久化

要想让消息实现持久化需要在消息生产者修改代码

在发送消息时将**props参数设置为`MessageProperties.PERSISTENT_TEXT_PLAIN`** 

```java
channel.basicPublish("",QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN,next.getBytes());
```

​	**将消息标记为持久化并不能完全保证不会丢失消息。**尽管它告诉 RabbitMQ 将消息保存到磁盘，但是这里依然存在当消息刚准备存储在磁盘的时候 但是还没有存储完，消息还在缓存的一个间隔点。此时并没有真正写入磁盘，持久性保证并不强。



### 3.不公平分发

​	RabbitMQ 分发消息采用的轮询分发，为了避免有工作队列长期处于空闲的情况，我们可以设置参数 **channel.basicQos(1);**改为不公平分发（**设置信道容量的大小，采取轮询的方式往信道中放消息，信道满了就跳过**），轮询分发为channel.basicQos(0)。

**由消费者进行设置**

```java
void basicQos(int prefetchCount) throws IOException; 
// prefetchCount : 服务器将传递的最大消息数，如果无限制则为 0
//改为不公平分发
channel.basicQos(1);
```

在客户端的channels中可以查看

<img src="img.assets\image-20220405170228967.png" alt="image-20220405170228967" style="zoom: 67%;" />

### 4.**预取值** 

提前设置信道的容量大小

本身消息的发送就是异步发送的，所以在任何时候，channel 上肯定不止只有一个消息另外来自消费者的手动确认本质上也是异步的。因此这里就存在一个未确认的消息缓冲区，因此希望开发人员能**限制此缓冲区的大小，以避免缓冲区里面无限制的未确认消息问题**。这个时候就可以通过使用 basic.qos 方法设置“预取计数”值来完成的。**该值定义通道上允许的未确认消息的最大数量**。

![image-20220405172848917](img.assets\image-20220405172848917.png)

# 

# 七.发布确认

## 1.发布确认原理

 生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式，**所有在该信道上面发布的消息都将会被指派一个唯一的 ID**(从 1 开始)，一旦消息被投递到所有匹配的队列之后，broker 就会发送一个确认给生产者(包含消息的唯一 ID)，这就使得生产者知道消息已经正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出，broker 回传给生产者的确认消息中 delivery-tag 域包含了确认消息的序列号，此外 broker 也可以设置basic.ack 的multiple 域，表示到这个序列号之前的所有消息都已经得到了处理。



## 2.发布确认的策略

**在生产者端开启**

### 1.开启发布确认的方法

发布确认默认是没有开启的，如果要开启需要调用方法 `confirmSelect`，每当你要想使用发布确认，都需要在 channel 上调用该方法

```java
Confirm.SelectOk confirmSelect() throws IOException;
```

等待消息确定发布

```java
boolean waitForConfirms() throws InterruptedException;
```

消息发布异步监听器

```java
ConfirmListener addConfirmListener(ConfirmCallback ackCallback, ConfirmCallback nackCallback);
//ackCallback 消息发布成功的回调
//nackCallback 消息发布失败的回调
```



### 2.单个确认发布

​	这是一种简单的确认方式，它是一种**同步确认发布**的方式，也就是发布一个消息之后只有它被确认发布，后续的消息才能继续发布,waitForConfirmsOrDie(long)这个方法只有在消息被确认的时候才返回，如果在指定时间范围内这个消息没有被确认那么它将抛出异常。这种确认方式有一个最大的缺点就是:**发布速度特别的慢，**因为如果没有确认发布的消息就会阻塞所有后续消息的发布，这种方式最多提供每秒不超过数百条发布消息的吞吐量。

```java
public static void oneMessagePublish() throws IOException, TimeoutException, InterruptedException {
        Channel channel = RabbitMQUtils.getChannel();
        //开启发布确认
        channel.confirmSelect();
        //创建队列
        String uuid = UUID.randomUUID().toString();
        channel.queueDeclare(uuid,false,false,false,null);
        //发送消息
        long start = System.currentTimeMillis();
        for (int i = 0; i < MAX_MESSAGE_NUMBER; i++) {
            channel.basicPublish("",uuid,null,(i+"").getBytes());
            //等待消息确定发布
            channel.waitForConfirms();
            //服务端返回 false 或超时时间内未返回，生产者可以消息重发
        }
        long end = System.currentTimeMillis();
        System.out.println("发布"+MAX_MESSAGE_NUMBER+"消息所需要的时间:"+(end-start));//80832ms
}
```



### 3.批量确认发布

上面那种方式非常慢，与单个等待确认消息相比，先发布一批消息然后一起确认可以极大地提高吞吐量，当然这种方式的缺点就是:当发生故障导致发布出现问题时，不知道是哪个消息出现问题了，我们必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。当然这种方案仍然是同步的，也一样阻塞消息的发布。

```java
public static void batchMessagePublish() throws IOException, TimeoutException, InterruptedException {
        Channel channel = RabbitMQUtils.getChannel();
        //开启发布确认
        channel.confirmSelect();
        //设置批量确认消息的大小
        int batchSize = 100;
        //创建队列
        String uuid = UUID.randomUUID().toString();
        channel.queueDeclare(uuid,false,false,false,null);
        //发送消息
        long start = System.currentTimeMillis();
        for (int i = 0; i < MAX_MESSAGE_NUMBER; i++) {
            channel.basicPublish("",uuid,null,(i+"").getBytes());
            //等待消息确定发布
            if((i+1)%batchSize==0){
                //发布100条确认一次
               channel.waitForConfirms();
            }

        }
        long end = System.currentTimeMillis();
        System.out.println("发布"+MAX_MESSAGE_NUMBER+"消息所需要的时间:"+(end-start));//1145ms
}
```



### 4.异步确认发布

​	异步确认发布是性价比最高的，是利用回调函数来达到消息可靠性传递的，这个中间件也是通过函数回调来保证是否投递成功。

<img src="img.assets\image-20220406144813569.png" alt="image-20220406144813569;" style="zoom: 67%;" />

生产者发送消息到broker，发生确认回调，broker把消息投递到队列，发生失败回调

该方式确认发布不用等待消息是否确定发布（**不调用waitForConfirms()方法**），开发者只用发布消息就行

```java
public static void asyncMessagePublish()throws IOException, TimeoutException{
        Channel channel = RabbitMQUtils.getChannel();
        //开启发布确定
        channel.confirmSelect();
        String uuid = UUID.randomUUID().toString();
        channel.queueDeclare(uuid,false,false,false,null);

        //准备一个线程安全有序的哈希表，适用于高并发的情况下
        //1.将序号和消息经行关联
        //2.可以批量删除条数，主需要给到序列号
        //3.支持高并发
        ConcurrentSkipListMap<Long,String> concurrentSkipListMap = new ConcurrentSkipListMap<>();

        //消息确定成功回调函数
        //deliveryTag:消息的编号
        //multiple:是否批量确定消息
        ConfirmCallback ackCallback = (deliveryTag,multiple)->{
            //3.删除掉已经确认的消息
            //单个删除
            concurrentSkipListMap.remove(deliveryTag);
            //批量删除
            if(multiple){
                //headMap方法得到传进去的key到第一个key的组成的所有key值
                ConcurrentNavigableMap<Long, String> ConcurrentNavigableMap
                        = concurrentSkipListMap.headMap(deliveryTag, true);
                ConcurrentNavigableMap.clear();
            }
        };
        //消息确定失败回调函数
        ConfirmCallback nackCallback = (deliveryTag,multiple)->{
            //4.记录发布失败的消息
            System.out.println("未监听到的消息:"+concurrentSkipListMap.get(deliveryTag));
        };
    
        //准备消息的监听器，监听哪些消息发布成功了
        channel.addConfirmListener(ackCallback,nackCallback);//异步监听

        //发送消息
        long start = System.currentTimeMillis();
        String message = "";
        for (int i = 0; i < MAX_MESSAGE_NUMBER; i++) {
            message = i+"";
            //1.此处记录下所有要发布的消息
            //channel.getNextPublishSeqNo() 下一条要发布的消息的序号
            concurrentSkipListMap.put(channel.getNextPublishSeqNo(),message);
            //2.发送消息
            channel.basicPublish("",uuid,null,message.getBytes());
        }
        long end = System.currentTimeMillis();
        System.out.println("发布"+MAX_MESSAGE_NUMBER+"消息所需要的时间:"+(end-start));//170ms
    }
```

#### 4.1如何处理异步未确认消息

​	最好的解决的解决方案就是把未确认的消息放到一个基于内存的能被发布线程访问的队列，比如说用 ConcurrentSkipListMap 这个哈希表在 confirm callbacks 与发布线程之间进行消息的传递。

### 5. 3 种发布确认速度对比

单独发布消息

​		同步等待确认，简单，但吞吐量非常有限。

批量发布消息

​		批量同步等待确认，简单，合理的吞吐量，一旦出现问题但很难推断出是那条消息出现了问题。

异步处理：

​		最佳性能和资源使用，在出现错误的情况下可以很好地控制



### 6.如何尽可以保证消息不丢失 

​	1.将队列实例化

​	2.将消息实例化

​	3.发布确认



# 八.交换机

## 1.Exchanges

**生产者发生消息到交换机，交换机将消息给不同的队列中，可以实现同一个消息被多个消费者使用**(一个队列中的消息只能消费一次)

RabbitMQ 消息传递模型的核心思想是: **生产者生产的消息从不会直接发送到队列**。实际上，通常生产者甚至都不知道这些消息传递传递到了哪些队列中。

相反，**生产者只能将消息发送到交换机(exchange)**，交换机工作的内容非常简单，一方面它接收来自生产者的消息，另一方面将它们推入队列。交换机必须确切知道如何处理收到的消息。是应该把这些消息放到特定队列还是说把他们到许多队列中还是说应该丢弃它们。这就的由交换机的类型来决定。

![image-20220406153853752](img.assets\image-20220406153853752.png)

**声明交换机**

```java
Exchange.DeclareOk exchangeDeclare(String exchange, String type) throws IOException;
//exchange 交换机的名字
//type 交换机的类型,以下四种类型(direct,topic,headers,fanout)
```

**绑定队列**

```java
Queue.BindOk queueBind(String queue, String exchange, String routingKey) throws IOException;
//queue 队列的名字
//exchange 交换机的名字
//routingKey 路由的key
```



### 1.1.Exchanges 的类型

​	总共有以下类型：

​		直接(direct)(路由类型),

​		主题(topic) ,

​		标题(headers)，--不常用

​		扇出(fanout)(发布订阅类型)，

### 1.2.无名exchange

```java
void basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body) throws IOException;
```

​	第一个参数是交换机的名称。**空字符串表示默认或无名称交换机**：消息能路由发送到队列中其实是由 routingKey(bindingkey)绑定 key 指定的，如果它存在的话。

![image-20220406154556984](img.assets\image-20220406154556984.png)

### 1.3 **临时队列** 

​		每当我们连接到 Rabbit 时，我们都需要一个全新的空队列，为此我们可以创建一个具有**随机名称的队列**，或者能让服务器为我们选择一个随机队列名称那就更好了。其次**一旦我们断开了消费者的连接，队列将被自动删除**。

创建临时队列的方式如下:

```java
String queueName = channel.queueDeclare().getQueue();
```

![image-20220406155125886](img.assets\image-20220406155125886.png)

### 1.4 绑定(bindings)

binding 其实是 exchange 和 queue 之间的桥梁，它告诉我们 exchange 和那个队列进行了绑定关系。比如说下面这张图告诉我们的就是 X 与 Q1 和 Q2 进行了绑定

![image-20220406155236843](img.assets\image-20220406155236843.png)

## 2.Fanout交换机 

fanout(发布订阅模式)是将接收到的所有消息**广播**(**一个人发，其它绑定的队列都可以收到**)到它知道的所有队列中。系统中默认有些 exchange 类型

![image-20220406155422501](img.assets\image-20220406155422501.png)

**交换机要在消费者声明（最好在两端都声明），防止因先启动消费者，找不到交换机保存。**

**在fanout模式下，routingKey可以被忽略,只要队列绑定的交换机相同，就可以接收到消息**。



生产者端：

```java
private static final String EXCHANGE_NAME = "logs";

public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        //声明交换机
        //1.exchange: 交换机的名字
        //2.type: 交换机的类型
        channel.exchangeDeclare(EXCHANGE_NAME,BuiltinExchangeType.FANOUT);
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String message = scanner.next();
            //发生消息
            //使用fanout模式发生消息生产者不需要创建队列，由消费者创建队列，并且绑定
            //fanout模式是先启动消费者端，若先启动生产者端，则消息会被直接舍弃
            //fanout模式将消息发布在所有和该交换机绑定的队列上
            channel.basicPublish(EXCHANGE_NAME,"",null,message.getBytes(StandardCharsets.UTF_8));
            System.out.println("发出消息:"+message);
        }
}
```

消费者端:

```java
private static final String EXCHANGE_NAME = "logs";

public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        //声明交换机
        //1.exchange: 交换机的名字
        //2.type: 交换机的类型
        channel.exchangeDeclare(EXCHANGE_NAME,BuiltinExchangeType.FANOUT);
        //生成一个临时队列
        String queueName = channel.queueDeclare().getQueue();
        //绑定交换机
        channel.queueBind(queueName,EXCHANGE_NAME,"");
        //接收消息
        DeliverCallback deliverCallback = (consumerTag,message)->{
            System.out.println(new String(message.getBody()));
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
        };
        CancelCallback cancelCallback = System.out::println;
        System.out.println("消费者1接收消息");
        channel.basicConsume(queueName,false,deliverCallback,cancelCallback);
    }
```



## 3.Direct交换机

​	绑定是交换机和队列之间的桥梁关系。也可以这么理解：**队列只对它绑定的交换机的消息感兴趣**。绑定用参数：routingKey 来表示也可称该参数为 binding key，创建绑定我们用代码:channel.queueBind(queueName, EXCHANGE_NAME, "routingKey");**绑定之后的意义由其交换类型决定**

![image-20220406164835042](img.assets\image-20220406164835042.png)



生产者:

```java
private static final String EXCHANGE_NAME = "logs";
public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        //声明交换机
        //1.exchange: 交换机的名字
        //2.type: 交换机的类型
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String message = scanner.next();
            //发送消息
            //direct根据和交换机绑定的队列的routingKey来将消息发生在不同的队列中
            channel.basicPublish(EXCHANGE_NAME,"consumer1",null,message.getBytes(StandardCharsets.UTF_8));
            System.out.println("发出消息:"+message);
        }
    }
```



消费者:

```java
private static final String EXCHANGE_NAME = "logs";
public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        //声明交换机
        //1.exchange: 交换机的名字
        //2.type: 交换机的类型
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        //生成一个临时队列
        String queueName = channel.queueDeclare().getQueue();
        //绑定交换机,绑定交换机和队列之间的routingKey
        channel.queueBind(queueName,EXCHANGE_NAME,"consumer1");
        //接收消息
        DeliverCallback deliverCallback = (consumerTag,message)->{
            System.out.println(new String(message.getBody()));
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
        };
        CancelCallback cancelCallback = System.out::println;
        System.out.println("消费者1接收消息");
        channel.basicConsume(queueName,false,deliverCallback,cancelCallback);
    }
```



## 4.Topics交换机		

​		**路由的模糊绑定**

​		发送到类型是 topic 交换机的消息的 routing_key 不能随意写，必须满足一定的要求，它**必须是一个单词列表，以点号分隔开**。这些单词可以是任意单词，比如说："stock.usd.nyse", "nyse.vmw","quick.orange.rabbit".这种类型的。当然这个单词列表最多不能超过 255 个字节。

​	在这个规则列表中，其中有两个替换符需要注意:

​		1.***(星号)可以代替一个单词**

​		2.**#(井号)可以替代零个或多个单词**

**当一个队列绑定键是*#,那么这个队列将接收所有数据，就有点像fanout了**

**如果队列绑定键当中没有#和*出现，那么该队列绑定类型就是 direct 了**

例如：

Q1-->绑定的是

​		中间带 orange 带 3 个单词的字符串(*.orange.*)

Q2-->绑定的是

​		最后一个单词是 rabbit 的 3 个单词(*.*.rabbit)

​		第一个单词是 lazy 的多个单词(lazy.#)

![image-20220406171001295](img.assets\image-20220406171001295.png)

生产者:

```java
private static final String EXCHANGE_NAME = "logs";

public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        //声明交换机
        //1.exchange: 交换机的名字
        //2.type: 交换机的类型
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String message = scanner.next();
            //发送消息
            //topic根据和交换机绑定的队列的routingKey的动态形式来将消息发生在不同的队列中
            channel.basicPublish(EXCHANGE_NAME,"11.consumer.xy",null,message.getBytes(StandardCharsets.UTF_8));
            System.out.println("发出消息:"+message);
        }
    }
```

消费者:

```java
private static final String EXCHANGE_NAME = "logs";

public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        //声明交换机
        //1.exchange: 交换机的名字
        //2.type: 交换机的类型
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
        //生成一个临时队列
        String queueName = channel.queueDeclare().getQueue();
        //绑定交换机,绑定交换机和队列之间的routingKey
        channel.queueBind(queueName,EXCHANGE_NAME,"*.consumer.*");
        //接收消息
        DeliverCallback deliverCallback = (consumerTag,message)->{
            System.out.println(new String(message.getBody()));
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
        };
        CancelCallback cancelCallback = System.out::println;
        System.out.println("消费者1接收消息");
        channel.basicConsume(queueName,false,deliverCallback,cancelCallback);
    }
```

# 

# 九.死信队列

​		死信，顾名思义就是无法被消费的消息，字面意思可以这样理解，一般来说，producer 将消息投递到 broker 或者直接到queue 里了，consumer 从 queue 取出消息进行消费，但某些时候由于特定的**原因导致** **queue** **中的某些消息无法被消费**，这样的消息如果没有后续的处理，就变成了死信，有死信自然就有了死信队列。

​	应用场景:为了保证订单业务的消息数据不丢失，需要使用到 RabbitMQ 的死信队列机制，当消息消费发生异常时，将消息投入死信队列中。还有比如说: 用户在商城下单成功并点击去支付后在指定时间未支付时自动失效。



## 1.死信的来源

​	1.消息 TTL 过期

​	2.队列达到最大长度(队列满了，无法再添加数据到 mq 中)

​	3.消息被拒绝(basicReject 或 basicNack)并且 requeue=false

<img src="img.assets\image-20220406180030525.png" alt="image-20220406180030525" style="zoom:80%;" />



## 2.死信构建

死信队列的相关设置:

```java
//正常队列绑定死信队列消息
Map<String, Object> arguments = new HashMap<>();
//正常队列设置死信交换机 参数key是固定值:x-dead-letter-exchange
arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE_NAME);
//正常队列设置死信 routing-key 参数key是固定值:x-dead-letter-routing-key
arguments.put("x-dead-letter-routing-key", "liSi");
//设置过期时间,可以由生产者设置
arguments.put("x-message-ttl",10000);
//设置队列的最大长度
arguments.put("x-max-length",6);
```



生产者：

```java
private final static String EXCHANGE_NAME = "normal_exchange";

public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        Scanner scanner = new Scanner(System.in);
        String message = "";
        while (scanner.hasNext()){
            message = scanner.next();
            //设置过期时间 单位是毫秒
            AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().expiration("10000").build();
            channel.basicPublish(EXCHANGE_NAME,"zhangSan",properties,message.getBytes(StandardCharsets.UTF_8));
            System.out.println("发生的消息:"+message);
        }
    }
```



消费者1：设置死信队列

```java

    private final static String EXCHANGE_NAME = "normal_exchange";

    private final static String DEAD_EXCHANGE_NAME = "dead_exchange";

    private final static String QUEUE_NAME = "normal_queue";

    private final static String DEAD_QUEUE_NAME = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = RabbitMQUtils.getChannel();

        //声明正常交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        //声明死信交换机
        channel.exchangeDeclare(DEAD_EXCHANGE_NAME,BuiltinExchangeType.DIRECT);

        //正常队列绑定死信队列消息
        Map<String, Object> arguments = new HashMap<>();
        //正常队列设置死信交换机 参数key是固定值:x-dead-letter-exchange
        arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE_NAME);
        //正常队列设置死信 routing-key 参数key是固定值:x-dead-letter-routing-key
        arguments.put("x-dead-letter-routing-key", "liSi");
        //设置过期时间,可以由生产者设置
        arguments.put("x-message-ttl",10000);
        //设置队列的最大长度
        arguments.put("x-max-length",6);

        //声明正常队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,arguments);
        //绑定正常队列
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"zhangSan");

        //声明死信队列
        channel.queueDeclare(DEAD_QUEUE_NAME,false,false,false,null);
        //绑定死信队列
        channel.queueBind(DEAD_QUEUE_NAME,DEAD_EXCHANGE_NAME,"liSi");

        //接收消息
        DeliverCallback deliverCallback = (consumerTag,message) ->{
            String msg = new String(message.getBody());
            System.out.println(msg);
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
            //拒绝接收消息
            if("111".equals(msg)){
                //requeue:如果被拒绝的消息应该重新排队而不是丢弃，则为 true
                //如果为false，则该消息成为死信消息
                channel.basicNack(message.getEnvelope().getDeliveryTag(),false,false);
            }

        };
        CancelCallback cancelCallback = System.out::println;
        channel.basicConsume(QUEUE_NAME,false,deliverCallback,cancelCallback);
    }
```



# 十.SpringBoot整合RabbitMQ

```xml
 <!--rabbitMQ起步依赖-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<!--rabbitMQ测试依赖   -->
<dependency>
  <groupId>org.springframework.amqp</groupId>
  <artifactId>spring-rabbit-test</artifactId>
  <scope>test</scope>
</dependency>
```

## 1.在配置文件中进行配置

```properties
#rabbitmq配置
spring.rabbitmq.host=124.221.229.229
spring.rabbitmq.port=5672
spring.rabbitmq.username=root
spring.rabbitmq.password=root
```

## 2.交换机和队列的配置类

```java
@Configuration
public class RabbitMQConfig {
    //direct交换机
    @Bean
    public DirectExchange myDirectExchange(){
        // 参数意义:
        // name: 名称
        // durable: 是否持久化
        // autoDelete: 自动删除
        //arguments: 交换机的相关设置
        return new DirectExchange("myDirectExchange", true, false);
    }

    //topic交换机
    @Bean
    public TopicExchange myTopicExchange(){
        return new TopicExchange("myTopicExchange", true, false);
    }

    //fanout交换机
    @Bean
    public FanoutExchange myFanoutExchange(){
        return new FanoutExchange("myFanoutExchange", true, false);
    }

    @Bean
    public Queue myQueue(){
        //name:名称
        // durable:是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
        // exclusive:默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
        // autoDelete:是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
        //arguments:队列的相关设置

        Map<String, Object> arguments = new HashMap<>();
        ///正常队列设置死信交换机 参数key是固定值:x-dead-letter-exchange
        arguments.put("x-dead-letter-exchange","myTopicExchange");
        //正常队列设置死信 routing-key 参数key是固定值:x-dead-letter-routing-key
        arguments.put("x-dead-letter-routing-key", "liSi");
        //设置过期时间,可以由生产者设置
        arguments.put("x-message-ttl",10000);
        //设置队列的最大长度
        arguments.put("x-max-length",6);
        return new Queue("myQueue",false,false,false,arguments);
        //或者
        //QueueBuilder.nonDurable("myQueue").ttl(10000)
        //.deadLetterExchange("myTopicExchange").deadLetterRoutingKey("liSi").build();
    }

    //绑定
    @Bean
    public Binding bindingDirectExchange(){
        //bind 绑定队列
        //to 将队列绑定到交换机
        //with 路由key
        return BindingBuilder.bind(myQueue()).to(myDirectExchange()).with("routingKey");
    }
}
```

### 2.1 @Qualifier注解

在使用@Autowire自动注入的时候，加上@Qualifier(“”)可以指定注入哪个对象；

```java
public Binding queueABindingXExchange(@Qualifier("queueA") Queue queueA,
                                      @Qualifier("xExchange") DirectExchange xExchange){
```



## 3.使用RabbitTemplate模板

​	Rabbitmq默认用的是SimpleMessageConverter()序列化,我们实际开发中发送接受的消息一般为json，我们可以换个序列化方式：我们看到`MessageConverter`接口的实现类有多个我们选择`Jackson2JsonMessageConverter`

### 3.1 配置序列化方式

使用 JSON 序列化与反序列化

```java
@Configuration
public class RabbitMQConfig {
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    } 
}
```

或者:

```java
 @Bean
public RabbitListenerContainerFactory<?> rabbitListenerContainerFactory(ConnectionFactory connectionFactory){
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setMessageConverter(new Jackson2JsonMessageConverter());
        return factory;
}
```



### 3.2 发送消息

发送消息的两种方式：

1.sent():Message需要自己构造

2.(常用) **convertAndSent()**: message自动序列化发送给MQ

```java
void convertAndSend(String exchange, String routingKey, Object message) throws AmqpException;
```

### 3.3 接收消息

接收消息的两种方式:

1.receive--将消息转换为Message

2.receiveAndConvert--将消息转换为Object

```java
public Object receiveAndConvert() throws AmqpException
public Object receiveAndConvert(String queueName) throws AmqpException
public Object receiveAndConvert(long timeoutMillis) throws AmqpException
public Object receiveAndConvert(String queueName, long timeoutMillis) throws AmqpException
```

### 3.4 设置监听

在一个方法上添加`@RabbitListener`参数为队列的名，方法参数就是收到的消息。

```java
@Service
public class MyRabbitmq {
    @RabbitListener(queues = "")
    public void doSomeThing(Message message){
        System.out.println("收到！"+new String(message.getBody));
    }
}
```

## 4.相关注解说明

### 4.1 @RabbitListener

使用 @RabbitListener 注解标记方法，当监听到队列 中有消息时则会进行接收并处理。

消息处理方法参数是由 MessageConverter 转化，若使用自定义 MessageConverter 则需要在 RabbitListenerContainerFactory 实例中去设置（默认 Spring 使用的实现是 SimpleRabbitListenerContainerFactory）

```java
public class MyRabbitmq {
    @RabbitListener(queues = "")
    public void doSomeThing(Message message){
        System.out.println("收到！"+new String(message.getBody));
    }
}
```

### 4.2 @Payload 与 @Headers，@Header

使用 @Payload 和 @Headers 注解可以消息中的 body 与 headers 信息

```java
@RabbitListener(queues = "debug")
public void processMessage1(@Payload String body, @Headers Map<String,Object> headers) {
    System.out.println("body："+body);
    System.out.println("Headers："+headers);
}
```

@Header 也可以获取单个 Header 属性

```java
@RabbitListener(queues = "debug")
public void processMessage1(@Payload String body, @Header String token) {
    System.out.println("body："+body);
    System.out.println("token："+token);
}
```

### 4.3 通过 @RabbitListener 注解声明 Binding

通过 @RabbitListener 的 bindings 属性声明 Binding（若 RabbitMQ 中不存在该绑定所需要的 Queue、Exchange、RoutingKey 则自动创建，若存在则抛出异常）

```java
@RabbitListener(bindings = @QueueBinding(
        exchange = @Exchange(value = "topic.exchange",durable = "true",type = "topic"),
        value = @Queue(value = "consumer_queue",durable = "true"),
        key = "key.#"
))
public void processMessage1(Message message) {
    System.out.println(message);
}
```

### 4.4 @RabbitListener 和 @RabbitHandler 搭配使用

**@RabbitListener 可以标注在类上面，需配合 @RabbitHandler 注解一起使用**
@RabbitListener 标注在类上面表示当有收到消息的时候，就交给 @RabbitHandler 的方法处理，具体使用哪个方法处理，根据 MessageConverter 转换后的参数类型

```java
@Component
@RabbitListener(queues = "consumer_queue")
public class Receiver {
    @RabbitHandler
    public void onMessage(@Payload String message){
        System.out.println("Message content : " + message);
    }
 
    @RabbitHandler
    public void onMessage(@Payload User user) {
        System.out.println("Message content : " + user);
    }
}
```



# 十一.延迟队列

​		延时队列,队列内部是有序的，最重要的特性就体现在它的延时属性上，延时队列中的元素是希望在指定时间到了以后或之前取出和处理，简单来说，**延时队列就是用来存放需要在指定时间被处理的元素的队列，本质上为死信队列中消息过期后的情况**。

## 1.延迟队列使用场景

​	1.订单在十分钟之内未支付则自动取消

​	2.新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。

​	3.用户注册成功后，如果三天内没有登陆则进行短信提醒。

​	4.用户发起退款，如果三天内没有得到处理则通知相关运营人员。

​	5.预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议

<img src="img.assets\image-20220407203834998.png" alt="image-20220407203834998" style="zoom:80%;" />

## 2.RabbitMQ中的 TTL

​	TTL 是 RabbitMQ 中一个消息或者队列的属性，表明一条消息或者该队列中的所有消息的最大存活时间,单位是毫秒。如果一条消息设置了 TTL 属性或者进入了设置TTL 属性的队列，那么这条消息如果在TTL 设置的时间内没有被消费，则会成为"死信"。如果同时配置了队列的TTL 和消息的TTL，那么较小的那个值将会被使用。

### 2.1 构建延迟队列

![image-20220408153223233](E:\笔记\MQ\RabbitMQ\img.assets\image-20220408153223233.png)

#### 2.1.1 构建交换机和队列

```java
@Configuration
public class TTLRabbitMQConfig {
    //正常交换机
    private static final String X_EXCHANGE = "X";
    //正常队列，绑定死信交换机和正常交换机
    private static final String QUEUE_A = "QA";
    private static final String QUEUE_B = "QB";

    //死信交换机
    private static final String Y_DEAD_LETTER_EXCHANGE = "Y";
    //死信队列
    private static final String DEAD_LETTER_QUEUE = "QD";

    //声明x交换机
    @Bean("xExchange")
    public DirectExchange xExchange(){
        return new DirectExchange(X_EXCHANGE,false,false);
    }

    //声明y交换机
    @Bean("yExchange")
    public DirectExchange yExchange(){
        return new DirectExchange(Y_DEAD_LETTER_EXCHANGE,false,false);
    }

    //声明队列QA，设置10s的过期时间，并且绑定到死信交换机
    @Bean("queueA")
    public Queue queueA(){
        //QueueBuilder.nonDurable(QUEUE_A).ttl(10000).deadLetterExchange(Y_DEAD_LETTER_EXCHANGE).deadLetterRoutingKey("YD").build();
        Map<String, Object> arguments = new HashMap<>(3);
        //将该队列绑定到死信交换机
        arguments.put("x-dead-letter-exchange",Y_DEAD_LETTER_EXCHANGE);
        //设置死信的路由key
        arguments.put("x-dead-letter-routing-key","YD");
        //设置TTL10s
        arguments.put("x-message-ttl",10000);
        return new Queue(QUEUE_A,false,false,false,arguments);
    }
    //绑定X交换机
    @Bean
    //自动装配时，可以在字段或参数上使用@Qualifier作为候选bean的限定符
    public Binding queueABindingXExchange(@Qualifier("queueA") Queue queueA,
                                          @Qualifier("xExchange") DirectExchange xExchange){
        return BindingBuilder.bind(queueA).to(xExchange).with("XA");
    }


    //声明队列QB,设置40S过期时间,并且绑定到死信交换机
    @Bean("queueB")
    public Queue queueB(){
        return QueueBuilder
                .nonDurable(QUEUE_B)
                .ttl(40000)
                .deadLetterExchange(Y_DEAD_LETTER_EXCHANGE)
                .deadLetterRoutingKey("YD")
                .build();
    }
    //绑定X交换机
    @Bean
    //形参的注入默认为byName
    public Binding queueBBindingXExchange(Queue queueB,DirectExchange xExchange){
        return BindingBuilder.bind(queueB).to(xExchange).with("XB");
    }

    //声明队列QD
    @Bean("queueD")
    public Queue queueD(){
        return new Queue(DEAD_LETTER_QUEUE,false,false,false,null);
    }
    //将QD队列绑定到死信交换机
    @Bean
    public Binding queueDBindingYExchange(Queue queueD,DirectExchange yExchange){
        return BindingBuilder.bind(queueD).to(yExchange).with("YD");
    }
}
```

#### 2.1.2 生产者

```java
@Resource
private RabbitTemplate rabbitTemplate;

//发送消息
@GetMapping("/send/{message}")
public void test1(@PathVariable String message){
        log.info("发送的消息:{}",message);
        rabbitTemplate.convertAndSend("X","XA","消息来自ttl为10秒的队列"+message);
        rabbitTemplate.convertAndSend("X","XB","消息来自ttl为40秒的队列"+message);
}
```

#### 2.1.3 接收消息

```java
//监听消息
@RabbitListener(queues = "QD")
public void listener(@Payload String message){
   log.info("接收的消息:{}",message);
}
```

#### 2.1.4 将队列设置为延迟队列的缺点

​	每增加一个新的时间需求，就要新增一个队列。大大增加队列数量



### 2.2 构建消息延迟

新增了一个队列 QC,绑定关系如下,该队列不设置TTL 时间

![image-20220408160754882](E:\笔记\MQ\RabbitMQ\img.assets\image-20220408160754882.png)

#### 2.2.1 构建交换机和队列

```java
private static final String QUEUE_C = "QC";
    
@Bean("queueC")
public Queue queueC(){
//该队列不设置过期时间，由生产者设置过期时间
return QueueBuilder.nonDurable(QUEUE_C).deadLetterExchange(Y_DEAD_LETTER_EXCHANGE)
                            .deadLetterRoutingKey("YD").build();
}
    
@Bean()
public Binding queueCBindingXExchange(Queue queueC,DirectExchange xExchange){
    return BindingBuilder.bind(queueC).to(xExchange).with("XC");
}

```

#### 2.2.2 生产者

```java
@GetMapping("/sendTTL/{msg}/{ttl}")
public void test2(@PathVariable String msg, @PathVariable String ttl){
    log.info("发送的消息:{},过期时间:{}",msg,ttl);
     //messagePostProcessor – 在发送消息之前应用到消息的处理器
        rabbitTemplate.convertAndSend("X","XC",msg+"---消息过期时间为"+ttl,message -> {
            //获取消息的参数的延迟时间
            message.getMessageProperties().setExpiration(ttl);
            return message;
        });
}
```

MessagePostProcesso接口

```java
@FunctionalInterface
public interface MessagePostProcessor {
	Message postProcessMessage(Message message) throws AmqpException;
}
```

#### 2.2.3 结果

![image-20220408163015440](E:\笔记\MQ\RabbitMQ\img.assets\image-20220408163015440.png)

#### 2.2.4 问题

​		**RabbitMQ只会检查第一个消息是否过期**，如果过期则丢到死信队列，**如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行。**



### 2.3 基于插件的延迟队列

#### 2.3.1 安装延时队列插件

 在RabbitMQ官网下载**rabbitmq_delayed_message_exchange**插件**,该插件要与RabbitMQ的版本保持一致**,然后解压放置到 RabbitMQ 的插件目录(plugins)

进入 RabbitMQ 的安装目录下的 plgins 目录，执行下面命令让该插件生效，然后重启 RabbitMQ

```shell
cd /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.3/plugins
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

重启RabbitMQ

```shell
service rabbitmq-server restart
```

在客户端新建交换机出现以下选项，则说明安装成功

![image-20220408170419766](E:\笔记\MQ\RabbitMQ\img.assets\image-20220408170419766.png)

#### 2.3.2 基于插件延迟示意图

**该插件将延迟的位置变成了交换机**,设置交换机发送消息给队列的时间

![image-20220408170939269](img.assets\image-20220408170939269.png)

#### 2.3.3 构建插件延迟队列

![image-20220408172524235](img.assets\image-20220408172524235.png)

##### 2.3.3.1 构建交换机和队列

```java
@Configuration
public class PluginDelayConfig {

    private static final String EXCHANGE_NAME = "delayed.exchange";
    private static final String QUEUE_NAME = "delayed.queue";
    private static final String ROUTING_KEY = "delayed.routingKey";

    //声明自定义交换机
    @Bean
    public CustomExchange  delayedExchange(){
        //自定义交换机参数
        Map<String, Object> arguments = new HashMap<>();
        //延迟类型为直接类型 ----将消息发送到队列的方式
        arguments.put("x-delayed-type","direct");
        //将type替换为插件中的类型 ---- 将消息返送到队列的时机
        return new CustomExchange(EXCHANGE_NAME,"x-delayed-message",false,false,arguments);
    }
    //声明队列
    @Bean
    public Queue delayedQueue(){
        //使用插件延迟队列不需要绑定死信队列
        return QueueBuilder.nonDurable(QUEUE_NAME).build();
    }
    //绑定交换机
    @Bean
    public Binding delayedQueueBindingDelayedExchange(Queue delayedQueue,CustomExchange delayedExchange){
        //noargs 构建方法 在交换机是自定义交换机时使用
        return BindingBuilder.bind(delayedQueue).to(delayedExchange).with(ROUTING_KEY).noargs();
    }
}
```

##### 2.3.3.2 发送消息和接收消息

```java
 //基于插件延迟队列
    @GetMapping("/delayedSend/{msg}/{ttl}")
    public void test3(@PathVariable String msg, @PathVariable Integer ttl){
        log.info("发送的消息:{},过期时间:{}",msg,ttl);
        rabbitTemplate.convertAndSend("delayed.exchange","delayed.routingKey",msg+"---消息过期时间为"+ttl,
                message -> {
                    //设置交换机发送消息的时间
                    message.getMessageProperties().setDelay(ttl);
                    return message;
                });

    }//监听消息
    @RabbitListener(queues = "delayed.queue")
    public void listener1(@Payload String message){
        log.info("接收的消息:{}",message);
    }

```

##### 2.3.3.3 结果

![image-20220408175851736](img.assets\image-20220408175851736.png)



### 3.**总结** 

​		延时队列在需要延时处理的场景下非常有用，使用 RabbitMQ 来实现延时队列可以很好的利用RabbitMQ 的特性，如：消息可靠发送、消息可靠投递、死信队列来保障消息至少被消费一次以及未被正确处理的消息不会被丢弃。另外，通过 RabbitMQ 集群的特性，可以很好的解决单点故障问题，不会因为单个节点挂掉导致延时队列不可用或者消息丢失。

​		延时队列还有很多其它选择，比如利用 Java 的 DelayQueue，利用 Redis 的 zset，利用 Quartz或者利用 kafka 的时间轮，这些方式各有特点,看需要适用的场景。



# 十二.Springboot发布确认

​		在生产环境中由于一些不明原因，导致 rabbitmq 重启，在 RabbitMQ 重启期间生产者消息投递失败，导致消息丢失，需要手动处理和恢复。在这种情况下如何进行 RabbitMQ 的消息可靠投递？

```ABAP
应 用 [xxx] 在 [08-1516:36:04] 发 生 [ 错 误 日 志 异 常 ] ， alertId=[xxx] 。 由
[org.springframework.amqp.rabbit.listener.BlockingQueueConsumer:start:620] 触 发 。
应用 xxx 可能原因如下
服 务 名 为 ： 异 常 为 ： org.springframework.amqp.rabbit.listener.BlockingQueueConsumer:start:620,
产 生 原 因 如 下 :1.org.springframework.amqp.rabbit.listener.QueuesNotAvailableException:
Cannot prepare queue for listener. Either the queue doesn't exist or the broker will not
allow us to use it.||Consumer received fatal=false exception on startup
```

## 1.确认机制方案

![image-20220409144746622](E:\笔记\MQ\RabbitMQ\img.assets\image-20220409144746622.png)

### 1.1配置文件

在配置文件当中需要添加

```properties
#开启交换机确定接收消息
spring.rabbitmq.publisher-confirm-type=correlated
```

NONE   禁用发布确认模式，是默认值

CORRELATED  发布消息成功到交换器后会触发回调方法

SIMPLE  有两种效果，其一效果和 CORRELATED 值一样会触发回调方法，

其二在发布消息成功后使用 rabbitTemplate 调用 `waitForConfirms` 或` waitForConfirmsOrDie` 方法等待 broker 节点返回发送结果，根据返回结果来判定下一步的逻辑，要注意的点是waitForConfirmsOrDie 方法如果返回 false 则会关闭 channel，则接下来无法发送消息到 broker。（**相当于单个确认**）



## 2.**回调接口** 

在RabbitTemplate类中有一个ConfirmCallback接口,我们可以实现该接口来实现回调

**回调接口只能确定交换机是否接收到消息，不能保证队列是否可以接收到消息**

```java
@FunctionalInterface
	public interface ConfirmCallback {
		void confirm(@Nullable CorrelationData correlationData, boolean ack, @Nullable String cause);

}
```

### 2.1实现回调接口

**将该接口的实现类注入到RabbitTemplate 中**

在配置文件中添加`spring.rabbitmq.publisher-confirm-type=correlated`，否则无法使用

```java
@Component
@Slf4j
public class MyConfirmCallBack implements RabbitTemplate.ConfirmCallback {
    //实现RabbitTemplate的内部接口,将实现后的类注入到RabbitTemplate中

    @Resource
    private RabbitTemplate rabbitTemplate;
    //将该类注入到模板对象中

    @PostConstruct //被注解的方法在bean创建并且完成赋值，在执行初始化方法之间调用
    //@PostConstruct该注解被用来修饰一个非静态的void（）方法。
    //被@PostConstruct修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器执行一次。
    //PostConstruct在构造函数之后执行，init（）方法之前执行。
    //Constructor(构造方法) -> @Autowired(依赖注入) -> @PostConstruct(注释的方法)
    public void init(){
        rabbitTemplate.setConfirmCallback(this);
    }

    /**
     * 交换机确定回调的方法
     * @param correlationData  保存回调消息的相关信息，ID等
     * @param ack 应答，若为true 则交换机收到消息
     * @param cause 失败的原因，若成功则为null
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
      if (ack){
          log.info("交换机收到了消息,消息的id:{},消息的内容:{}",correlationData.getId(),
                  new String(correlationData.getReturned().getMessage().getBody()));
      } else {
          log.info("交换机没有收到消息,消息的id:{},原因:{}",correlationData.getId(),cause);
      }
    }
}
```

### 2.2 发送和接收消息

```java
@GetMapping("/confirm/{message}")
    public void test1(@PathVariable String message){
        //设置消息的id，若不不设置，默认使用UUID
        CorrelationData correlationData = new CorrelationData();
        rabbitTemplate.convertAndSend("confirm.exchange","key", "发送的消息"+message,correlationData);
        log.info("发送的消息为:{}",message);
    }

    @RabbitListener(queues = "confirm.queue")
    public void listener(Message message){
        log.info("接收的消息:{}",new String(message.getBody()));
    }
```



## 3.**回退消息** 

### 3.1 **Mandatory 参数** 

​		**在仅开启了生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息，如果发现该消息不可路由，那么消息会被直接丢弃，此时生产者是不知道消息被丢弃这个事件的**。

​	通过设置 `mandatory` 参数可以在当消息传递过程中不可达目的地时将消息返回给生产者

```java
//true:交换机无法将消息进行路由时，会将该消息返回给生产者
//false:如果发现消息无法进行路由，则直接丢弃
public void setMandatory(boolean mandatory) {
		this.mandatoryExpression = new ValueExpression<>(mandatory);
}
```

### 3.2 配置文件

```properties
#开启路由失败后消息回退
spring.rabbitmq.publisher-returns=true
```

### 3.3 回退接口

在RabbitTemplate类中有一个ReturnsCallback接口,我们可以实现该接口来实现消息的回退

```java
@FunctionalInterface
public interface ReturnsCallback extends ReturnCallback {       
	@Override
	void returnedMessage(ReturnedMessage returned);
}
```

#### 3.3.1 实现退回接口

**将该接口的实现类注入到RabbitTemplate 中**，并且将`mandatory`参数设置为true

在配置文件中添加`spring.rabbitmq.publisher-returns=true`,否则无法使用

```java
@Component
@Slf4j
public class MyConfirmCallBack implements RabbitTemplate.ConfirmCallback,RabbitTemplate.ReturnsCallback{
    //实现RabbitTemplate的内部接口,将实现后的类注入到RabbitTemplate中

    @Resource
    private RabbitTemplate rabbitTemplate;
    //将该类注入到模板对象中

    @PostConstruct
    public void init(){
        rabbitTemplate.setConfirmCallback(this);
        //设置Mandatory参数
        //true:交换机无法将消息进行路由时，会将该消息返回给生产者
        //false:如果发现消息无法进行路由，则直接丢弃
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setReturnsCallback(this);
    }

    /**
     * 消息回退接口的实现
     * @param returned 消息的相关参数
     */
    @Override
    public void returnedMessage(ReturnedMessage returned) {
        log.info("消息{},被交换机{}退回,退回的原因:{},路由key:{}",
                new String(returned.getMessage().getBody()),
                returned.getExchange(),
                returned.getReplyText(),
                returned.getRoutingKey());
    }
}
```



## 4.备份交换机

​			备份交换机可以理解为 RabbitMQ 中交换机的“备胎”，当我们为某一个交换机声明一个对应的备份交换机时，就是为它创建一个备胎，**当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处**理，通常备份交换机的类型为 `Fanout` ，这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进入这个队列了。我们还可以建立一个报警队列，用独立的消费者来进行监测和报警。

注意：

​			1.**备份交换机只能处理不可路由的情况，若交换机没有收到消息，使用回调接口**

​			2.**使用备份交换机后将不再执行回退消息返回给生产者,而是直接返送给备份交换机**

mandatory 参数与备份交换机可以一起使用的时候，如果两者同时开启，**备份交换机优先级高**

![image-20220409163755486](img.assets\image-20220409163755486.png)

### 4.1 指定备份交换机

```java
@Bean
public DirectExchange confirmExchange(){
        //指定备份交换机
        //备份交换机需要持久化
        //alternate 指定备用交换机
return ExchangeBuilder.directExchange(EXCHANGE_NAME).durable(true).alternate(BACKUP_EXCHANGE_NAME).build();
}
```



# 十三. 幂等性

​		**用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。**

​		举个最简单的例子，那就是支付，用户购买商品后支付，支付扣款成功，但是返回结果的时候网络异常，此时钱已经扣了，用户再次点击按钮，此时会进行第二次扣款，返回结果成功，用户查询余额发现多扣钱了，流水记录也变成了两条。在以前的单应用系统中，我们只需要把数据操作放入事务中即可，发生错误立即回滚，但是再响应客户端的时候也有可能出现网络中断或者异常等等。



## 1.**消息重复消费** 

​		消费者在消费 MQ 中的消息时，MQ 已把消息发送给消费者，消费者在给MQ 返回 ack 时网络中断，故 MQ 未收到确认信息，该条消息会重新发给其他的消费者，或者在网络重连后再次发送给该消费者，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息。



## 2.**解决思路** 

​		MQ 消费者的幂等性的解决一般使用**全局 ID 或者写个唯一标识比如时间戳 或者 UUID** 或者订单消费者消费 MQ 中的消息也可利用 MQ 的该 id 来判断，或者可按自己的规则生成一个全局唯一 id，每次消费消息时用该 id 先判断该消息是否已消费过。



## 3.消费端的幂等性保障

​		在海量订单生成的业务高峰期，生产端有可能就会重复发生了消息，这时候消费端就要实现幂等性，这就意味着我们的消息永远不会被消费多次，即使我们收到了一样的消息。业界主流的幂等性有两种操作:a.唯一 ID+指纹码机制,利用数据库主键去重, b.利用 redis 的原子性去实现



### 3.1 唯一ID+指纹码机制

​		指纹码:我们的一些规则或者时间戳加别的服务给到的唯一信息码,它并不一定是我们系统生成的，基本都是由我们的业务规则拼接而来，但是一定要保证唯一性，然后就利用查询语句进行判断这个 id 是否存在数据库中,优势就是实现简单就一个拼接，然后查询判断是否重复；劣势就是在高并发时，如果是单个数据库就会有写入性能瓶颈当然也可以采用分库分表提升性能，但也不是最推荐的方式。



### 3.2 **Redis 原子性**

​	利用 redis 执行 setnx 命令，天然具有幂等性。从而实现不重复消费



# 十四.**优先级队列** 

​        简单来说就是会员机制。

​	   **优先级队列 0-255** 越大就越先执行

## 1.如何添加

### 1.1 在客户端添加

![image-20220409172252705](img.assets\image-20220409172252705.png)

### 1.2 代码实现

​	要让队列实现优先级需要做的事情有如下事情:**队列需要设置为优先级队列，消息需要设置消息的优先级**，消费者需要等待消息已经发送到队列中才去消费,因为这样才有机会对消息进行排序。

**设置队列最大优先级**

```java
@Bean
public Queue queue(){
   //设置队列优先级 官方允许在0-255之间,不要设置过大
   return QueueBuilder.nonDurable("1").maxPriority(10).build();
}
```

**设置消息优先级**  消息的优先级不能高于队列的优先级

```java
rabbitTemplate.convertAndSend("1","", message,msg->{
            //设置消息的优先级
            msg.getMessageProperties().setPriority(5);
            return msg;
});
```



# 十五.**惰性队列** 

**队列具备两种模式：default 和 lazy**。默认的为default 模式，在3.6.0 之前的版本无需做任何变更。**lazy模式即为惰性队列的模式**，可以通过调用 channel.queueDeclare 方法的时候在参数中设置，也可以通过Policy 的方式设置，如果一个队列同时使用这两种方式设置的话，那么 Policy 的方式具备更高的优先级。如果要通过声明的方式改变已有队列的模式的话，那么只能先删除队列，然后再重新声明一个新的。

​	正常队列：消息保存在内存中

​	惰性队列：消息保存在磁盘中

## 1.使用场景

​		惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储。当消费者由于各种各样的原因(比如消费者下线、宕机亦或者是由于维护而关闭等)而致使长时间内不能消费消息造成堆积时，惰性队列就很有必要了。



## 2.声明惰性队列

​	在队列声明的时候可以通过`x-queue-mode`参数来设置队列的模式，取值为“default”和“lazy”。

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("myqueue", false, false, false, args);
```

或者

```java
@Bean
public Queue queue(){
    //设置惰性队列
    return QueueBuilder.nonDurable("1").lazy().build();
}
```



## 3. 内存开销对比

![image-20220409195302196](img.assets\image-20220409195302196.png)

​		在发送 1 百万条消息，每条消息大概占 1KB 的情况下，普通队列占用内存是 1.2GB，而惰性队列仅仅占用 1.5MB



# 十六.**RabbitMQ** 集群

## 1.搭建步骤

### 1.1 修改 3 台机器的主机名称

```shell
vim /etc/hostname
```

### 1.2 配置各个节点的 hosts 文件

​	让各个节点都能互相识别对方

```shell
vim /etc/hosts
```

```shell
10.211.55.74 node1
10.211.55.75 node2
10.211.55.76 node3
```

![image-20220409200221258](.\img.assets\image-20220409200221258.png)

### 1.3 确保各个节点的 cookie 文件使用的是同一个值

在 node1 上执行远程操作命令

```shell
scp /var/lib/rabbitmq/.erlang.cookie root@node2:/var/lib/rabbitmq/.erlang.cookie
scp /var/lib/rabbitmq/.erlang.cookie root@node3:/var/lib/rabbitmq/.erlang.cookie
```

### 1.4 启动 RabbitMQ 服务,顺带启动 Erlang 虚拟机和 RbbitMQ 应用服务

在三台节点上分别执行以下命令

```shell
rabbitmq-server -detached

rabbitmqctl stop_app #rabbitmqctl stop 会将Erlang 虚拟机关闭，rabbitmqctl stop_app 只关闭 RabbitMQ 服务

rabbitmqctl reset

rabbitmqctl join_cluster rabbit@node1 #根据主机名进行修改

rabbitmqctl start_app #只启动应用服务
```

### 1.5  集群状态

```shell
rabbitmqctl cluster_status
```

### 1.6 重新设置用户

```shell
#创建账号
rabbitmqctl add_user root root
#设置用户角色
rabbitmqctl set_user_tags root administrator
#设置用户权限
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
```

### 1.7 解除集群节点

(node2 和 node3 机器分别执行)

```shell
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
rabbitmqctl cluster_status

rabbitmqctl forget_cluster_node rabbit@node2(node1 机器上执行)
```



## 2. 镜像队列

​		引入镜像队列(Mirror Queue)的机制，可以将队列镜像到集群中的其他 Broker 节点之上，如果集群中的一个节点失效了，队列能自动地切换到镜像中的另一个节点上以保证服务的可用性。

​		就算整个集群只剩下一台机器了 依然能消费队列里面的消息

### 2.1 搭建步骤

​	1.启动三台集群节点

​	2.随便找一个节点添加 policy

![image-20220409201722390](img.assets\image-20220409201722390.png)

添加的policy为：

![image-20220409201755626](img.assets\image-20220409201755626.png)



## 3.**Haproxy+Keepalive** 实现高可用负载均衡

**整体架构图** 

<img src="img.assets\image-20220409202236958.png" alt="image-20220409202236958" style="zoom:80%;" />

### 3.1 Haproxy 实现负载均衡

​		HAProxy 提供高可用性、负载均衡及基于TCPHTTP 应用的代理，支持虚拟主机，它是免费、快速并且可靠的一种解决方案，包括 Twitter,Reddit,StackOverflow,GitHub 在内的多家知名互联网公司在使用。HAProxy 实现了一种事件驱动、单一进程模型，此模型支持非常大的井发连接数。

​	 nginx,lvs,haproxy 之间的区别: http://www.ha97.com/5646.html

### 3.2 搭建步骤

1.下载 haproxy(在 node1 和 node2)

```shell
yum -y install haproxy
```

2.修改 node1 和 node2 的 haproxy.cfg

```shell
vim /etc/haproxy/haproxy.cfg
```

需要修改红色 IP 为当前机器 IP

![image-20220409202456862](img.assets\image-20220409202456862.png)

3.在两台节点启动 haproxy

```java
haproxy -f /etc/haproxy/haproxy.cfg
ps -ef | grep haproxy
```

4.访问地址

```
http://IP地址:8888/stats
```



### 3.3 Keepalived 实现双机(主备)热备

​		试想如果前面配置的 HAProxy 主机突然宕机或者网卡失效，那么虽然 RbbitMQ 集群没有任何故障但是对于外界的客户端来说所有的连接都会被断开结果将是灾难性的为了确保负载均衡服务的可靠性同样显得十分重要，这里就要引入 Keepalived 它能够通过自身健康检查、资源接管功能做高可用(双机热备)，实现故障转移。



### 3.4 搭建步骤

1.下载 keepalived

```shell
yum -y install keepalived
```

2.节点 node1 配置文件

```shell
vim /etc/keepalived/keepalived.conf
```

```shell
global_defs {
   # 路由id: 当前安装的keepalived节点主机的标识符，全局唯一
   router_id kepp_155
}

# 计算机节点
vrrp_instance VI_1 {
    # 表示的状态，当前155位nginx的主节点，MASTER/BACKUP
    state MASTER
    # 当前实例绑定的网卡
    interface enp0s3
    # 表示那些服务器一个组，保证主备节点一致
    virtual_router_id 51
    # 优先级/权重，谁的优先级高，在MASTER关掉以后，就能成为MASTER
    priority 100
    # 主备之间同步检查的时间间隔，默认1s
    advert_int 1
    # 认知授权的密码，防止非法节点的进入
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    #  虚拟ip
    virtual_ipaddress {
        192.168.1.111
    }
}
```

3.节点 node2 配置文件

```
需要修改global_defs 的 router_id,如:nodeB
其次要修改 vrrp_instance_VI 中 state 为"BACKUP"；
最后要将priority 设置为小于 100 的值
```

4.添加 haproxy_chk.sh

​	(为了防止 HAProxy 服务挂掉之后 Keepalived 还在正常工作而没有切换到 Backup 上，所以这里需要编写一个脚本来检测 HAProxy 务的状态,当 HAProxy 服务挂掉之后该脚本会自动重启HAProxy 的服务，如果不成功则关闭 Keepalived 服务，这样便可以切换到 Backup 继续工作)

```shell
vim /etc/keepalived/haproxy_chk.sh #(可以直接上传文件)
chmod 777 /etc/keepalived/haproxy_chk.sh #修改权限 
```

5.启动 keepalive 命令(node1 和 node2 启动)

```shell
systemctl start keepalived
```

6. 观察 Keepalived 的日志

```shell
tail -f /var/log/messages -n 200
```

7.观察最新添加的 vip

```shell
ip add show
```

8. node1 模拟 keepalived 关闭状态

```shell
systemctl stop keepalived
```

9. 使用 vip 地址来访问 rabbitmq 集群



## 4.Federation Exchange

​	解决两地之间因网络问题造成的延迟问题

### 4.1 搭建步骤

1.需要保证每台节点单独运行

2.在每台机器上开启 federation 相关插件

```shell
rabbitmq-plugins enable rabbitmq_federation

rabbitmq-plugins enable rabbitmq_federation_management
```

3.原理图(先运行 consumer 在 node2 创建 fed_exchange)

<img src="img.assets\image-20220409214510482.png" alt="image-20220409214510482" style="zoom:80%;" />

4.在 downstream(node2)配置 upstream(node1)

![image-20220409214707951](img.assets\image-20220409214707951.png)

5.添加 policy

![image-20220409214736207](img.assets\image-20220409214736207.png)

6.成功的前提

![image-20220409214807308](img.assets\image-20220409214807308.png)

## 5.Federation Queue

​	联邦队列可以在多个 Broker 节点(或者集群)之间为单个队列提供均衡负载的功能。一个联邦队列可以连接一个或者多个上游队列(upstream queue)，并从这些上游队列中获取消息以满足本地消费者消费消息的需求

**搭建步骤** 



![image-20220409215120846](img.assets\image-20220409215120846.png)

2.添加 upstream(同上)

3.添加 policy

![image-20220409215140709](img.assets\image-20220409215140709.png)

## 6.Shovel

​		Federation 具备的数据转发功能类似，Shovel 够可靠、持续地从一个 Broker 中的队列(作为源端，即source)拉取数据并转发至另一个 Broker 中的交换器(作为目的端，即 destination)。作为源端的队列和作为目的端的交换器可以同时位于同一个 Broker，也可以位于不同的 Broker 上。Shovel 可以翻译为"铲子"，是一种比较形象的比喻，这个"铲子"可以将消息从一方"铲子"另一方。Shovel 行为就像优秀的客户端应用程序能够负责连接源和目的地、负责消息的读写及负责连接失败问题的处理。

**搭建步骤**

1.开启插件(需要的机器都开启)

```
rabbitmq-plugins enable rabbitmq_shovel

rabbitmq-plugins enable rabbitmq_shovel_management
```

2.原理图(在源头发送的消息直接回进入到目的地队列)

![image-20220409215305288](img.assets\image-20220409215305288.png)

3.添加 shovel 源和目的地

![image-20220409215326175](img.assets\image-20220409215326175.png)
