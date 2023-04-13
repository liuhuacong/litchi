# 荔枝(Litchi)

![OSCS](https://www.oscs1024.com/platform/badge/yhzdys/litchi.svg?size=small)
![LICENSE](https://img.shields.io/github/license/yhzdys/litchi)
![Maven](https://img.shields.io/maven-metadata/v?metadataUrl=https%3A%2F%2Frepo1.maven.org%2Fmaven2%2Fcom%2Fyhzdys%2Flitchi%2Fmaven-metadata.xml)
![Java](https://img.shields.io/badge/java-1.8%2B-green)
![Code Size](https://img.shields.io/github/languages/code-size/yhzdys/litchi)

## 关于

一款基于Spring+Mybatis开发的轻量化***本地多数据源事务控制***插件，荔枝很好吃😁

## 快速开始

### 添加依赖

~~~xml
<dependency>
    <groupId>com.yhzdys</groupId>
    <artifactId>litchi</artifactId>
    <version>${litchi.version}</version>
</dependency>
~~~

### 多数据源配置

~~~ java
@Bean
public DataSource masterDataSource() {
    // ....
    return dataSource;
}

@Bean
public DataSource slave1DataSource() {
    // ....
    return dataSource;
}

@Bean
public DataSource slave2DataSource() {
    // ....
    return dataSource;
}

@Bean
@Primary
public DataSource multiDataSource(@Qualifier("master") DataSource masterDataSource,
                                    @Qualifier("slave1") DataSource slave1DataSource,
                                    @Qualifier("slave1") DataSource slave2DataSource) {
    Map<String, DataSource> map = new HashMap<>(4);
    map.put("master", masterDataSource);
    map.put("slave1", slave1DataSource);
    map.put("slave2", slave2DataSource);

    MultiDataSource dataSource = new MultiDataSource();
    // 多数据源
    dataSource.setDataSources(map);
    // 默认数据源
    dataSource.setDefaultDataSource("master");
    return dataSource;
}
~~~

### 多数据源事务控制

~~~ java
/**
 * 多数据源事务切面配置
 */
@Bean
public PointcutAdvisor multiTransactionAdvisor() {
    return new MultiTransactionAdvisor();
}
~~~

### 多数据源自动切换

_以下两种方案任选其一即可！_
_以下两种方案任选其一即可！_
_以下两种方案任选其一即可！_

#### 1. 使用Litchi动态代理（推荐）

~~~ java
import com.yhzdys.litchi.support.mybatis.MapperFactoryBean;

// 使用LitchiMapperFactoryBean替换Mybatis原生的MapperFactoryBean
@MapperScan(basePackages = "com.xxx.xxx.mapper", factoryBean = MapperFactoryBean.class)
~~~

#### 2. 使用Mybatis插件

~~~ java
@Bean
public Interceptor dataSourceInterceptor() {
    return new DataSourceInterceptor();
}
~~~

### Mapper接口定义切换的数据源

~~~java
@RoutingDataSource("slave1")
public interface UserMapper {
  // ...
}
~~~

### 事务控制

~~~java
@Service
public class PaperMoonService {

    @Resource
    private MasterMapper masterMapper;
    @Resource
    private Salve1Mapper salve1Mapper;
    @Resource
    private Salve2Mapper salve2Mapper;

    @MultiTransactional(rollbackFor = {BizException.class, RpcException.class}, noRollbackFor = {IgnoreException.class}, propagation = Propagation.REQUIRED)
    public void doService() {
        // ...
        masterMapper.insert();
        // ...
        salve1Mapper.update();
        // ...
        salve2Mapper.delete();
    }
}
~~~

#### 支持的事务传播方式

~~~java
public enum Propagation {

    /**
     * 支持当前事务，如果当前没有事务，就新建一个事务。
     */
    REQUIRED,

    /**
     * 支持当前事务，如果当前没有事务，就以非事务方式执行。
     */
    SUPPORTS,

    /**
     * 支持当前事务，如果当前没有事务，就抛出异常。
     */
    MANDATORY,

    /**
     * 新建事务，如果当前存在事务，把当前事务挂起。
     */
    REQUIRES_NEW,

    /**
     * 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
     */
    NOT_SUPPORTED,

    /**
     * 以非事务方式执行，如果当前存在事务，则抛出异常。
     */
    NEVER,
}
~~~
