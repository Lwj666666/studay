## 大作业
* **教师** 姓名  年龄 性别 手机号
* **课程** 名称 授课教师 课时 学生个数
* **实验室** 教室号 座位数 配置
* **实验课** 上课教室 第几周 星期几 第几节课 上课教师 课程
* **用户** 用户 密码
* **管理员** 姓名  年龄 性别 手机号
### 测试链接
http://localhost:8080/swagger-ui/index.html
### 管理员/教师登陆
 ```java
POST http://localhost:8080/api/login
Content-Type: application/json

{
  "name": "王波",
  "password": "123456"
}
```
#### 实现过程
>* 创建一个非对称加密的类 并且对异常进行统一处理
>* 获取用户名和密码之后，进行验证
>* 验证成功，返回包含教师信息并且经过加密处理的Token
* **登录模块**
>* 密码验证成功后，将用户的信息封装到token中加密后放到HTTP请求的header上面
```java
@Api(value = "登录请求")
@RestController
@RequestMapping("/api/")
public class LoginController {

    @Autowired
    public UserService userService;

    @Autowired
    public PasswordEncoder encoder;

    @Autowired
    public EncryptComponent encryptComponent;

    @ApiOperation(value = "登录")
    @PostMapping("login")
    public ResultVO login(@RequestBody User user, HttpServletResponse response) {
        User u = userService.getUser(user.getName());
        if (u == null || !encoder.matches(user.getPassword(), u.getPassword())) {
            return ResultVO.error(401, "用户名密码错误");
        }
        String token = encryptComponent.encrypt(Map.of("username", u.getName(), "role", u.getRole()));
        response.addHeader("token", token);
        return ResultVO.success(Map.of(u.getName()+u.getRole(),"登陆成功"));
    }
}
```
* **请求拦截模块**
> **全局控制，声明每一个拦截器的拦截范围**
```java
@Configuration
public class WebMvcConfiguration implements WebMvcConfigurer {
    @Autowired
    public LoginInterceptor loginInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginInterceptor)
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/login");
    }
}
```
* **对token进行非对称加密和解密的过程**
```java
@Component
public class EncryptComponent {
    @Autowired
    private ObjectMapper objectMapper;
    @Value("${my.secretkey}")
    private String secretKey;
    @Value("${my.salt}")
    private String salt;
    @Autowired
    private TextEncryptor encryptor;

    /**
     * 直接基于密钥/盐值创建单例TextEncryptor对象。避免反复创建
     * @return
     */
    @Bean
    public TextEncryptor getTextEncryptor() {
        return Encryptors.text(secretKey, salt);
    }

    public String encrypt(Map<String, Object> payload) {
        try {
            String json = objectMapper.writeValueAsString(payload);
            return encryptor.encrypt(json);
        } catch (JsonProcessingException e) {
            throw new MyException(500, "服务器端错误");
        }
    }

    /**
     * 无法验证/解密/反序列化，说明数据被篡改，判定无权限
     * @param auth
     * @return
     */
    public Map<String, Object> decrypt(String auth) {
        try {
            String json = encryptor.decrypt(auth);
            return objectMapper.readValue(json, Map.class);
        } catch (Exception e) {
            throw new MyException(403, "无权限");
        }
    }
}
```
* **对异常进行统一处理**
```java
@Slf4j
@RestControllerAdvice
public class ExceptionController {
    @ExceptionHandler(MyException.class)
    public ResultVO handleValidException(MyException exception) {
        return ResultVO.error(exception.getCode(), exception.getMessage());
    }
    /**
     * 属性校验失败异常
     *
     * @param exception
     * @return
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResultVO handleValidException(MethodArgumentNotValidException exception) {
        StringBuilder strBuilder = new StringBuilder();
        exception.getBindingResult()
                .getFieldErrors()
                .forEach(e -> {
                    strBuilder.append(e.getField());
                    strBuilder.append(": ");
                    strBuilder.append(e.getDefaultMessage());
                    strBuilder.append("; ");
                });
        return ResultVO.error(400, strBuilder.toString());
    }


    /**
     * 请求类型转换失败异常
     *
     * @param exception
     * @return
     */
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResultVO handleMethodArgumentTypeMismatchException(
            MethodArgumentTypeMismatchException exception,
            HttpServletRequest request) {
        String msg = request.getRequestURI() +
                ": " + "请求地址参数" + exception.getValue() + "错误";
        return ResultVO.error(400, msg.toString());
    }

    /**
     * jackson json类型转换异常
     *
     * @param exception
     * @return
     */
    @ExceptionHandler(InvalidFormatException.class)
    public ResultVO handleInvalidFormatException(InvalidFormatException exception) {
        StringBuilder strBuilder = new StringBuilder();
        exception.getPath()
                .forEach(p -> {
                    strBuilder.append("属性");
                    strBuilder.append(p.getFieldName());
                    strBuilder.append("，您输入的值：").append(exception.getValue());
                    strBuilder.append(", 类型错误！");
                });
        return ResultVO.error(400, strBuilder.toString());
    }

    /**
     * 方法级参数校验失败异常
     *
     * @param exception
     * @return
     */
    @ExceptionHandler(ConstraintViolationException.class)
    public ResultVO handleConstraintViolationException(ConstraintViolationException exception) {
        StringBuilder strBuilder = new StringBuilder();
        Set<ConstraintViolation<?>> violations = exception.getConstraintViolations();
        violations.forEach(v -> {
            strBuilder.append(v.getMessage()).append("; ");
        });
        return ResultVO.error(400, strBuilder.toString());
    }

    @ExceptionHandler(Exception.class)
    public ResultVO handleException(Exception exception) {
        return ResultVO.error(400, "请求错误");
    }
}
```
* **拦截器**
> **拦截到HTTP请求后，取出请求的header（也就是登录时候返回的token）并进行解密（反序列化），再将解密出来的信息存到HTTP请求中**
>* **@RequestAttribute**用于获取HTTP请求中指定名称的信息
```java
@Component
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {
    @Autowired
    private EncryptComponent encryptComponent;
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String token = request.getHeader("token");
        if (token == null) {
            throw new MyException(401, "未登录");
        }
        Map<String, Object> result = encryptComponent.decrypt(token);
        request.setAttribute("username",result.get("username"));
        request.setAttribute("role",result.get("role"));
        return true;
    }
}
```
### 实验课的查询和添加
>#### 查询(基于教室)
> * 创建一个类LabClassesRe，用于接收实验室查询信息的结果
```java
@Data
public class LabClassesRe {

    private Integer labClass;
    private String teacherName;
    private Integer weekNumMin;
    private Integer weekNumMax;
    private Integer week;
    private Integer classNum;
    private String courseName;

}
```
> * XML配置文件，映射查询
```java
<mapper namespace="com.example.labmanage.mapper.LabClassesMapper">
    <select id="listLabClassByC" resultType="com.example.labmanage.entity.LabClassesRe">
        select MAX(teacher_name) teacherName ,
               course_name courseName ,
               MIN(week_num) weekNumMin,
               MAX(week_num) weekNumMax,
               MAX(week) week,
               MAX(class_num) classNum
        from lab_classes where lab_class=#{labClass} group by course_name,week,class_num
    </select>
</mapper>
```
> * 返回查询结果，并对结果进行缓存
```java
@ApiOperation("输入实验室名 查询实验室的预约信息")
@PostMapping("labclasses")
@Cacheable(value = "labclasses",key = "#lab.labClass")
public ResultVO getLabClass(@RequestBody Lab lab){
    List<LabClassesRe> labClasses = labClassesMapper.listLabClassByC(lab.getLabClass());
    return ResultVO.success(Map.of(lab.getLabClass()+"",labClasses));
}
```
>#### 添加
>* 输入的信息是LabClassesRe类
>* 删除缓存的实验室信息
```java
public void addlabclasses(LabClassesRe labClassesRe,String name){
    for(int i=labClassesRe.getWeekNumMin();i<=labClassesRe.getWeekNumMax();i++){
        LabClasses classes = LabClasses.builder()
                .labClass(labClassesRe.getLabClass())
                .teacherName(name)
                .courseName(labClassesRe.getCourseName())
                .weekNum(i)
                .week(labClassesRe.getWeek())
                .classNum(labClassesRe.getClassNum())
                .build();
        labClassesMapper.insert(classes);
    }
}
```
### XML的动态更新语句
>* 基于trim实现
>>* prefix="set" 省略更新语句中的set
>>* suffixOverrides 删除trim语句中末尾的逗号
```java
<mapper namespace="com.example.labmanage.mapper.TeacherMapper">
    <delete id="deleteByName">
        delete from teacher where name=#{name}
    </delete>
    <update id="updateByName" parameterType="com.example.labmanage.entity.Teacher">
        update teacher
        <trim prefix="set" suffixOverrides=",">
            <if test="telephone != null">telephone=#{telephone}, </if>
            <if test="sex != null">sex=#{sex} ,</if>
            <if test="age != null">age=#{age} ,</if>
        </trim> where name = #{name}
    </update>
</mapper>
```
* 缓存再次查询的时候
* 把name设置成主键
* 声明一个接收结果的类