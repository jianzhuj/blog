#### 日志traceId追踪可以使用MDC机制(Mapped Diagnostic Context)

   1. 第一步 改动日志输入格式 加上一个%X{traceId}

   2. 应用中添加过滤器或者统一的方法入口加上traceId

        比如在网关中加一个GlobalFilter

      ```
      public class WebTraceFilter extends OncePerRequestFilter {
          @Autowired
          private TraceProperties traceProperties;
      
          @Override
          protected boolean shouldNotFilter(HttpServletRequest request) {
              return !traceProperties.getEnable();
          }
      
          @Override
          protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                          FilterChain filterChain) throws IOException, ServletException {
              try {
                  String traceId = request.getHeader(MdcTraceUtils.TRACE_ID_HEADER);
                  if (StringUtils.isEmpty(traceId)) {
                      MdcTraceUtils.addTraceId();
                  } else {
                      MdcTraceUtils.putTraceId(traceId);
                  }
                  filterChain.doFilter(request, response);
              } finally {
                  MdcTraceUtils.removeTraceId();
              }
          }
      }
      
      ```

	3. 跨服务传递traceId

        	1. feignClient

    ```
    @Configuration
    public class FeignInterceptor implements RequestInterceptor {
    
        private static final String TRACE_ID = "traceId";
    
        @Override
        public void apply(RequestTemplate requestTemplate) {
    
            requestTemplate.header(TRACE_ID, MDC.get(TRACE_ID));
    
        }
    }
    ```
    
    2. dubbo  拓展filter,并指定filter 需要在resources/META-INF/dubbo/org.apache.dubbo.rpc.Filter中指定实现类的全限定名
    
       ```
       @Activate(group = {"provider", "consumer"})
       public class TraceIdFilter implements Filter {
       
           @Override
           public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {     RpcContext rpcContext = RpcContext.getContext();
                 String traceId;
               if (rpcContext.isConsumerSide()) {
       
                   traceId = MDC.get("traceId");
       
                   if (traceId == null) {
                       traceId = UUID.randomUUID().toString();
                   }
       
                   rpcContext.setAttachment("traceId", traceId);
       
               }
       
               if (rpcContext.isProviderSide()) {
                   traceId = rpcContext.getAttachment("traceId");
                   MDC.put("traceId", traceId);
               }
       
               return invoker.invoke(invocation);
           }
       }
       ```
    
    4. 跨线程传递traceId
    
    通用思路就是在调用线程b之前先从线程a中取出traceId,然后放入到线程b中
    
    父子线程情况:
    
    可以将log4j2.isThreadContextMapInheritable设置为true，可以将Map配置为使InheritableThreadLocal，这样可以从父线程传递给子线程，但父子线程的情况比较少，大部分都是通过线程池使用多线程。
    
    线程池情况：使用装饰器
    
    ```
    public class TaskTraceIdDecorator implements TaskDecorator {
        @Override
        public Runnable decorate(Runnable runnable) {
            Map<String, String> contextMap = MDC.getCopyOfContextMap();
            return () -> {
                try {
                    if (contextMap != null) {
                        MDC.setContextMap(contextMap);
                    }
                    runnable.run();
                } finally {
                    MDC.clear();
                }
            };
        }
    }


​    
​    // 配置线程池的时候 
​      ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
​            executor.setCorePoolSize(coreSize);
​            executor.setMaxPoolSize(maxSize);
​            executor.setQueueCapacity(queueCapacity);
​            executor.setKeepAliveSeconds(keepAlive);
​            executor.setThreadNamePrefix(threadNamePrefix);
​            executor.setRejectedExecutionHandler(getHandler(rejectPolicy));
​            executor.setWaitForTasksToCompleteOnShutdown(true);
​            executor.setAwaitTerminationSeconds(60);
​            **executor.setTaskDecorator(new TaskTraceIdDecorator());**
​            return executor;
​    ```


  参考资料

https://blog.csdn.net/xingbaozhen1210/article/details/89230570