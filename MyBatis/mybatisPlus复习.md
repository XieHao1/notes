# mybatisPlus

# 1.配置

pom:

```xml
		 <!--mybatis-plus起步依赖-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.1</version>
        </dependency>

        <!--  mysql驱动  -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.30</version>
        </dependency>
```

propertites:

```properties
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis_plus
spring.datasource.username=root
spring.datasource.password=root

#mybatis-plus配置
mybatis-plus.mapper-locations=classpath:mapper/*.xml
#日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
#去除表的前缀
mybatis-plus.global-config.db-config.table-prefix=xh_
```

主启动类:

```java
@SpringBootApplication
@MapperScan("com.xh.mybatisplusreview.mapper")
public class MybatisPlusReviewApplication {
    public static void main(String[] args) {
        SpringApplication.run(MybatisPlusReviewApplication.class, args);
    }
}
```

mapper:

```java
public interface UserMapper extends BaseMapper<User> {

}
```



# 2.主键的生成策略

```java
public enum IdType {
     //自动增长
    AUTO(0),
   	//该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT) 
    NONE(1),
    //手工输入
    //（insert 前自行 set 主键值）
    INPUT(2),
    //使用雪花算法自动生成主键 ID，主键类型为 Long 或 String---分布式系统
    ASSIGN_ID(3),
    //自动生成不含中划线的 UUID 作为主键。
   //主键类型为 String，对应 MySQL 的表字段为 VARCHAR(32)
    ASSIGN_UUID(4);
    private final int key;
    private IdType(int key) {this.key = key;}
    public int getKey() {return this.key;}
}
```

使用：在实体类中主键的属性上加上@TableId注解

**只有加上@TableId注解，才可以使用XXXById方法** 

```java
@TableId(type = IdType.ASSIGN_ID)
private Long id;
```



# 3.自动填充

项目中经常会遇到一些数据，每次都使用相同的方式填充，例如记录的`创建时间，更新时间`等。

我们可以使用 MyBatis Plus 的自动填充功能，完成这些字段的赋值工作:

原理:

- 实现元对象处理器接口：com.baomidou.mybatisplus.core.handlers.MetaObjectHandler
- 注解填充字段 `@TableField(.. fill = FieldFill.INSERT)` 生成器策略部分也可以配置！

```java
    //插入时修改
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;

    //更新和插入时修改
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
```

FieldFill:

```java
public enum FieldFill {
    //默认不处理
    DEFAULT,
    //插入时填充字段
    INSERT,
    //更新时填充字段
    UPDATE,
	//插入和更新时填充字段
    INSERT_UPDATE
}
```

- **自定义实现类 MyMetaObjectHandler,实现MetaObjectHandler接口,重写insertFill和updataFill方法**
- **并且交给spring管理**

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        //字段名，传入的值，要和字段的类型相同
        this.setFieldValByName("createTime", new Date(), metaObject);
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
}
```

运行结果

```
==>  Preparing: INSERT INTO user ( id, name, age, email, create_time, update_time ) VALUES ( ?, ?, ?, ?, ?, ? )
==> Parameters: 1530596237416386561(Long), 李四(String), 20(Integer), 222@qq.com(String), 2022-05-29 01:05:54.763(Timestamp), 2022-05-29 01:05:54.763(Timestamp)
<==    Updates: 1
```



# 4.乐观锁

**乐观锁**：总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，只在更新的时候会判断一下在此期间别人有没有去更新这个数据。

**悲观锁**：总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞，直到它拿到锁（共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程）。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。Java中`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现。



**乐观锁主要适用场景**：当要更新一条记录的时候，希望这条记录没有被别人更新，也就是说实现线程安全的数据更新，**解决更新丢失的问题**

乐观锁实现方式：

> - 取出记录时，获取当前 version
> - 更新时，带上这个 version
> - 执行更新时， set version = newVersion where version = oldVersion
> - 如果 version 不对，就更新失败

在提交数据时，比较当前版本号和数据库版本号是否一致，一致可提交，并将版本号+1；不一致就提交失败



## 4.1 MP乐观锁实现步骤

### 4.1.1 数据库中添加version字段

### 4.1.2 实体类添加version字段，并添加 `@Version` 注解

```java
//乐观锁配置注解 @Version
@Version
private Integer version;
```

说明:

- **支持的数据类型只有:int,Integer,long,Long,Date,Timestamp,LocalDateTime**
- 整数类型下 `newVersion = oldVersion + 1`
- `newVersion` 会回写到 `entity` 中
- 仅支持 `updateById(id)` 与 `update(entity, wrapper)` 方法
- **在 `update(entity, wrapper)` 方法下, `wrapper` 不能复用!!!**



### 4.1.3 配置乐观锁插件

```java
@Configuration
@MapperScan("com.xh.mybatisplusreview.mapper")
public class MybatisPlusConfig {
    /**
     * 新版
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return mybatisPlusInterceptor;
    }
}
```

### 4.1.4 先查后修改

```java
//根据id查询数据
final User user = userMapper.selectById(1L);
//进行修改
user.setAge(50);
userMapper.updateById(user);
```



# 5.分页插件

 **PaginationInnerInterceptor**

配置：

```java
@Configuration
@MapperScan("com.xh.mybatisplusreview.mapper")
public class MybatisPlusConfig {
    /**
     * 新版
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        //乐观锁
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        //分页
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return mybatisPlusInterceptor;
    }

}
```

使用：

```java
QueryWrapper<User> qw  = new QueryWrapper<>();
IPage<User> page = new Page<>();
//设置分页的信息
page.setCurrent(1);//第一页
page.setSize(3);//每页的记录数
//指向分页操作
IPage<User> page1 = userDao.selectPage(page, qw);
//获取分页后的记录
List<User> users = page1.getRecords();
//获取每一页的记录数
System.out.println("每页的记录数"+page1.getSize());
//获取总页数
System.out.println("页数"+page1.getPages());
//获取总记录数，数据的个数
System.out.println("总记录数"+page1.getTotal());
//获取当前页码
System.out.println("当前页数"+page1.getCurrent());

//判断是否有下一页
System.out.println("是否有下一页"+((Page<User>)iPage1).hasNext());
//判断是否有上一页
System.out.println("是否有上一页"+((Page<User>)iPage1).hasPrevious());
```



自定义分页：

1.在Dao接口中自定义分页方法

```java
/**      
* 查询 : 根据state状态查询用户列表，分页显示 
* @param page 分页对象,xml中可以从里面进行取值,传递参数 Page 即自动分页,必须放在第一位(你可以继承Page实现自己的分页对象)    
* @param state 状态     
* @return 分页对象     
*/    
IPage<User> selectPageVo(Page<?> page, Integer state);
```

2.写SQL语句，mybatis-plus 自动替你分页

```sql
<select id="selectPageVo" resultType="com.baomidou.cloud.entity.UserVo">    
	SELECT id,name FROM user WHERE state=#{state} 
</select>
```

3.调用分页方法

```java
 //页数+每页的记录数 Page
 page = new Page<>(1, 2); 
 //调用自定义sql IPage iPage = userMapper.selectUserPage(page, 10);
```



# 6.逻辑删除

- 物理删除：**真实删除**，将对应数据从数据库中删除，之后查询不到此条被删除数据
- 逻辑删除：**假删除**，将对应数据中代表是否被删除字段状态修改为“被删除状态”，之后在数据库中仍
  旧能看到此条数据记录

使用步骤：

**（1）数据库中添加 deleted字段**

**（2）实体类添加@TableLogic注解**

```java
//逻辑删除
@TableLogic
private Integer deleted;
```

字段类型支持说明:

- 支持所有数据类型(推荐使用 `Integer`,`Boolean`,`LocalDateTime`)
- 如果数据库字段使用`datetime`,逻辑未删除值和已删除值支持配置为字符串`null`,另一个值支持配置为函数来获取值如`now()`

**（3）配置application**

此为默认值，如果你的默认值和 mp 默认的一样 , 该配置可无，指定删除为1，没删为0

```properties
#全局逻辑删除的实体字段名(since 3.3.0,配置后可以忽略不配置步骤2)
mybatis-plus.global-config.db-config.logic-delete-field=deleted
#逻辑已删除值(默认为 1)
mybatis-plus.global-config.db-config.logic-delete-value=1
#逻辑未删除值(默认为 0)
mybatis-plus.global-config.db-config.logic-not-delete-value=0 
```

测试：

```java
//测试逻辑删除
@Test
public void testLogicDelete() {
    //删除，但是底层的SQL语句变成了修改deleted的值
   userMapper.deleteById(1L);
}
```

```sql
 UPDATE user SET deleted=1 WHERE id=? AND deleted=0
```

说明:

只对自动注入的 sql 起效:

- 插入: 不作限制
- 查找: 追加 where 条件过滤掉已删除数据,且使用 wrapper.entity 生成的 where 条件会忽略该字段
- 更新: 追加 where 条件防止更新到已删除数据,且**使用 wrapper.entity 生成的 where 条件会忽略该字段**
- 删除: 转变为 更新

例如:

- 删除: `update user set deleted=1 where id = 1 and deleted=0`
- 查找: `select id,name,deleted from user where deleted=0`





# 7.其它

**若未使用mybatis主键策略，数据库中没有设置自增，默认使用雪花算法生成一个主键**

```java
public void insertUser(){
        User user = new User();
        user.setAge(20);
        user.setName("张三");
        user.setEmail("222@qq.com");
        final int insert = userMapper.insert(user);
        System.out.println(insert);
    }
```

```
==>  Preparing: INSERT INTO user ( id, name, age, email ) VALUES ( ?, ?, ?, ? )
==> Parameters: 1530579786005508097(Long), 张三(String), 20(Integer), 222@qq.com(String)
<==    Updates: 1
```

```java
    /* 以下3种类型、只有当插入对象ID 为空，才自动填充。 */
    /**
     * 分配ID (主键类型为number或string）,雪花算法，生成分布式id
     */
   ASSIGN_ID(3),
    /**
     * 分配UUID (主键类型为 string)
     */
    ASSIGN_UUID(4);
```



**防止前端精度损失**

```java
//防止前端精度损失 把id转为string    
//分布式id比较长，传到前端会有精度损失，必须转为string类型 进行传输，就不会有问题了    
@JsonSerialize(using = ToStringSerializer.class)    
private Long id;
```

