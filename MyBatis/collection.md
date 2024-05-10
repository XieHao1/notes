# Connection标签

Mybatis的 collection 是`一对多的`使用的, 在 resultMap 标签内使用
当一个Bean中有 一个list属性需要关联查询出来的使用就用collection 标签



> 用户实体

```java
@Data
public class User {
    
    private Integer id;
    
    private String name;
    
    //一对多
    private List<Role> roles;
}
```

> 角色

```java
@Data
public class Role {

    private Integer id;

    private Integer userId;

    private String name;

    private String type;

}
```





# 使用方式

## 1.嵌套结果映射

```java
    <!-- 定义resultMap -->
    <resultMap id="UserResultMap" type="User">
        <result column="id" property="id"/>
        <result column="name" property="name"/>
        <collection ofType="role" property="roles">
            <result column="role_id" property="id"/>
            <result column="role_name" property="name"/>
            <result column="role_type" property="type"/>
        </collection>
    </resultMap>

    <!--查询语句-->
    <select id="selectUserById" resultMap="UserResultMap">
        select u.id ,u.name,r.id AS role_id ,r.name AS role_name ,r.type AS role_type FROM user AS u INNER JOIN role AS r ON u.id = r.user_id where u.id = #{id}
    </select>

```



## 2. 嵌套select查询

```java
    <!-- 定义resultMap -->
    <resultMap id="UserResultMap" type="User">
        <result column="id" property="id"/>
        <result column="name" property="name"/>
        <collection ofType="role" property="roles" column="id" select="selectRoleByUserId"/>
    </resultMap>

 

    <!--查询语句-->
    <select id="selectUserById" resultMap="UserResultMap">
        select  id , name FROM user where id = #{id}
    </select>

    <!--查询语句-->
    <select id="selectRoleByUserId" resultMap="role">
        select id , name,type FROM role where user_id = #{userId}
    </select>

```

- select=“selectRoleByUserId” 找的是第二个sql语句,如果调用别的xml文件中方法写全路径就可以找到.
- column=“id” 参数id 传多个参数的话就是 `{“属性名”=“参数”,“属性名”=“参数”}` 这样的.

```sql
        <collection property="productionPreGrindingInputDailyReportDetailVOList" 
        			column="{recordId = id,}" javaType="java.util.ArrayList"
                    ofType="com.bpm.common.vo.ProductionPreGrindingInputDailyReportDetailVO"
                    select="queryRecordDetailList">
```

