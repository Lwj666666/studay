# Spring
>* **@Component**，声明为组件
>* **@Repository**，声明为持久化组件(a mechanism for encapsulating storage, retrieval, and search behavior which emulates a collection of objects)
>* **@Service** ，声明为业务逻辑组件("an operation offered as an interface that stands alone in the model, with no encapsulated state)
>* **@Controller**，声明为逻辑控制组件
>* **@Bean**，方法级，声明方法返回的对象，由Spring容器管理。当需注入非自定义组件时
>* **@Configuration**，声明为配置类组件
* **组件注解，仅用于区分组件功能**
* **注解本身，不实现任何特殊功能**
* **单例 (Singleton)**，容器中仅存在bean的一个实例。生命周期由容器管理；即容器对所有注入请求使用相同实例
* **原型(Prototype)**，容器不保存bean的实例，每次请求容器均创建一个新的实例
### AOP Concepts
* **AOP(Aspect-Oriented Programming)**：面向对象编程(OOP)模式的扩展，一种程序设计思想，使调用者与被调用者之间解耦
* **Joinpoint(连接点)**。一个预，添加/插入扩展操作的方法
* **Pointcut(切入点)**。通过表达式，描述一组具有共同性的连接点。往往预扩展的方法不是一个方法，而是一组具有共同性的很多方法。例如
>* 对所有remove前缀方法的执行，均记录日志
>* 对所有@Transaction注解方法的执行，均启用事务
* **Advice(通知)**。对单个连接点或切入点(一组连接点)，采取的扩展操作行为。例如，记录日志行为/事务行为/权限验证行为
* **Weaving(织入)**。将通知连接到连接点，创建代理对象的过程
* **Target Object(目标对象)**。连接点被织入通知后，生成的代理类创建的对象。Spring AOP通过运行时代理实现，因此必然是一个代理对象
* **Aspect(切面)**。一个支持对横跨多个模块功能，实现通用统一扩展的行为。例如，事务处理切面/日志管理切面/权限管理切面等
* **@Before** Before advice，前置通知，在切入点方法执行前通知
* **@AfterReturning** After returning advice，后置返回通知，在切入点方法执行并返回后执行，方法抛出异常时不执行
* **@AfterThrowing** After throwing advice:后置异常通知，在切入点方法抛出异常时执行
* **@After** After(finally) advice，后置最终通知，在切入点方法执行后，后置通过前执行，即使抛出异常也执行
* **@Around** Around advice，环绕通知，在切入点方法执行前、或执行后执行，可阻止目标方法的执行，替换参数等
* **@Aspect(org.aspectj.lang.annotation.Aspect)**
>* 声明为切面类声明切面类为组件，由容器扫描创建
>* 声明切入点，也可将表达式单独声明在通知
```java
@Pointcut("execution(* com.example.springexamples.example01.AOPServiceImpl.*(..))")
public void pointcut(){}
```
>* 基于相应切入点方法名称，声明各种通知
* **@annotation**，连接点方法包含指定注解
>* @annotation(org.springframework.transaction.annotation.Transactional)
* **@args**，连接点方法内参数类型包含指定注解，而不是参数的注解
@args(com.xyz.security.Classified)
* **@within**，连接点类型包含指定注解(RetentionPolicy.CLASS级注解)
>* **@within**(org.springframework.transaction.annotation.Transactional)
### JoinPoint(org.aspectj.lang.JoinPoint)
* 仅可以获取信息，无法改变连接点的执行
>* Object getTarget()，获取连接点对象
>* Object[] getArgs()，获取方法参数
>* Object getThis()，获取代理对象
>* Signature getSignature()，获取方法前面
### ProceedingJoinPoint(org.aspectj.lang.ProceedingJoinPoint)
* 继承JoinPoint，仅可注入around通知
>* Object proceed() ，执行连接点方法，并返回执行结果，**不执行则不会调用连接点方法**
>* Object proceed(Object[] args)，基于指定参数执行连接点对象方法
* **Around通知可，改变连接点方法的执行，修改原参数，修改返回结果等**
* 接口对象可直接作为通知方法参数，注入使用
```java
@Around("pointcut()")
                         //直接注入参数对象
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        // 获取参数
        Object[] objects = joinPoint.getArgs();
        if(objects[0].equals("BO")){
            // 替换第一个参数对象
            objects[0]="SUN";
        }
        // 基于新的参数对象执行，并返回参数结果
        Object object = joinPoint.proceed(objects);
        log.debug("返回的类型为：{}",object.getClass());
        return object;
    }
```