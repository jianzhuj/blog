cookie存在客户端

```
Cookie cookie = new Cookie("name", "ekko");
response.addCookie(cookie);
   

响应头
HTTP/1.1 200
Set-Cookie: name=ekko
Content-Type: text/html;charset=UTF-8
Content-Length: 8
Date: Sat, 01 Oct 2022 08:08:02 GMT
Keep-Alive: timeout=60
Connection: keep-alive
   
```

浏览器接收到这些响应头后，会把它们作为Cookie文件存在客户端。当我们第二次请求**同一个**服务器，浏览器在发送HTTP请求时，会带上这些Cookie。



session存在服务端

