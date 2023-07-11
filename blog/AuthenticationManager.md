AuthenticationManager接受一个Authentication对象作为输入参数，并返回一个已经完成认证的Authentication对象。

Authentication对象代表了一个用户的身份认证信息，它包含了用户的认证凭据（比如用户名和密码）、用户的权限（比如角色和权限）以及其他相关信息。Authentication对象由认证过滤器负责创建和传递，最终由AuthenticationManager完成身份认证并返回已认证的Authentication对象。

AuthenticationManager接口有两个主要的实现类：ProviderManager和AuthenticationProvider。ProviderManager是AuthenticationManager的默认实现类，它是一个委托模式的实现，它会将身份认证的任务委托给一个或多个AuthenticationProvider实现类。

如下图所示  挨个调用AuthenticationProvider的实现类去调用authenticate

![image-20230426103151713](https://janekko.oss-cn-hangzhou.aliyuncs.com/imgimage-20230426103151713.png)

所以我们需要实现AuthenticationProvider接口,重写authenticate和supports方法

```
public interface AuthenticationProvider {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;

    boolean supports(Class<?> authentication);
}
```

