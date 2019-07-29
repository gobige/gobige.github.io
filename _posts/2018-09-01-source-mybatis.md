---
layout: post
title: '源码解析-mybatis'
subtitle: 'mybatis的源码解析'
date: 2018-09-01
categories: 源码
author: yates
cover: ''
tags: source
---

**mybatis调用实例**
```java
public static void main(String[] args) {
        String resource="mybatis-config.xml";
        InputStream inputStream = null;
        try {
            inputStream = Resources.getResourceAsStream(resource);  1
        } catch (IOException e) {
            e.printStackTrace();
        }
        SqlSessionFactory sqlSessionFactory=null;
        sqlSessionFactory=new SqlSessionFactoryBuilder().build(inputStream); 2
        SqlSession sqlSession=null;
        try {
            sqlSession=sqlSessionFactory.openSession(); 3
            SysAppVersionInfoDao sysAppVersionInfoDao = sqlSession.getMapper(SysAppVersionInfoDao.class); 4
            SysAppVersionInfo role = sysAppVersionInfoDao.queryByIdBySelectProvider(16);  5
            System.out.println(role.getId()+":"+role.getRemark()+":"+role.getAppVersion());
            sqlSession.commit(); 6 

        } catch (Exception e) {
            sqlSession.rollback(); 6
            e.printStackTrace();
        }finally {
            sqlSession.close();  7
        }
    }
```

mybatis调用大体可分为上面7步：

1. 加载解析mybatis的xml配置文件
2. 根据得到的配置文件流建立sqlsesiionfactory
3. 通过sqlsessionfactory为此次sql执行创建sqlsession
4. 通过反射为sqlsession加载对应的mapper映射
5. 通过sqlsession执行对应的sql语句
6. 提交/回滚sql执行语句
7. 关闭sqlsession，释放资源


**详解**

第一步：
此步的解析请移步
http://muyibeyond.cn/2017/11/11/stream-file-get.html

第二步：
通过得到的流装载封装到XPathParser对象中，使用XMLConfigBuilder对XPathParser解析（XPathParser把流数据转换成XNODE，从configuration根元素开始，通过对一个个节点的eval得到各个node封装到XNODE对象，属性，变量，XPATH）

XNODE对象 声明的变量
```java
private final Node node;
private final String name;
private final String body;
private final Properties attributes;
private final Properties variables;
private final XPathParser xpathParser;
```

XPathParser对象 声明的变量
```java
private final Document document;
private boolean validation;
private EntityResolver entityResolver;
private Properties variables;
private XPath xpath;
```

xmlconfigbuilderr对象 声明的变量
```java
private boolean parsed;
private final XPathParser parser;
private String environment;
private final ReflectorFactory localReflectorFactory;
```

xmlconfigbuilder解析
```java
public Configuration parse() {
    if (this.parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    } else {
        this.parsed = true;
        this.parseConfiguration(this.parser.evalNode("/configuration"));
        return this.configuration;
    }
}
```

xmlconfigbuilder解析 XNode
```java
private void parseConfiguration(XNode root) {
    try {
		// 取出properties元素封装进configuration和parser对象
        this.propertiesElement(root.evalNode("properties"));
		// 获取，验证settings元素
        Properties settings = this.settingsAsProperties(root.evalNode("settings"));
		// 加载虚拟文件系统配置
        this.loadCustomVfs(settings);
		// 声明类型别名
        this.typeAliasesElement(root.evalNode("typeAliases"));
		// 加载配置插件元素
        this.pluginElement(root.evalNode("plugins"));
		// 自定义对象工厂（MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成。 默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。 如果想覆盖对象工厂的默认行为，则可以通过创建自己的对象工厂来实现）
        this.objectFactoryElement(root.evalNode("objectFactory"));
		// 加载自定义objectWrapperFactory对象
        this.objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
		//创建配置自定义反射工厂
        this.reflectorFactoryElement(root.evalNode("reflectorFactory"));
		// 配置属性元素配置
        this.settingsElement(settings);
		// 配置，mybatis多环境运行环境 
        this.environmentsElement(root.evalNode("environments"));
		// 配置多数据库支持（mybatis加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略）,配置多数据库，mybatis会和当前environment匹配合适的databaseId选择合适的数据库
        this.databaseIdProviderElement(root.evalNode("databaseIdProvider"));
		// 类型处理器将获取的值以合适的方式转换成 Java 类型当我们没有配置指定TypeHandler时，Mybatis会根据参数或者返回结果的不同，默认为我们选择合适的TypeHandler处理(实际也是注册typeAlias来实现)
        this.typeHandlerElement(root.evalNode("typeHandlers"));
		// 加载mapper到configuration中
        this.mapperElement(root.evalNode("mappers"));
    } catch (Exception var3) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + var3, var3);
    }
}
```

将得到的configuration对象封装到sqlsessionFactory中
```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    SqlSessionFactory var5;
    try {
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
        var5 = this.build(parser.parse());
    } catch (Exception var14) {
        throw ExceptionFactory.wrapException("Error building SqlSession.", var14);
    } finally {
        ErrorContext.instance().reset();

        try {
            inputStream.close();
        } catch (IOException var13) {
        }

    }

    return var5;
}
```

第三步： 获取configuration对象里面的environment对象，使用事务工厂transactionfactory，生成一条新的事务，创建执行器excutor，创建一条默认的session
```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;

    DefaultSqlSession var8;
    try {
        Environment environment = this.configuration.getEnvironment();
        TransactionFactory transactionFactory = this.getTransactionFactoryFromEnvironment(environment);
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
        Executor executor = this.configuration.newExecutor(tx, execType);
        var8 = new DefaultSqlSession(this.configuration, executor, autoCommit);
    } catch (Exception var12) {
        this.closeTransaction(tx);
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + var12, var12);
    } finally {
        ErrorContext.instance().reset();
    }

    return var8;
}
```

defaultsqlsession
```java
public class DefaultSqlSession implements SqlSession {
    private final Configuration configuration;
    private final Executor executor;
    private final boolean autoCommit;
    private boolean dirty;
    private List<Cursor<?>> cursorList;
}
```

sqlsession提供的接口
```java
<T> T selectOne(String var1);
<E> List<E> selectList(String var1);
<K, V> Map<K, V> selectMap(String var1, String var2);
<T> Cursor<T> selectCursor(String var1);
void select(String var1, Object var2, ResultHandler var3);
int insert(String var1);
int update(String var1);
int delete(String var1);
void commit();
void rollback(boolean var1);
List<BatchResult> flushStatements();
void close();
void clearCache();
Configuration getConfiguration();
<T> T getMapper(Class<T> var1);
Connection getConnection();
```

第四步：根据sqlsession获取configuration里面的maaper配置，我们可以看到是从mapperr注册器中获取
```java
// sqlsession.getMapper方法
public <T> T getMapper(Class<T> type) {
    return this.getConfiguration().getMapper(type, this);
}
// configuration.getMapper方法
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return this.mapperRegistry.getMapper(type, sqlSession);
}
// mapperRegistry.getMapper方法
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory)this.knownMappers.get(type);
    if (mapperProxyFactory == null) {
        throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    } else {
        try {
            return mapperProxyFactory.newInstance(sqlSession);
        } catch (Exception var5) {
            throw new BindingException("Error getting mapper instance. Cause: " + var5, var5);
        }
    }
}
```

通过mapper代理工厂在当前sqllsession中生成新的mapper实例,生成代理对象，代理mapper
```java
public T newInstance(SqlSession sqlSession) {
    MapperProxy<T> mapperProxy = new MapperProxy(sqlSession, this.mapperInterface, this.methodCache);
    return this.newInstance(mapperProxy);
}

protected T newInstance(MapperProxy<T> mapperProxy) {
    return Proxy.newProxyInstance(this.mapperInterface.getClassLoader(), new Class[]{this.mapperInterface}, mapperProxy);
}
```

第五步：根据mapper对象调用对应方法，映射到对应的sql，使用相应的excutor执行相应statement，形成sql缓存

```java
protected PerpetualCache localCache;
protected PerpetualCache localOutputParameterCache;
```

PerpetualCache类
```java
public class PerpetualCache implements Cache {
	private final String id;
	// 执行语句集合
	private Map<Object, Object> cache = new HashMap();
	
	
	// 
    public void clear() {
        this.cache.clear();
    }
	
	public ReadWriteLock getReadWriteLock() {
        return null;
    }
	
	// 
}
	
```
**提交/回滚/关闭session都是有excutor执行器去操作**
第六步：提交/回滚
调用excutor提交/回滚事务，清缓存，刷statement
```java
public void commit(boolean required) throws SQLException {
	if (this.closed) {
		throw new ExecutorException("Cannot commit, transaction is already closed");
	} else {
		this.clearLocalCache();
		this.flushStatements();
		if (required) {
			this.transaction.commit();
		}

	}
}	
	
public void rollback(boolean required) throws SQLException {
	if (!this.closed) {
		try {
			this.clearLocalCache();
			this.flushStatements(true);
		} finally {
			if (required) {
				this.transaction.rollback();
			}

		}
	}
}
```

第七步：关闭session

```java
public void close(boolean forceRollback) {
	try {
		try {
			this.rollback(forceRollback);
		} finally {
			if (this.transaction != null) {
				this.transaction.close();
			}

		}
	} catch (SQLException var11) {
		log.warn("Unexpected exception on closing transaction.  Cause: " + var11);
	} finally {
		this.transaction = null;
		this.deferredLoads = null;
		this.localCache = null;
		this.localOutputParameterCache = null;
		this.closed = true;
	}

}
```

**#{}和${}的传参区别**

#{}是预编译处理，会将其替换为?，然后调用prepareStatement的set方法赋值，值会加上单引号
${}是字符串替换，会将其替换为变量的值，传入数据，不会加上单引号
