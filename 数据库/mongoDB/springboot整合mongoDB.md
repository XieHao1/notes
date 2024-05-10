# springboot整合mongodb

## 1.依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```



## 2.配置文件

```properties
# mongodb 没有开启任何安全协议
# mongodb(协议)://121.5.167.13(主机):27017(端口)/xh(库名)
spring.data.mongodb.uri=mongodb://192.168.153.131:27017/xh


# mongodb 存在密码
#spring.data.mongodb.host=192.168.153.131
#spring.data.mongodb.port=27017
#spring.data.mongodb.database=xh
#spring.data.mongodb.username=root
#spring.data.mongodb.password=root
```



## 3.集合操作

`MongoTemplate`操作模板

```java
   @Resource
   private MongoTemplate mongoTemplate;
```



### 3.1 创建集合

```java
@Test
public void testCreateCollection(){
  mongoTemplate.createCollection("users");//参数: 创建集合名称
}


@Test
void creatCollection(){
    CollectionOptions collectionOptions = CollectionOptions.empty();
    collectionOptions.maxDocuments(100L).size(50000L).capped();
    //capped() 如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。当该值为 true 时，必须指定 size 参数
    //size:为固定集合指定一个最大值，即字节数。如果 capped 为 true，也需要指定该字段。
    //maxDocuments:最大文档数
    mongoTemplate.createCollection("user",collectionOptions);
}
```

`注意:创建集合不能存在,存在报错`

```sql
//判断集合是否存在
mongoTemplate.collectionExists("user");
```



### 3.2 删除集合

```java
@Test
public void testDeleteCollection(){
  mongoTemplate.dropCollection("users");
}
```



## 4.文档操作

- ```java
  @Document
  ```

  - 修饰范围：用在类上
  - 作用：用来映射这个类的一个对象为 mongo 中一条文档数据
  - 属性：(`value 、collection` )用来指定操作的集合名称

- ```java
  @Id
  ```

  - 修饰范围：用在成员变量、方法上
  - 作用：用来将成员变量的值映射为文档的`_id `的值

- ```java
  @Field
  ```

  - 修饰范围：用在成员变量、方法上
  - 作用：用来将成员变量以及值映射为文档中一个key、value对
  - 属性：( `name,value `)用来指定在文档中 key 的名称,默认为成员变量名

- ```java
  @Transient
  ```

  - 修饰范围：用在成员变量、方法上
  - 作用：用来指定改成员变量，不参与文档的序列化
  
- ```java
  @Indexed
  ```

  用于为指定字段添加索引，会调用 MongoDB 的 `createIndex` 方法。值得注意的是：**必须 `@Document` 注解，否则不会自动生成索引**。

| Annotation       | Desc                                                         |
| ---------------- | ------------------------------------------------------------ |
| `@Id`            | 用于指定 MongoDB 内部使用字段 `_id` 的值，如果不指定，则使用自动生成的值。 |
| `@Field`         | 用于指定数据库中存储的字段名。                               |
| `@Document`      | 用于指定该类的实例对应 MongoDB 的某个指定 Collection 下的 Document。 |
| `@Indexed`       | 用于为指定字段添加索引。`@Indexed(unique = true)` 可注解唯一键 |
| `@CompoundIndex` | 用于指定复合索引。                                           |
| `@Transient`     | 用于将某些字段排除，不与数据库匹配。                         |
| `@Version`       | 用于指定字段的版本，默认值为 0，在每次更新字段后自增。       |



`构造实体类`

```java
@Document(value = "user")
@Data
public class User implements Serializable {

    @Transient
    private static final long serialVersionUID = 111L;

    @Id
    private Integer id;

    @Field(value = "username")
    private String name;

    @Field
    private LocalDateTime birthday;
    
}
```



### 4.1 文档添加

- **insert**：插入重复数据时：`insert`报`DuplicateKeyException`提示主键重复；`save`对已存在的数据进行更新。
- **save**：批处理操作时：`insert`可以一次性插入整个数据，效率较高；`save`需遍历整个数据，一次插入或更新，效率较低。

```java
    @Test
    void insertDocument(){
        User user = new User();
        user.setId(1);
        user.setName("谢豪");
        user.setBirthday(LocalDateTime.now());
        //添加文档
        mongoTemplate.insert(user);
    }
```



### 4.2 文档查询

#### 4.2.1 常规查询

##### 4.2.1.1 查询所有

```java
mongoTemplate.findAll(User.class);

mongoTemplate.find(new Query(), User.class);
```



##### 4.2.1.2 id查询

```java
mongoTemplate.findById(1, User.class);
```



#### 4.2.2 条件查询

- Criteria

![image-20230224090813049](img.asstes\image-20230224090813049.png)



##### 4.2.2.1 等值查询

```java
mongoTemplate.find(Query.query(Criteria.where("username").is("AA")), User.class);
```



##### 4.2.2.2 比较

```java
//lt <
mongoTemplate.find(Query.query(Criteria.where("_id").lt(5)), User.class);
//lte <=
mongoTemplate.find(Query.query(Criteria.where("_id").lte(5)), User.class);
//gt >
mongoTemplate.find(Query.query(Criteria.where("_id").gt(5)), User.class);
//gte >=
mongoTemplate.find(Query.query(Criteria.where("_id").get(5)), User.class);
```



##### 4.2.2.3 AND OR

```java
//and
Criteria criteria = new Criteria().andOperator(
        Criteria.where("_id").lt(4), Criteria.where("name").is("AA")
);
mongoTemplate.find(Query.query(criteria),User.class);

//and()方法只能查询类型为String类型的
mongoTemplate.find(Query.query(Criteria.where("_id").is(3).and("name").is("AA")), User.class);


//OR
Criteria criteria = new Criteria().orOperator(
       Criteria.where("_id").is(1),
       Criteria.where("name").is("AA")
 );
mongoTemplate.find(Query.query(criteria), User.class);
```



##### 4.2.2.4 模糊查询

`使用正则表达式进行匹配`

```java
Pattern pattern = Pattern.compile("豪");
List<User> userList = mongoTemplate.find(Query.query(Criteria.where("name").regex(pattern)), User.class);
```



##### 4.2.2.5 排序

```java
//降序
mongoTemplate.find(new Query().with(Sort.by(Sort.Order.desc("_id"))), User.class);
//升序
mongoTemplate.find(new Query().with(Sort.by(Sort.Order.asc("_id"))), User.class);
```



##### 4.2.2.6 skip limit 分页

```java

Query query = new Query().with(Sort.by(Sort.Order.desc("_id")))
    					 .skip(0) //起始记录数
    					 .limit(3); //一页记录数
mongoTemplate.find(query, User.class);
```



##### 4.2.2.7 count 总条数

```java
mongoTemplate.count(new Query(), User.class);
```



##### 4.2.2.8 distinct 去重

```java
 //distinct 去重
 //参数 1:查询条件 参数 2: 去重字段  参数 3: 操作集合  参数 4: 返回类型
mongoTemplate.findDistinct(new Query(), "name", User.class, String.class);
```



##### 4.2.2.9 使用 json 字符串方式查询

```java
        Query query = new BasicQuery(
                "{name:'AA'}"
        );
        mongoTemplate.find(query, User.class);
```



### 4.3 文档更新

```java
        //更新一条
        mongoTemplate.updateFirst(Query.query(Criteria.where("name").is("AA")),  //更新条件
                new Update().set("name","BB") //更新内容
                ,User.class);

        //更新多条
        mongoTemplate.updateMulti(Query.query(Criteria.where("_id").gt(0)),
                new Update().set("name","aaa"),
                User.class);

        //更新插入
        mongoTemplate.upsert(Query.query(Criteria.where("name").is("xh")),
                new Update().set("name","aaa"),
                User.class);
```



### 4.4 文档删除

```java
@Test
void removeDocument(){
    mongoTemplate.remove(Query.query(Criteria.where("name").is("aaa")),User.class);
}
```
