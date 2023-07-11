### springmvc 几种主要的接受参数的注解

1. @PathVariable

   @PathVariable只能接收url路径中的参数

   ```
    @PostMapping("/test/{age}/{name}")
     public String name ( @PathVariable("name") String name ,@PathVariable("age") String age ){
           log.info("{},{}",name,age);
           return name ;
       }
   ```

   

2. @RequestParam

   ​	RequestParam接受url路径之后携带的参数 类似 url**?a=1&b=2

   ```
   @RequestParam("a")String a,@RequestParam("b")String b
   ```

3. @RequestBody

3. @RequestPart

   ​		

	multipart/form-data
	application/x-www-form-urlencoded