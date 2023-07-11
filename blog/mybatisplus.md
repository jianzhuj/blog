有个坑  

我自定义了SqlSessionFactory，打上了@Bean("sqlSessionFactory")的注解

而mybatisplus的MybatisPlusAutoConfiguration类中有@ConditionalOnMissingBean这种注解

导致我在配置文件中写的配置被mybatisplus读取之后，没有配置

因为这些配置都是在sqlSessionFactory方法中配置的，但是我自定义了sqlSessionFactory，所以mybatisplus的就不会执行。

```
@Bean
@ConditionalOnMissingBean
public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
```