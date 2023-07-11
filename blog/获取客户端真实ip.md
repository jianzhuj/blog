在通过反向代理访问后端的时候，要想拿到客户端的真实ip

nginx为例,需要加上两项配置项,X-Real-IP和X-Forwarded-For

```
 location ^~ /app/ {               
	 proxy_set_header        X-Real-IP       $remote_addr;               
     proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;                 	  proxy_pass    ***
 }
```

需要注意的是X-Forwarded-For的会追加的,每经过一层nginx,就会有一个nginx地址记录。

而X-Real-IP会被覆盖,所有只有最外层的nginx需要加上这个配置,其他nginx不能加上这一项.

比如客户端通过a->b-c->后端 通过三台nginx转发后访问到后台,

那么a上需要配置

```
 location ^~ /app/ {               
	 proxy_set_header        X-Real-IP       $remote_addr;               
     proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;                 	  proxy_pass    ***
 }
```

b和c上需要配置

```
 location ^~ /app/ {               
	
     proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;                 	  proxy_pass    ***
 }
```

这样后台拿到的ip地址就会是客户端的地址,通过X-Forwarded-For也会记录客户端,nginxA,nginxB,nginxC 这样的ip链路



```
  public static String getIpAddr(HttpServletRequest request) {
       
    
        String ipAddress = request.getHeader("X-Real-IP");//先从nginx自定义配置获取
     
        if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("x-forwarded-for");
         
        }
        if(ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("Proxy-Client-IP");
          
        }
        if(ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("WL-Proxy-Client-IP");
            
        }
        if(ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getRemoteAddr();
            if(ipAddress.equals("127.0.0.1") || ipAddress.equals("0:0:0:0:0:0:0:1")){
                //根据网卡取本机配置的IP
                InetAddress inet=null;
                try {
                    inet = InetAddress.getLocalHost();
                } catch (UnknownHostException e) {
                    e.printStackTrace();
                }
                ipAddress= inet.getHostAddress();
            }
        }
        //对于通过多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割
        if(ipAddress!=null){ //"***.***.***.***".length() = 15
            if(ipAddress.indexOf(",")>0){
                ipAddress = ipAddress.substring(0,ipAddress.indexOf(","));
            }
        }
        return ipAddress;
    }
```

参考资料

https://www.cnblogs.com/luxiaojun/p/10451860.html

https://juejin.cn/post/6891465308083585037

https://blog.csdn.net/xiaoxiao_yingzi/article/details/92835704