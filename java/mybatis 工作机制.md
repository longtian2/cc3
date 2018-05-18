# Mybatis工作机制 #

图片来源于网络：

![](https://github.com/longtian2/cc3/blob/master/images/mybatis-uml.png)

简单示例如下：

	 public class SelectDemo {  
	
		 public static void main(String[] args) throws Exception {  
		
		  /* 
		   * 1.加载mybatis的配置文件，初始化mybatis，创建出SqlSessionFactory，是创建SqlSession的工厂 
		   * 这里只是为了演示的需要，SqlSessionFactory临时创建出来，在实际的使用中，SqlSessionFactory只需要创建一次，当作单例* *来使用 
		   */  
		  InputStream inputStream = Resources.getResourceAsStream("mybatisConfig.xml");  
		  SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();  
		  SqlSessionFactory factory = builder.build(inputStream);  
		 
		  //2. 从SqlSession工厂 SqlSessionFactory中创建一个SqlSession，进行数据库操作  
		  SqlSession sqlSession = factory.openSession();  

		 
		  //3.使用SqlSession查询  
		  Map<String,Object> params = new HashMap<String,Object>();  
		  params.put("min_salary",10000);  
		 
		  //a.查询工资低于10000的员工  
		  List<Employee> result = sqlSession.selectList("com.louis.mybatis.dao.EmployeesMapper.selectByMinSalary",params);  
		 
		  //b.未传最低工资，查所有员工  
		  List<Employee> result1 = sqlSession.selectList("com.louis.mybatis.dao.EmployeesMapper.selectByMinSalary");  
		 
		  System.out.println("薪资低于10000的员工数："+result.size());  
		  //~output :   查询到的数据总数：5    

		  System.out.println("所有员工数: "+result1.size());  
		  //~output :  所有员工数: 8  
		 
		 }  
	}  



## SqlSession ##

SqlSession是Mybatis与数据库交互的最顶层设计，也就是我们常说的会话。SqlSession接口包含了数据库的增删改查功能，还提供了当前会话的Configuration和Connection信息。该接口的“真正”实现类是DefaultSqlSession；另外，SqlSessionTemplate和SqlSessionManager也实现了SqlSession接口，不过他们是通过SqlSessionFactory工厂生成DefaultSqlSession，然后通过DefaultSqlSession完成接口的实现。

## SqlSessionFactory ##

SqlSessionFactory就是生成SqlSession的工厂。SqlSessionFactory接口主要是提供根据不同条件来创建SqlSession。该接口的“真正”实现类是DefaultSqlSessionFactory；另外，SqlSessionManager不仅实现了SqlSession接口，还实现了SqlSessionFactory接口，在获得SqlSessionManager时，会通过SqlSessionFactoryBuilder类来构建DefaultSqlSessionFactory对象，然后再通过DefaultSqlSessionFactory完成接口的实现。

## Configuration ##

Configuration类是配置信息，是Mybatis与数据库交互贯彻始终的上下文。Configuration类包含了Mapper的注册、拦截器链、类型转换器注册，还包括方法（statementId）与SQL语句、缓存、缓存级别、结果集转换、参数集映射关系等等。SqlSessionFactoryBuilder类完成了DefaultSqlSessionFactory与Configuration的关联。

在了解了SqlSession是怎么来的，再来看看SqlSession是怎么用的。我们以DefaultSqlSession的selectList举例。

![](https://github.com/longtian2/cc3/blob/master/images/mybatis-sqlSession.png)

我们看到从Configuration中取出要执行的SQL查询语句和查询条件，需要交由Executor来执行。

![](https://github.com/longtian2/cc3/blob/master/images/mybatis.png)

## MappedStatement ##

MappedStatement类负责维护了具体某一条SQL语句的<select|update|delete|insert>的封装。

## Executor ##

Executor是Mybatis的执行器，负责完成SQL语句和入参的拼装，生成真正执行的SQL语句，以及查询缓存的维护。Executor接口的直接实现类是BaseExecutor，BaseExecutor是抽象类，负责完成接口的实现，并定义一些抽象方法，待子类来实现，采用了模板设计模式。

BatchExecutor、SimpleExecutor、ReuseExecutor都是BaseExecutor抽象类的子类，分别的职能是批处理执行器、简单执行器、重复执行器。重点说一下ReuseExecutor类，该类中维护了一个final修饰的Map<String, Statement>，用于缓存Statement，另外，通过doUpdate()、doQuery()等方法我们也发现，方法最后都没有调用closeStatement()方法，相反，BatchExecutor、SimpleExecutor中的doUpdate()、doQuery()等方法都调用了closeStatement()方法。

CachingExecutor类也实现了Executor接口，采用的是静态代理模式。负责的是二级缓存的维护。

![](https://github.com/longtian2/cc3/blob/master/images/mybatis-executor.png)

## StatementHandler ##

StatementHandler封装了JDBC Statement操作，负责对SQL语句的执行和对结果集的转换。StatementHandler接口跟Executor的实现差不多。StatementHandler接口的直接实现类是BaseStatementHandler，BaseStatementHandler是抽象类，负责完成接口的实现，并定义一些抽象方法，待子类来实现，采用了模板设计模式。

PreparedStatementHandler、CallableStatementHandler、SimpleStatementHandler都是StatementHandler抽象类的子类，分别的职能是预编译处理器、存储过程处理器、非预编译处理器器。

**引申一下**，${ } 变量的替换阶段是在动态 SQL 解析阶段，即是由Mybatis完成的；而 #{ }变量的替换是通过JDBC的预编译来实现的，即是由数据库完成的。因此，${ } 非数字变量需要使用‘’，虽然我们不加‘’也能运行，那是因为数据库帮我们完成了类型的转换，正因为这样，如果并发量多的话，数据库将压力山大。还有一点很重要，#{ } 变量方式能够很大程度防止sql注入，${ } 变量方式无法防止Sql注入。

重点说一下，RoutingStatementHandler类也实现了StatementHandler接口，采用的是适配器模式。通过StatementType生成不同的StatementHandler。

![](https://github.com/longtian2/cc3/blob/master/images/mybatis-statement.png)

## ResultSetHandler ##

ResultSetHandler负责将JDBC返回的ResultSet结果集对象转换成List类型的集合。

## ParameterHandler ##

ParameterHandler负责对用户传递的参数转换成JDBC Statement 所需要的参数。该接口由DefaultParameterHandler类实现，参数转换可以查看setParameters()方法。

## Interceptor ##

Interceptor是Mybatis提供的拦截器，通过实现该接口自定义拦截器，并通过Plugin.warp完成代理。比如分页拦截器就是可以实现的该接口来完成分页计算。

------------------------------------------------------------------------------------------------------------

## 缓存机制 ##

Mybatis将数据缓存设计成两级结构，分为一级缓存和二级缓存。

![](https://github.com/longtian2/cc3/blob/master/images/mybatis-cache-2.png)

## 一级缓存 ##

![](https://github.com/longtian2/cc3/blob/master/images/mybatis-cache-1.png)

一级缓存是SqlSession会话级别的缓存，位于表示一次数据库会话的SqlSession对象之中，又被称之为本地缓存。一级缓存是MyBatis内部实现的一个特性，用户不能配置，默认情况下自动支持的缓存，用户没有定制它的权利（不过这也不是绝对的，可以通过开发插件对它进行修改）；

为什么设计一级缓存？     

每当我们使用MyBatis开启一次和数据库的会话，MyBatis会创建出一个SqlSession对象表示一次数据库会话。在对数据库的一次会话中，我们有可能会反复地执行完全相同的查询语句，如果不采取一些措施的话，每一次查询都会查询一次数据库,而我们在极短的时间内做了完全相同的查询，那么它们的结果极有可能完全相同，由于查询一次数据库的代价很大，这有可能造成很大的资源浪费。

为了解决这一问题，减少资源的浪费，MyBatis会在表示会话的SqlSession对象中建立一个简单的缓存，将每次查询到的结果结果缓存起来，当下次查询的时候，如果判断先前有个完全一样的查询，会直接从缓存中直接将结果取出，返回给用户，不需要再进行一次数据库查询了。

一级缓存的实现是在BaseExecutor抽象类中通过持有PerpetualCache变量来实现的，PerpetualCache类中包含一个简单的HashMap<k,v>来缓存数据。

## 二级缓存 ##

![](https://github.com/longtian2/cc3/blob/master/images/mybatis-cache-3.png)

二级缓存是Application应用级别的缓存，它的是生命周期很长，跟Application的声明周期一样，也就是说它的作用范围是整个Application应用。

设计二级缓存的目的是为了更进一步的提高数据的查询效率，降低访问数据库的频率。

二级缓存的实现是通过CachingExecutor类实现的，由该类完成缓存的读写操作，如果缓存中不存在，则发起Executor读取数据库数据。

MyBatis还允许用户自定义Cache接口实现，用户是需要实现org.apache.ibatis.cache.Cache接口，然后将Cache实现类配置在<cache  type="">节点的type属性上即可。


参考文献：

   http://blog.csdn.net/luanlouis/

联系方式：

https://github.com/longtian2

**如有用请不吝打赏**

![](https://github.com/longtian2/cc3/blob/master/images/wechat_pay.png)