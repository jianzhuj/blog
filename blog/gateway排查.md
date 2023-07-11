gateway排查技巧

1. 日志放开的trace级别

```
logging:
  level:
    org.springframework.cloud.gateway: trace
```

2. actuator端点暴露

   1. 添加actuator依赖

      ```
        <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-actuator</artifactId>
         </dependency>
      ```

   2. 配置actuator

      ```
      management.endpoints.web.exposure.include=gateway
      ```

   配置完之后可以使用actuator的api获取网关信息

   gateway排查技巧

   1. 日志放开的trace级别

   ```
   logging:
     level:
       org.springframework.cloud.gateway: trace
   ```

   2. actuator端点暴露

      1. 添加actuator依赖

         ```
           <dependency>
                     <groupId>org.springframework.boot</groupId>
                     <artifactId>spring-boot-starter-actuator</artifactId>
            </dependency>
         ```

      2. 配置actuator

         ```
         management.endpoints.web.exposure.include=gateway
         ```

      配置完之后可以使用actuator的api获取网关信息![1684982367457](https://janekko.oss-cn-hangzhou.aliyuncs.com/img1684982367457.jpg)

      ```
      http://localhost:9900/actuator/gateway/routes
      ```

   ​	

   参考资料
   https://cloud.tencent.com/developer/article/1506254  
   https://mp.weixin.qq.com/s?__biz=MzI4ODQ3NjE2OA==&mid=2247485549&idx=2&sn=b825639686e18f3073564f80acc9c7a6&chksm=ec3c950adb4b1c1cef9774933c85efd6d681217e12bb12cc2b1299f014ddb042fc8139a7af12&scene=21#wechat_redirect

   ```
   http://localhost:9900/actuator/gateway/routes
   ```

   

​	

参考资料
https://cloud.tencent.com/developer/article/1506254  
https://mp.weixin.qq.com/s?__biz=MzI4ODQ3NjE2OA==&mid=2247485549&idx=2&sn=b825639686e18f3073564f80acc9c7a6&chksm=ec3c950adb4b1c1cef9774933c85efd6d681217e12bb12cc2b1299f014ddb042fc8139a7af12&scene=21#wechat_redirect