## 1. JDBC编程有哪些不足之处，MyBatis是如何解决这些问题的？
1、数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。 解决：在SqlMapConfig.xml中配置数据链接池，使用连接池管理数据库链接。

2、Sql语句写在代码中造成代码不易维护，实际应用sql变化的可能较大，sql变动需要改变java代码。 解决：将Sql语句配置在XXXXmapper.xml文件中与java代码分离。

3、 向sql语句传参数麻烦，因为sql语句的where条件不一定，可能多也可能少，占位符需要和参数一一对应。 解决： Mybatis自动将java对象映射至sql语句。

4、 对结果集解析麻烦，sql变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成pojo对象解析比较方便。 解决：Mybatis自动将sql执行结果映射至java对象。

## 2. MyBatis编程步骤是什么样的？
1、创建SqlSessionFactory 
2、通过SqlSessionFactory创建SqlSession
3、 通过sqlsession执行数据库操作
4、 调用session.commit()提交事务 
5、 调用session.close()关闭会话

## 3. MyBatis与Hibernate有哪些不同？
1、Mybatis 和 hibernate 不同，它不完全是一个 ORM 框架，因为 MyBatis 需要 程序员自己编写 Sql 语句。 

2、Mybatis 直接编写原生态 sql，可以严格控制 sql 执行性能，灵活度高，非常 适合对关系数据模型要求不高的软件开发，因为这类软件需求变化频繁，一但需 求变化要求迅速输出成果。但是灵活的前提是 mybatis 无法做到数据库无关性， 如果需要实现支持多种数据库的软件，则需要自定义多套 sql 映射文件，工作量大。 

3、Hibernate 对象/关系映射能力强，数据库无关性好，对于关系模型要求高的 软件，如果用 hibernate 开发可以节省很多代码，提高效率 

## 4. Mybaits 的优点：

1、基于 SQL 语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任 何影响，SQL 写在 XML 里，解除 sql 与程序代码的耦合，便于统一管理；提供 XML 标签，支持编写动态 SQL 语句，并可重用。

2、与 JDBC 相比，减少了 50%以上的代码量，消除了 JDBC 大量冗余的代码，不 需要手动开关连接； 

3、很好的与各种数据库兼容（因为 MyBatis 使用 JDBC 来连接数据库，所以只要 JDBC 支持的数据库 MyBatis 都支持）。

4、能够与 Spring 很好的集成； 5、提供映射标签，支持对象与数据库的 ORM 字段关系映射；提供对象关系映射 标签，支持对象关系组件维护

## 5. MyBatis 框架的缺点： 

1、SQL 语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写 SQL 语句的功底有一定要求。

2、SQL 语句依赖于数据库，导致数据库移植性差，不能随意更换数据库。

## 6. #{}和${}的区别？
{}是预编译处理，${}是字符串替换。
Mybatis在处理#{}时，会将sql中的#{}替换为？号，调用PreparedStatement的set方法来赋值；

Mybatis在处理${}时，就是把${}替换成变量的值。

使用#{}可以有效的防止SQL注入，提高系统安全性。

## 7. 通常一个Xml映射文件，都会写一个Dao接口与之对应，那么这个Dao接口的工作原理是什么？Dao接口里的方法、参数不同时，方法能重载吗？
Dao接口即Mapper接口。接口的全限名就是映射文件中的namespace的值；接口的方法名，就是映射文件中Mapper的Statement的id值；接口方法内的参数，就是传递给sql的参数。Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名的拼接字符串作为key值，可唯一定位一个MapperStatement。

Dao接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略。

Dao接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。
## 8. 在Mapper中如何传递多个参数？
1、若Dao层函数有多个参数，那么其对应的xml中，#{0}代表接收的是Dao层中的第一个参数，#{1}代表Dao中的第二个参数，以此类推。

2、使用@Param注解：在Dao层的参数中前加@Param注解,注解内的参数名为传递到Mapper中的参数名。

3、多个参数封装成Map，以HashMap的形式传递到Mapper中。
## 9. Mybatis动态sql有什么用？执行原理是什么？有哪些动态sql？
Mybatis动态sql可以在xml映射文件内，以标签的形式编写动态sql，执行原理是根据表达式的值完成逻辑判断，并动态拼接sql的功能。

Mybatis提供了9种动态sql标签：trim、where、set、foreach、if、choose、when、otherwise、bind
## 10. xml映射文件中，不同的xml映射文件id是否可以重复？
不同的xml映射文件，如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复；

原因是namespace+id是作为Map<String,MapperStatement>的key使用的，如果没有namespace，就剩下id，那么id重复会导致数据互相覆盖。有了namespace，自然id就可以重复，namespace不同，namespace+id自然也不同。
## 11. Mybatis实现一对一有几种方式？具体是怎么操作的？
有联合查询和嵌套查询两种方式。

联合查询是几个表联合查询，通过在resultMap里面配置association节点配置一对一的类就可以完成；

嵌套查询是先查一个表，根据这个表里面的结果的外键id，再去另外一个表里面查询数据，也是通过association配置，但另外一个表的查询是通过select配置的。
## 12. Mybatis实现一对多有几种方式？具体是怎么操作的？
有联合查询和嵌套查询两种方式。

联合查询是几个表联合查询，只查询一次，通过在resultMap里面的collection节点配置一对多的类就可以完成；

嵌套查询是先查一个表，根据这个表里面的结果的外键id，再去另外一个表里面查询数据，也是通过collection，但另外一个表的查询是通过select配置的。
## 13. Mybatis的一级、二级缓存
1、 一级缓存：基于PerpetualCache的HashMap本地缓存，其存储作用域为Session，当Session flush或close之后，该Session中的所有Cache就将清空，默认打开一级缓存。
2、 二级缓存与一级缓存机制相同，默认也是采用PerpetualCache，HashMap存储，不同在于其存储作用域为Mapper（namespace），并且可自定义存储源，如Ehcache。默认打不开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口（可用来保存对象的状态），可在它的映射文件中配置。

对于缓存数据更新机制，当某一个作用域（一级缓存Session/二级缓存Namespace）进行了增/删/改操作后，默认该作用域下所有select中的缓存将被clear。
## 14. 使用MyBatis的Mapper接口调用时有哪些要求？
1、Mapper接口方法名和mapper.xml中定义的每个sql的id相同；
2、Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql的parameterType类型相同；
3、Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同；
4、Mapper.xml文件中的namespace即是mapper接口的类路径。