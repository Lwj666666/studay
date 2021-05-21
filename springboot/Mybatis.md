# Mybatis
## yml配置
> 基于springboot默认的自动配置策略，当引入持久化框架时，自动读取配置中的数据源配置，没有配置数据源则抛出异常**无法启动**
``` java
//MySQL连接基本配置
spring:
  application:
    name: mybatis-examples
  datasource:
    url: 'jdbc:mysql://114.116.213.241:3306/2018214247?
          createDatabaseIfNotExist=true
          &serverTimezone=Asia/Shanghai'
    username: 2018214247
    password: 2018214247
    driver-class-name: com.mysql.cj.jdbc.Driver
    initialization-mode: always

//声明日志配置
logging:
  level:
    root: warn
    com:
      example: debug
  pattern:
    console: '%-5level %C.%M[%line] - %msg%n'
```
## Mapper
### @Repository
> 声明为spring持久化组件(不声明使用时提示错误，但运行正常)
### @Mapper
> 声明为mybatis映射接口。运行时动态生成接口代理类
```java 
@Repository
@Mapper
public interface UserMapper01 {

}
```
### @Insert
>* 修饰接口中的方法。声明预执行的SQL insert语句
>* **#{}**，占位符。支持自动从传入的对象属性名称/参数名称提前对应的值
```java 
@Insert("insert into user(id,name,company) values (#{id},#{name},#{company})")
    public void insert(User user);
```
### 编写测试类
```java
@SpringBootTest
//声明为事务类
@Transactional
//防止数据回滚
@Rollback(value = false)
//声明为日志
@Slf4j
public class UserMapper01Test {
    @Autowired
    private UserMapper01 userMapper01;

    @Test
    public void test_adduser(){
        User user = new User();
        //添加user的信息
        user.setId(2L);
        user.setName("SUN");
        user.setCompany("facebook");
        userMapper01.insert(user);
    }
}
```
### @Select
> 修饰接口中的方法。声明预执行的SQL select语句
```java
@Select("select * from user")
//支持自动封装成集合
List<User> list();
```
### @Param
>方法参数注解。声明SQL语句中占位符对应的方法参数。当方法参数/占位符名称相同时，可省略
### BaseMapper
>BaseMapper<T>，提供了通用CURD操作的接口
>* T insert(T entity)
>* int deleteById(Serializable id)
>* int deleteByMap(Map<String, object> columnMap)
>* Int updateById(T entity)
>* T selectById(Serializable id)
>* List<T> selectBatchIds( Collection<? extends Serializable> idList)
```java
@Repository
@Mapper
public interface UserMapper02 extends BaseMapper<User> {
    //接口中没有任何声明方法
}

//测试类
    @Test
    public void adduser_tset(){
        User user = new User();
        //继承BaseMapper接口，可直接添加信息
        user.setName("SUN");
        user.setCompany("amazon");
        userMapper02.insert(user);
    }

    @Test
    public void update_test(){
      // 更新信息，通过builder()创建对象，再通过updateById()更新信息
        User user = User.builder()
                .id(1L)
                .company("nike")
                .build();
        userMapper02.updateById(user);
    }
```
>#### 简单的关联查询
>1. 创建一个独立的DTO类，可以封装user/address的全部属性
```java
@Data
public class AddressDTO04 {
    private String name;
    private String company;
    private LocalDateTime userCreateTime;
    private Long id;
    private String detail;
    private Long userId;
    private LocalDateTime addressCreateTime;
}

```
>2. 可直接将结果封装在DTO中,结果字段名称与DTO中属性名称不同的，必须显式指定。基于SQL指定段的别名为DTO属性名
```java
    //属性名称与DTO中的名称对应上
    @Select("select *," +
            "a.create_time addressCrestTime," +
            "u.create_time userCreatTime," +
            "a.id id " +
            "from address a join user u " +
            "on u.id = a.user_id" +
            " where a.detail=#{detail}")
    // 封装在创建的DTO类中
    List<AddressDTO04> list(String detail);
```
## One-to-many
> 需求：基于用户ID获取用户基本信息以及所有地址信息
* 创建DTO类，里面包含user的属性和address信息的集合
```java
@Data
@NoArgsConstructor
public class UserDTO05 {
    private Long id;
    private String name;
    private String company;
    private List<Address> addresses;
}
```
### @Result
> 声明字段属性映射规则
>* Column，字段
>* Property，映射的属性
>* javaTpe，映射类型
### 配置XML文件
* 基于原生xml映射文件的实现
* 创建resources/mapper目录。启动时自动扫描加载此目录下映射配置
* xml文件名称建议与Mapper接口同名，一一对应便于维护
* mapper映射器标签。namespace属性声明为Mapper接口的全限定性名称。即，此映射配置支持的指定接口
```java
// xml文件声明版本和配置
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
//<mapper>映射器 namespace属性为Mapper接口的全限定性名称
<mapper namespace="com.example.mybatisexamples.example05.UserMapper05">
```
### select标签
>* **Id**，映射规则名称/标识。必须声明对应名称的方法
>* **resultMap**，结果集映射规则namespace+ID。复杂结果由独立映射规则处理
>* **resultType**，查询结果直接映射为指定类型
>* **parameterType**，查询参数类型。支持idea提示

```java
// 在接口中声明与id同名方法
<select id="getByXML2" resultMap="userDTOResultMap">
        select u.*,
               a.id a_id,
               a.detail a_detail,
               a.user_id a_user_id,
               a.create_time a_create_time
        from user u join address a on u.id=a.user_id
        where u.id=#{uid}
</select>
```
### resultMap标签
> 声明映射规则；
>* id规则名称；
>* type映射类型；
>* autoMapping是否开启自动映射
>* id和result是常用的两个**子标签**，id用于匹配属性和主键字段的映射
>>* column 是**表**的字段名
>>* property 是**类**的属性名 
>* collection，将部分结果映射为集合属性(many端)
>>* Property，集合属性名称
>>* columnPrefix，结果集中段的统一前缀
>>* ofType，集合中元素类型
>* accociation，将指定部分结果映射为复合属性(one端)。使用与collection相似
>>* type，对one段的映射类型

```java
// id与select标签中的resultMap名称相对应
// type是结果映射类型的全限定性名称
<resultMap id="userDTOResultMap" type="com.example.mybatisexamples.example05.UserDTO05">
        <id column="id" property="id" />
        <result column="name" property="name" />
        <result column="company" property="company" />
        // 映射属性是一个集合类型
        //将结果中前缀为a_的自动基于另一映射规则映射将映射的结果置于addresses属性
        <collection property="addresses"
                    // 映射是忽略前缀
                    columnPrefix="a_"
                    // 指定映射规则namespace+映射规则名称
                    resultMap="com.example.mybatisexamples.example05.AddressMapper05.addressResultMap">
        </collection>
</resultMap>
```
### 此映射也可卸载同一个xml文件中
```java
// 另一个addressMapper.XML，对应上面的address的结果映射
<mapper namespace="com.example.mybatisexamples.example05.AddressMapper05">
//id与上面的address结果对应
<resultMap id="addressResultMap" 
           type="com.example.mybatisexamples.entity.Address" 
           //true，开启自动映射
           autoMapping="true" />
```
* UserMapper05接口中声明ID同名方法
无需其他注解配置
* 基于注解寻找执行方法的策略是，全限定名(namespace)+方法名(id)，**不包含参数列表**。因此，使用时**不支持方法重载**
### 动态查询
### AbstractWrapper
>#### 基于MyBatis Dynamic SQL API原生查询接口，构建较复杂
>* 包含多种查询方法
>>* eq()等于
>>* ne()不等于
>>* ge()大于等于
>>* gt()大于
>>* lt()小于
>>* le()小于等于
>>* between()
>>* like()
>>* QueryWrapper/UpdateWrapper类，基于数据表字段
>>>* LambdaQueryWrapper类，基于表达式映射属性/字段名称
>* 创建一个包含多种查询数据的类
用于模拟传入的查询数据
```java
@Builder
@Data
@NoArgsConstructor
@AllArgsConstructor
public class QueryOptional {
    private String gender;
    private Integer followers;
    private Integer stars;
    private Integer repos;
    private LocalDateTime beforeCreateTime;
}
```
>* 在mapper中声明default方法
```java
                                        //接受查询封装的参数对象
default List<GithubUser> listByOptionals(QueryOptional optionals){
        // 创建基于lambda表达式的查询对象
        LambdaQueryWrapper<GithubUser> qw = new QueryWrapper<GithubUser>().lambda();
}
```
>* 当本次查询提供了stars参数时，动态生成基于stars的查询语句
>* ge(lambda, value)方法。 通过表达式映射属性名称到对应的字段名称。否则需要使用字符串声明字段名称
```java
if(optionals.getStars()!=null){
                  //等效于op->op.getStars()
            qw.ge(GithubUser::getStars,optionals.getStars());
        }
```
>* 最后，调用BaseMapper中支持wrapper的方法，基于查询对象，执行查询
```java
return selectList(qw);
```
>* 测试类
```java
@Test
    public void guthub_test(){
        // 创建查询数据的类
        QueryOptional queryOptional = QueryOptional.builder()
                .stars(10)
                .repos(10)
                .beforeCreateTime(LocalDateTime.of(2018,1,5,2,25))
                .build();
        // 执行查询的方法
        List<GithubUser> githubUsers = githubUserMapper06.listByOptionals(queryOptional);
        for (GithubUser g : githubUsers) {
            log.debug("{}/{}",
                    g.getId(),
                    g.getName());
        }
    }
```
#### 基于MyBatis xml Dynamic SQL支持动态关联查询
```java
        // 属性为mapper接口的全限定性名称
<mapper namespace="com.example.mybatisexamples.example06.GithubUserMapper06">
    <select id="githubUserResult"
            // 声明查询方法参数类型，可直接使用
            parameterType="com.example.mybatisexamples.example06.QueryOptional"
            resultType="com.example.mybatisexamples.entity.GithubUser">
                 //where 1=1 不知道拼接多少语句，省去多余的or/and
        select * from github_user g where 1=1
        <if test="stars != null">
            and g.stars>=#{stars}
        </if>
        <if test="repos != null">
            and g.repos>=#{repos}
        </if>
        <if test="beforeCreateTime != null">
            and g.create_time<=#{beforeCreateTime}
        </if>
        <if test="followers != null">
            and g.followers>=#{followers}
        </if>
        <if test="gender != null">
            and g.gender=#{gender}
        </if>
    </select>
</mapper>
```