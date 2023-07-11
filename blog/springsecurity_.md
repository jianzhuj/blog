1. 三种设置用户名密码的方法  优先级  重写接口>配置类>配置文件

   1. 配置文件设置，写死用户名密码的形式，一般不用

      ```
      spring:
        security:
          user:
            name: ekko
            password: ekko
      ```

      

      2.配置类中配置  继承WebSecurityConfigurerAdapter并重写config方法，config中可指定加密方法

      ​	  PasswordEncoder接口实现密码加密，需要在spring中添加一个PasswordEncoder接口的实现类

      ```
      @Configuration
      public class SecurityConfig extends WebSecurityConfigurerAdapter {
      
          @Override
          protected void configure(AuthenticationManagerBuilder auth) throws Exception {
              BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
              String ekko = passwordEncoder.encode("ekko");
              auth.inMemoryAuthentication().withUser("ekko1").password(ekko).roles("admin");
          }
      
          @Bean
          PasswordEncoder  password(){
              return new BCryptPasswordEncoder();
          }
      }
      
      ```

      3.实现UserDetailsService接口 并重写configure方法

      ```
      	@Autowired
          private UserDetailsService userDetailsService ;
          
          @Override
          protected void configure(AuthenticationManagerBuilder auth) throws Exception {
              auth.userDetailsService(userDetailsService).passwordEncoder(password());
          }
      
          @Bean
          PasswordEncoder  password(){
              return new BCryptPasswordEncoder();
          }
          
          @Service
      public class MyUserDeatilsServiceImple  implements UserDetailsService {
          @Override
          public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
              List<GrantedAuthority> authorities = AuthorityUtils.commaSeparatedStringToAuthorityList("admin");
              return new User("ekko2","ekko",authorities);
          }
      }
      
          
      ```

2. 实现UserDetailsService接口,这个接口就是根据用户名查找用户信息

3. 架构脉络

   ```
   FilterChainProxy 维护一个List<SecurityFilterChain>filterChains
   每个请求都是依次遍历filterChains,调用SecurityFilterChain接口的match方法来匹配请求,如果为true,就调用SecurityFilterChain接口的getFilters()拿到这个SecurityFilterChain中注入的一系列过滤器来挨个执行。
   
   这里存在一个问题，就是FilterChainProxy维护了多个SecurityFilterChain,但只会执行第一个match的true的SecurityFilterChain.
   ```

    

4. FilterChainProxy 

   ```
   FilterChainProxy:
   
   private List<SecurityFilterChain> filterChains;
   
   
   
   
   ```

   

5. springSecurityFilterChain

```
public interface SecurityFilterChain {

   boolean matches(HttpServletRequest request);
	
   List<Filter> getFilters();

}
```





​	