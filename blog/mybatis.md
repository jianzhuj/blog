### JDBC中的一些概念

Driver  DriverManager   DataSource   connection  statement

statement来执行sql语句

```
Statement stmt = conn.createStatement(); ResultSet rs = stmt.executeQuery(sql);
```



### mybatis 使用流程

1. 读取配置文件

2. SqlSessionFactoryBuilder利用配置文件创建SqlSessionFactory

   ```
   String resource = "chapter1/mybatis-config.xml"
   InputStream inputStream = Resources.getResourceAsStream(resource); 
   sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream); inputStream.close();
   ```

   

3. 利用SqlSessionFactory获取session

   ```
   SqlSession session = sqlSessionFactory.openSession();
   ```

   

4. 利用session来获取mapper，执行SQL， getMapper会返回一个代理类，用这个实现了ArticleDao接口的代理类来执行sql

   ```
   ArticleDao articleDao = session.getMapper(ArticleDao.class); List
   
   articles = articleDao. findByAuthorAndCreateTime("coolblog.xyz", "2018-06-10")
   ```

   

#### mybatis 配置文件解析

SqlSessionFactoryBuilder().build 是入口  

解析过程主要在XMLConfigBuilder.parseConfiguration

```
private void parseConfiguration(XNode root) {
  try {
    // issue #117 read properties first
    propertiesElement(root.evalNode("properties"));
    Properties settings = settingsAsProperties(root.evalNode("settings"));
    loadCustomVfs(settings);
    loadCustomLogImpl(settings);
    typeAliasesElement(root.evalNode("typeAliases"));
    pluginElement(root.evalNode("plugins"));
    objectFactoryElement(root.evalNode("objectFactory"));
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
    reflectorFactoryElement(root.evalNode("reflectorFactory"));
    settingsElement(settings);
    // read it after objectFactory and objectWrapperFactory issue #631
    environmentsElement(root.evalNode("environments"));
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));
    typeHandlerElement(root.evalNode("typeHandlers"));
    mapperElement(root.evalNode("mappers"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
  }
}
```

```
SqlSession sqlSession = sqlSessionFactory.openSession();
用的是DefaultSqlSessionFactory的实例对象来创建session



private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
      final Environment environment = configuration.getEnvironment();
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
      final Executor executor = configuration.newExecutor(tx, execType);
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      closeTransaction(tx); // may have fetched a connection so lets call close()
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
会返回一个DefaultSqlSession对象

public class DefaultSqlSession implements SqlSession {

  private final Configuration configuration;
  private final Executor executor;

  private final boolean autoCommit;
  private boolean dirty;
  private List<Cursor<?>> cursorList;
  }
  
  
  UserMapper mapper = sqlSession->.getMapper(UserMapper.class);
  
  sqlSession->configuration->mapperRegistry
  public class MapperRegistry {

  private final Configuration config;
  private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<>();

  public MapperRegistry(Configuration config) {
    this.config = config;
  }
  	
    public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
  }



```







mybatis



```
@MapKey("id")
Map<String ,User> selectAll(RowBounds rowBounds,ResultHandler resultHandler);
```

  @mapKey("id")

 如果一个接口定义的   

```
 List<User> selectUserList();
```

可以使用mapKey注解 将list转成map  ,id就会是Map中的key

```
  @MapKey("id")
  Map<String ,User> selectAll();
```



RowBounds   mybatis 提供的分页实现，用于兼容所有数据库的分页查询

*** 但是这个分页是逻辑分页***  也就是mysql是查出来所有的数据到内存再截取指定段的数据 

```



RowBounds rowBounds = new RowBounds(1,5);
List<User> selectUserList(rowBounds);
```



```

  public static final int NO_ROW_OFFSET = 0;
  public static final int NO_ROW_LIMIT = Integer.MAX_VALUE;
  public static final RowBounds DEFAULT = new RowBounds();

  private final int offset;
  private final int limit;

  public RowBounds() {
    this.offset = NO_ROW_OFFSET;
    this.limit = NO_ROW_LIMIT;
  }

  public RowBounds(int offset, int limit) {
    this.offset = offset;
    this.limit = limit;
  }

```

ResultHandler    mybatis 对查询结构的处理接口   

1. 使用这个接口 必须定义接口的返回值为void
2. 接口中定义的泛型要和mapper接口的resultType对应
3. 如果返回值是多条数据，那么会导致每一条数据都会调用这个接口一遍

```
public interface UserMapper {

    void  selectAll(RowBounds rowBounds , ResultHandler resultHandler );
}
```

```
public class MyResultHandle  implements  ResultHandler<User> {
    @Override
    public void handleResult(ResultContext<? extends User> resultContext) {
        int resultCount = resultContext.getResultCount();
        System.out.println(resultCount);
        User resultObject = resultContext.getResultObject();
        System.out.println(resultCount);
    }
}


```

