# 使用RabbitMQ经行缓存更新

## 1.使用aop向队列中发送消息

指定要发送的路由的key值

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface NoticeUpdateCache {

    String name() default "更新缓存";
    //发送消息的路由
    String routingKey();
}

```



```java
@Aspect
@Slf4j
@Component
public class NoticeUpdateCacheAspect {

    @Resource
    private RabbitTemplate rabbitTemplate;

    @Pointcut("@annotation(com.xh.readapp.common.NoticeUpdateCache)")
    public void pt(){}

    @AfterReturning("pt()")
    public void sendNotificationMessage(JoinPoint pjp){
        long start = System.currentTimeMillis();
        MethodSignature signature = (MethodSignature) pjp.getSignature();
        Method method = signature.getMethod();
        NoticeUpdateCache annotation = method.getAnnotation(NoticeUpdateCache.class);
        String name = annotation.name();
        String routingKey = annotation.routingKey();
        CorrelationData correlationData = new CorrelationData();
        correlationData.setId(UUIDUtil.getUUID());
        rabbitTemplate.convertAndSend(ExchangeNameEnum.EXCHANGE_NAME.getExchangeName(),routingKey,name,correlationData);
        log.info("发送消息:{},发送的路由key:{}",name,routingKey);
        long end = System.currentTimeMillis();
        RecordLogUtil.recordLog(pjp,end-start);
    }
}
```



## 2.监听消息

```java
@Component
@Slf4j
public class ListenerMessageService {

    @Autowired
    private UpdateCacheService updateCacheService;

    @RabbitListener(queues = "article.queue")
    public void listenerArticleMsg(@Payload String message){
        log.info("article.queue收到的消息为:{}",message);
        updateCacheService.deleteArticleCache();
    }

    @RabbitListener(queues = "data.queue")
    public void listenerDataMsg(@Payload String message){
        log.info("data.queue收到的消息为:{}",message);
        updateCacheService.deleteDataCache();
    }

    @RabbitListener(queues = "warning.queue")
    public void listenerWarningMsg(@Payload String message){
        log.info("warning.queue警告消息:{}",message);
    }

    @RabbitListener(queues = "comment.queue")
    public void listenerCommentMsg(@Payload String message){
        log.info("comment.queue收到消息为:{}",message);
        updateCacheService.deleteCommentCache();
    }

    @RabbitListener(queues = "history.queue")
    public void listenerHistoryMsg(@Payload String message){
        log.info("history.queue收到消息为:{}",message);
        updateCacheService.deleteHistoryCache();
    }
}
```



## 3.进行缓存的更新操作

**redis中不支持模糊删除**

```java
@Service
@Slf4j
public class UpdateCacheService {

    @Autowired
    private RedisTemplate<String,String> redisTemplate;

    public void deleteArticleCache(){
        Set<String> keys = redisTemplate.keys("查看*");
        if(null != keys){
            redisTemplate.delete(keys);
            log.info("删除所有查看文章缓存");
        }

    }

    public void deleteDataCache(){
        Set<String> keys = redisTemplate.keys("个人资料*");
        if(null != keys){
            redisTemplate.delete(keys);
            log.info("删除所有个人资料缓存");
        }

    }

    public void deleteCommentCache() {
        Set<String> keys = redisTemplate.keys("评论*");
        if(null != keys){
            redisTemplate.delete(keys);
            log.info("删除所有评论缓存");
        }
    }

    public void deleteHistoryCache() {
        Set<String> keys = redisTemplate.keys("阅读室*");
        if(null != keys){
            redisTemplate.delete(keys);
            log.info("删除所有阅读室缓存");
        }
    }
}
```

