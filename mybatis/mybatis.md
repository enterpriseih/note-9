### MyBatis

基于ORM（Object Relation Mapping）的半自动轻量级持久层框架



#### 一、MyBatis基础应用

pom.xml

mapper.xml

* mapper
  * namespqce
* select、insert、update、delete
  * id
  * resultType
  * parameterType

核心配置文件sqlMappConfig.xml

1. 约束头

   报错URI is not registered (Settings | Languages & Frameworks | Schemas and DTDs)，因为idea统一资源标识符没有注册

   ![image-20210131234402912](mybatis.assets/image-20210131234402912.png)

   ![image-20210131234649589](mybatis.assets/image-20210131234649589.png)

1. 标签
   
   * properties
   * configuration
   * typeAlias
   * environments
     * default
   * environment
     * id
   * transactionManger
   * dataSource
   * mappers

##### 2、MyBatis执行的API

```java
    @Test
    public void test1() throws IOException {
        // 通过Rescoures工具类加载sqlMapConfig.xml成字节流
        InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
        // 解析传入的配置文件字节流，创建sqlSession工厂
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder()
            .build(resourceAsStream);
        // 生产一个sqlSession，默认开启事务，执行增删改时不会自动提交，需要执行sqlSession.commit()
        // sessionFactory.openSession(true) 在构造方法中传true，开启自动提交
        SqlSession sqlSession = sessionFactory.openSession();
        // 执行sqlSession增删改查方法
 		List<User> userList = sqlSession.selectList("user.findAll");
		userList.forEach(System.out::println);

        User user = new User();
        user.setId(5);
        user.setUsername("ying");
		sqlSession.insert("user.saveUser", user);

		user.setUsername("yy");
        sqlSession.update("user.updateUser", user);
        sqlSession.delete("user.deleteUser", user);
        sqlSession.commit();
    }
```

##### 3、MyBatis的dao层代理开发

mapper标签的namespace为dao接口的相对路径

select、update、insert、delete标签的id为dao接口的对用方法



