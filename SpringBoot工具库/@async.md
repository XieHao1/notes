# @Async注解

## 1.含义

 1，在方法上使用该@Async注解，申明该方法是一个异步任务；

 2，在类上面使用该@Async注解，申明该类中的所有方法都是异步任务；

 3，使用此注解的方法的类对象，必须是`spring管理下的bean对象`； 

 4，要想使用异步任务，需要在主类上开启异步配置，即，配置上`@EnableAsync`注解；



## 2.使用

在Spring中启用@Async：

​    1，@Async注解在使用时，如果不指定线程池的名称，则使用Spring默认的线程池，Spring默认的线程池为**`SimpleAsyncTaskExecutor`**。

​    2，方法上一旦标记了这个@Async注解，当其它线程调用这个方法时，就会开启一个新的子线程去异步处理该业务逻辑。



## 3.使用默认线程池

### 3.1 启动类中增加@EnableAsync

以Spring boot 为例，启动类中增加@EnableAsync

```java
@EnableAsync
@SpringBootApplication
public class ManageApplication {
    //...
}
```

### 3.2 方法上加@Async注解

```java
@Component
public class MyAsyncTask {
    @Async
    public void asyncCpsItemImportTask(Long platformId, String jsonList){
        //...具体业务逻辑
    }
}
```

### 3.3 默认线程池的缺陷

  上面的配置会启用默认的线程池/执行器，异步执行指定的方法。

  Spring默认的线程池的默认配置

>- 默认核心线程数：8，
>
>- 最大线程数：Integet.MAX_VALUE，
>
>- 队列使用LinkedBlockingQueue，
>
>- 容量是：Integet.MAX_VALUE，
>
>- 空闲线程保留时间：60s，
>
>- 线程池拒绝策略：AbortPolicy。

 从最大线程数的配置上，相信你也看到问题了：**并发情况下，会无限创建线程**



### 3.4 默认线程池--自定义配置参数

```yml
spring:
  task:
    execution:
      pool:
        max-size: 6
        core-size: 3
        keep-alive: 3s
        queue-capacity: 1000
        thread-name-prefix: name
```



## 4.使用自定义线程池

### 4.1 编写配置类

> 读取线程池属性配置

```java
@Component
@ConfigurationProperties(prefix = "spring.task.execution.pool")
@Data
public class TaskExecutorPoolConfig {
    private int coreSize;
    private int maxSize;
    private int queueCapacity;
    private int keepAlive;
    private String threadNamePrefix;
}

```

> 线程池配置

```java
@Configuration
@EnableAsync
public class TaskExecutorConfig {

    private final TaskExecutorPoolConfig taskExecutorPoolConfig;

    @Autowired
    public TaskExecutorConfig(TaskExecutorPoolConfig taskExecutorPoolConfig) {
        this.taskExecutorPoolConfig = taskExecutorPoolConfig;
    }

    @Bean("myTaskExecutor")
    public Executor asyncServiceExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //核心线程数
        executor.setCorePoolSize(taskExecutorPoolConfig.getCoreSize());
        //最大线程数
        executor.setMaxPoolSize(taskExecutorPoolConfig.getMaxSize());
        //队列数
        executor.setQueueCapacity(taskExecutorPoolConfig.getQueueCapacity());
        //空闲线程时间
        executor.setKeepAliveSeconds(taskExecutorPoolConfig.getKeepAlive());
        //线程池名字前缀
        executor.setThreadNamePrefix(taskExecutorPoolConfig.getThreadNamePrefix());
        //当线程池线程数已满，并且工作队列达到限制,丢弃任务并抛出RejectedExecutionException异常
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());
        //初始化线程
        executor.initialize();
        return executor;
    }
}
```

### 4.2 java线程池7个参数

下面是ThreadPoolExecutor类的构造方法源码，其他创建线程池的方法最终都会导向这个构造方法，共有7个参数：**corePoolSize、maximumPoolSize、keepAliveTime、unit、workQueue、threadFactory、handler**。



#### 4.2.1 corePoolSize：核心线程数

线程池维护的最小线程数量，核心线程创建后不会被回收（注意：设置allowCoreThreadTimeout=true后，空闲的核心线程超过存活时间也会被回收）。

大于核心线程数的线程，在空闲时间超过keepAliveTime后会被回收。

线程池刚创建时，里面没有一个线程，当调用 execute() 方法添加一个任务时，如果正在运行的线程数量小于corePoolSize，则马上创建新线程并运行这个任务。



#### 4.2.2 maximumPoolSize：最大线程数

线程池允许创建的最大线程数量。

当添加一个任务时，核心线程数已满，线程池还没达到最大线程数，并且没有空闲线程，工作队列已满的情况下，创建一个新线程并执行



#### 4.2.3 keepAliveTime：空闲线程存活时间

当一个可被回收的线程的空闲时间大于keepAliveTime，就会被回收。

可被回收的线程：

1. 设置allowCoreThreadTimeout=true的核心线程。
2. 大于核心线程数的线程（非核心线程）。



#### 4.2.4 unit：时间单位

keepAliveTime的时间单位：

```
TimeUnit.NANOSECONDS
TimeUnit.MICROSECONDS
TimeUnit.MILLISECONDS // 毫秒
TimeUnit.SECONDS
TimeUnit.MINUTES
TimeUnit.HOURS
TimeUnit.DAYS
```



#### 4.2.5 workQueue：工作队列

存放待执行任务的队列：当提交的任务数超过核心线程数大小后，再提交的任务就存放在工作队列，任务调度时再从队列中取出任务。它仅仅用来存放被execute()方法提交的Runnable任务。工作队列实现了BlockingQueue接口。

JDK默认的工作队列有五种：

1. ArrayBlockingQueue 数组型阻塞队列：数组结构，初始化时传入大小，有界，FIFO，使用一个重入锁，默认使用非公平锁，入队和出队共用一个锁，互斥。
2. LinkedBlockingQueue 链表型阻塞队列：链表结构，默认初始化大小为Integer.MAX_VALUE，有界（近似无解），FIFO，使用两个重入锁分别控制元素的入队和出队，用Condition进行线程间的唤醒和等待。
3. SynchronousQueue 同步队列：容量为0，添加任务必须等待取出任务，这个队列相当于通道，不存储元素。
4. PriorityBlockingQueue 优先阻塞队列：无界，默认采用元素自然顺序升序排列。
5. DelayQueue 延时队列：无界，元素有过期时间，过期的元素才能被取出



#### 4.2.6 threadFactory：线程工厂

创建线程的工厂，可以设定线程名、线程编号等。

默认线程工厂

```java

    /**
     * The default thread factory
     */
    static class DefaultThreadFactory implements ThreadFactory {
        private static final AtomicInteger poolNumber = new AtomicInteger(1);
        private final ThreadGroup group;
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;
 
        DefaultThreadFactory() {
            SecurityManager s = System.getSecurityManager();
            group = (s != null) ? s.getThreadGroup() :
                                  Thread.currentThread().getThreadGroup();
            namePrefix = "pool-" +
                          poolNumber.getAndIncrement() +
                         "-thread-";
        }
 
        public Thread newThread(Runnable r) {
            Thread t = new Thread(group, r,
                                  namePrefix + threadNumber.getAndIncrement(),
                                  0);
            if (t.isDaemon())
                t.setDaemon(false);
            if (t.getPriority() != Thread.NORM_PRIORITY)
                t.setPriority(Thread.NORM_PRIORITY);
            return t;
        }
    }
```

#### 4.2.7 handler：拒绝策略

当线程池线程数已满，并且工作队列达到限制，新提交的任务使用拒绝策略处理。可以自定义拒绝策略，拒绝策略需要实现RejectedExecutionHandler接口。

JDK默认的拒绝策略有四种：

1. AbortPolicy：丢弃任务并抛出RejectedExecutionException异常。
2. DiscardPolicy：丢弃任务，但是不抛出异常。可能导致无法发现系统的异常状态。
3. DiscardOldestPolicy：丢弃队列最前面的任务，然后重新提交被拒绝的任务。
4. CallerRunsPolicy：由调用线程处理该任务。



#### 4.2.8 线程池的执行流程

![image-20230315102139641](img.assets\image-20230315102139641.png)

#### 4.2.9 关于Spring中的线程池(执行器)

​    Spring用TaskExecutor和TaskScheduler接口提供了异步执行和调度任务的抽象。

​    Spring的TaskExecutor和java.util.concurrent.Executor接口时一样的，这个接口只有一个方法execute(Runnable task)。

Spring已经内置了许多TaskExecutor的实现，没有必要自己去实现：

- **SimpleAsyncTaskExecutor**： 这种实现不会重用任何线程，**每次调用都会创建一个新的线程**。
- **SyncTaskExecutor**： 这种实现不会异步的执行，相反，每次调用都在发起调用的线程中执行。它的主要用处是在不需要多线程的时候，比如简单的测试用例；
- **ConcurrentTaskExecutor**：这个实现是对Java 5 java.util.concurrent.Executor类的包装。有另一个ThreadPoolTaskExecutor类更为好用，它暴露了Executor的配置参数作为bean属性。
-  **SimpleThreadPoolTaskExecutor**： 这个实现实际上是Quartz的SimpleThreadPool类的子类，它会监听Spring的生命周期回调。当你有线程池，需要在Quartz和非Quartz组件中共用时，这是它的典型用处。
-  **`ThreadPoolTaskExecutor`： `这是最常用、最通用的一种实现`**。它包含了java.util.concurrent.ThreadPoolExecutor的属性，并且用TaskExecutor进行包装。



### 4.3 方法上加@Async注解

```java
@Component
public class MyAsyncTask {
     @Async("MyExecutor") //使用自定义的线程池(执行器)
    public void asyncCpsItemImportTask(Long platformId, String jsonList){
        //...具体业务逻辑
    }
}
```



## 5.异步任务的事务问题

 @Async注解由于是异步执行的，在其进行数据库的操作之时，将无法控制事务管理。

 解决办法：**`可以把@Transactional注解放到内部的需要进行事务的方法上`**。



## 6.异步任务的返回结果

 异步的业务逻辑处理场景 有两种：一个是不需要返回结果，另一种是需要接收返回结果。

#### 6.1 需要返回结果

```java
@Async("MyExecutor")
public Future<Map<Long, List>> queryMap(List ids) {
    List<> result = businessService.queryMap(ids);
    ..............
    Map<Long, List> resultMap = Maps.newHashMap();
    ...
    return new AsyncResult<>(resultMap);
}
```

调用异步方法的示例：

```java
public Map<Long, List> asyncProcess(List<BindDeviceDO> bindDevices,List<BindStaffDO> bindStaffs, String dccId) {
        Map<Long, List> finalMap =null;
        // 返回值：
        Future<Map<Long, List>> asyncResult = MyService.queryMap(ids);
        try {
            finalMap = asyncResult.get();
        } catch (Exception e) {
            ...
        }
        return finalMap;
}
```



## 7.无法调用同类中的@Async的方法

**`@Async的原理概括`：**

​    @Async的原理是通过 Spring AOP `动态代理` 的方式来实现的。

​    Spring容器启动初始化bean时，判断类中是否使用了@Async注解，如果使用了则为其创建切入点和切入点处理器，根据切入点创建代理，

​    在线程调用@Async注解标注的方法时，会调用代理，执行切入点处理器invoke方法，将方法的执行提交给线程池中的另外一个线程来处理，从而实现了异步执行。

​    所以，需要注意的一个错误用法是，如果a方法调用它同类中的标注@Async的b方法，是不会异步执行的，因为从a方法进入调用的都是该类对象本身，不会进入代理类。

​    **`因此，相同类中的方法调用带@Async的方法是无法异步的，这种情况仍然是同步`。**