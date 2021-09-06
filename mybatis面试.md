## 执行过程
1. 读取mybatis的核心配置文件
2. 创建SqlSessionFactoryBuilder对象
3. 创建SqlSessionFactory对象
4. 获取SqlSession对象
5. 通过SqlSession对象插入数据
6. 关闭SqlSession对象
## mybatis原理
1. 运行开始时，先通过Resources加载全局配置文件
2. 实例化SqlsessionFactoryBuilder构建器，帮助SqlSessionFactory接口实现类DefaultSqlSessionFactory,在实例化DefaultSqlSessionFactory之前需要先创建XmlConfigBuilder解析全局配置文件，并把解析结果存放在Configuration中，之后把Configuration传递给DefaultSqlSessionFactory,到此SqlSessionFactory工厂创建成功
3. 由SqlSessionFactory工厂创建SqlSession,每次创建SqlSession时，都需要由TransactionFactory创建Transaction对象,同时还需要创建SqlSession的执行器Excotor,最后实例化DefaultSqlsession,传递给SqlSession接口
4. 根据项目的需求使用SqlSession接口中的API完成具体的事务操作，如果事务执行失败，需要进行rollback回滚，如果事务执行成功提交给数据库，关闭SqlSession
```java
public static void main(String[] args) {
		InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
		SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
		SqlSession sqlSession = factory.openSession();
		String name = "tom";
		List<User> list = sqlSession.selectList("com.demo.mapper.UserMapper.getUserByName",params);
}
```