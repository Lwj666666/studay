# spring
## Beans的理解
### 作用域
* **singleton:**
> 容器中只会存在一个共享的Bean实例
* **prototype:**
> 每次通过容器获取时，容器都会创建一个新的对象
* **request:**
> 对不同的HTTP请求，都会产生一个新的Bean，并且只会在当前HTTP内生效
* **session:**
> 再一次HTTP session请求中，容器会返回该bean的同一实例
* **global Session：**
> 在一个全局的HTTP Session中，容器会返回Bean的同一个实例
### 什么是Spring Bean？
* 他是构成应用程序主干的对象
* Bean由Spring IOC来管理
* 它们由Spring IOC容器实例化，配置，装配和管理
* Bean是基于用户提供给容器的配置元数据创建
### Spring提供了哪些配置方式
* 基于XML配置
* 基于注解配置
* 基于Java API配置
>*  Spring的Bean是通过@Bean和@Configuration来实现
>* @Configuration类通过简单地调用同一个类的其他@Bean方法来定义Bean中的依赖关系
### 什么是自动配置
> Spring会在容器中自动的查找，并自动的给bean装配及其关联的属性
## 谈谈理解
* 对于spring，核心就是IOC容器，这个容器就是把里面的对象进行统一管理，你不需要考虑对象如何创建和销毁
* 至于AOP，这个类就是切面（Aspect），这个被环绕的方法就是切点（Pointcut），你所做的执行前执行后的这些方法统一叫做增强处理（Advice）。
## AOP的原理
* 动态代理
## DI是什么
* 依赖注入
## Spring的优缺点
* 方便解耦 简化开发
* spring就是一个大工厂，可以将所有对象的创建和维护，都交给Spring管理
* AOP编程
* 支持事务
* 方便对程序的测试
## 常用的注解
### 给容器注入组件
#### 包扫描＋组件标注注解
1. component：泛指各种组件
2. Controller: 控制层
3. Service：业务层
4. Repository：
#### 导入第三方包的注解
1. import
#### 注入的注解
1. Autowired
2. Resource
#### 配置类的注解
1. Configuration：声明当前类为配置类
2. Bean：声明在方法上，声明当前方法返回值为一个bean，替代XML的方式
3. @ComponentScan：用于对component进行扫描
#### 切面类的注解
1. Aspect：声明为一个切面类
2. Pointcut：声明切点
3. after before around
## Autowired和Recourse的区别
1. Autowired主要是基于类型进行注入
2. Resource主要是基于名称进行注入
## 什么是spring IOC
* **控制反转**：将程序主动创建对象改为被动接受所需对象
>* 控制：创建所需对象的控制权
>* 将主动被动关系反转
* 管理对象的创建和依赖关系的维护
* 解耦，由容器去维护具体的对象
* 负责生成和管理代理类
* 可以实现依赖关系注入的容器，负责实例化/定位/配置应用程序所需对象，建立对象间的依赖关系，组装应用程序组件。
## IOC的优点
* IOC 把应用的代码量降到最低
* 使应用容易测试
* 以最小的代价和最小的侵入性实现松散耦合
* IOC支持饿汉加载和懒汉加载
## IOC支持哪些功能
* 依赖注入
* 依赖检查
* 自动装配
* 支持集合
## 代理类
* 静态代理
  * 编译时自动生成代理类
* 动态代理
  * JDK动态形成代理
  >* 原生动态代理是Java原生支持的，不需要任何外部依赖，但是它只能基于接口进行代理
  * 通过CGLIB工具实现代理
  >* 通过继承的方式进行代理（让需要代理的类成为Enhancer的父类），无论目标对象有没有实现接口都可以代理，但是无法处理final的情况。
## Spring用的啥
> Spring会根据具体的Bean是否具有接口去选择动态代理方式，如果有接口，使用的是Jdk的动态代理方式，如果没有接口，使用的是cglib的动态代理方式。
## 什么是AOP
> * 面向对象编程，一种程序设计思想，是调用者与被调用者之间解耦
>* 过程：
>   1.  Aspect声明切面类为组件，容器扫描创建 
>   2. Pointcut声明切入点
>   3. 声明各种通知 around before after..
## ResultMap和ResultType的区别
>* ResultMap是结果映射集
>* ResultType是直接返回结果类型
