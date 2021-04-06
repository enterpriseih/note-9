### MyBatis面试题

#### 1、缓存

##### 1.1  什么是查询缓存

​		MyBatis提供查询缓存，用于减轻数据压力，提高数据性能。MyBatis提供一级缓存和二级缓存。

​		一级缓存是SqlSession级别的缓存。在操作数据库时需要构造一个sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession 之间的缓存数据区域（hashMap）是互不影响的。
​		二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用一个二级缓存，二级缓存是跨SqlSession的。



##### 1.2  一级缓存工作原理

​		第一次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，如果缓存没有，从数据库查询用户信息。得到用户信息，将用户信息存储到一级缓存中。

​		如果sqlSession去执行commit操作（执行添加、更新、删除），情况sqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏数据。

​		第二次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，缓存有，直接从缓存中获取用户信息。



##### 1.3  一级缓存的应用

​		在正式开发中，是将mybatis和spring进行整合开发，事物控制在service中。一个service方法中包括很多mapper方法调用。

​		Service{
​				//开始执行时，开启事物，创建生气了Session对象
​				//第一次调用mapper的方法findUserById(1)

​				//第二次调用mapper的方法findUserById(1)，从一级缓存中取数据

​				//方法结束，sqlSession关闭

​		}

​		如果是执行两次service调用查询相同的用户信息，不走一级缓存，因为session方法结束，sqlSession就关闭了，一级缓存就清理空了。



##### 1.4  二级缓存原理

​		首先开启mybatis的二级缓存。

​		sqlSession1 去查询用户id为1的用户信息，查询到用户信息会将查询到的数据存储到二级缓存中如果sqlSession3去执行相同mapper下sql执行commit提交，清空mapper下的二级缓存区域的数据sqlSession2去查询用户id为1的用户信息，去缓存中找是否存在数据，如果存在直接从缓存中取中数据二级缓存与一级缓存区别，二级缓存的范围更大，多个sqlSssion可以共享一个userMapper的二级缓存区域。

​		userMapper有一个二级缓存区域（按namespance分）。其他mapper也由自己的二级缓存区域（按namespance分）每个namespance的mapper都有一个二级缓存区域，两个mapper的namespance如果相同，这两个mapper执行sql查询到的数据将存在相同的二级缓存区域中



#### 2、#{}和${}的区别是什么

​		${}是变量占位符，可以用于标签属性值和sql内部，属于静态文本替换。\#{}是sql的参数占位符，MyBatis会将sql中的#{}替换为?号，在sql执行前会使用PreparedStatement的参数设置方法，按序给sql的?号占位符设置参数值。



#### 3、为什么说 MyBatis是半自动ORM映射工具？它与全自动的区别在哪里？

​		MyBatis在查询关联对象或关联集合对象时，需要手动编写sql来完成，称之为半自动ORM映射工具。Hibernate属于全自动ORM映射工具，根据查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取。

