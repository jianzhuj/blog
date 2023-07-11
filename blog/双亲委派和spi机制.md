#### 双亲委派机制

![](https://janekko.oss-cn-hangzhou.aliyuncs.com/%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE.png)

​	类加载的时候,首先会向上传递,一直到最顶层的类加载器，如果底层类加载器不能加载,就会向下传递给子加载器加载。

​	**这样做的好处是安全性的保障，可以确保java核心类库不被篡改**，比如自己定义的Object类或者网络传输加载的类。

##### 	mysql驱动的加载

​	比如加载Mysql驱动，一般使用 Class.forName("com.mysql.jdbc.Driver"); 

​	程序在运行过程中，遇到未加载的类,会使用调用者的class对象的classLoader来加载当前的未知类。也就是说如果在a对象的方法调用中遇到了未加载的类B，这个时候就会调用a的Class的classLoader来加载B。

```
Reflection.getCallerClass()
```

这个时候就首先会尝试使用AppClassLoader来加载，根据双亲委派模型，ExtensionClassLoader和BootstrapClassLoader都无法加载,最后还是会被AppClassLoader加载。

而com.mysql.jdbc.Driver中有一段静态代码块,会向DriverManager中注册mysql的驱动。

```
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    public Driver() throws SQLException {
    }

    static {
        try {
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}
```

class.forName加载的方法原理大概如下

```java
public static Class<?> forName(String name, boolean initialize,
                                   ClassLoader loader)
        throws ClassNotFoundException
    // loader参数可选，用来指定用哪个ClassLoader加载
    {
        Class<?> caller = null;
 
           	// 获取当前调用者的class
    	caller = Reflection.getCallerClass();
    
            // 拿到当前调用者的ClassLoader
	    ClassLoader ccl = ClassLoader.getClassLoader(caller);
		    // 调用native方法去使用ClassLoader来加载
        return forName0(name, initialize, loader, caller);
    }

```

​	其中Class.forName()还可以传入指定的ClassLoad来加载。

#### SPI机制

JDBC4.0之后就不需要Class.forName来加载驱动了，只需要把驱动的jar包放到工程的类路径中就会被自动识别和加载。具体的方式就是去jar的META-INF/services/java.sql.Driver文件中寻找，在该文件中定义了Driver接口的mysql实现。

![](https://janekko.oss-cn-hangzhou.aliyuncs.com/image-20220927112708331-16642492298122.png)

这种就是spi机制(Service Provider Interface),JDK内置的一种服务提供发现机制,由JDK来定义接口,各大厂商做出自己的实现,并通过查找META-INF/services/接口全限定名文件去查找驱动名称来加载实现类。

这样在需要Mysql驱动的时候,只需要引入相关jar包,然后去使用，不需要Class.forName去加载驱动.

但这种机制实际破坏了双亲委派模型,因为电泳DriverManager.getConnection的时候,首先会加载DriverManager类,这个类是位于ra.jar中,有BootstrapClassLoad来加载，而DriverManager的静态代码块中会去尝试查找和加载各个数据库厂商的驱动。

```
 static {

        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }
    
     private static void loadInitialDrivers() {
        String drivers;
        try {
            drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
                public String run() {
                    return System.getProperty("jdbc.drivers");
                }
            });
        } catch (Exception ex) {
            drivers = null;
        }
        

        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {

                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();
                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                // Do nothing
                }
                return null;
            }
        });

        println("DriverManager.initialize: jdbc.drivers = " + drivers);

        if (drivers == null || drivers.equals("")) {
            return;
        }
        String[] driversList = drivers.split(":");
        println("number of Drivers:" + driversList.length);
        for (String aDriver : driversList) {
            try {
                println("DriverManager.Initialize: loading " + aDriver);
                // ClassLoader.getSystemClassLoader() 拿到的是AppClassLoad
                Class.forName(aDriver, true,
                        ClassLoader.getSystemClassLoader());
            } catch (Exception ex) {
                println("DriverManager.Initialize: load failed: " + ex);
            }
        }
    }
```

 这就产生了一个问题,DriverManager是BootStrapClassLoad加载器加载的,而各个数据库厂商的驱动并不在rt.jar中,BootStrapClassLoad无法加载这些驱动。**这个时候,双亲委派模型就无法实现spi机制,需要一种能够在父类加载器加载的类中去调用子类加载器去加载类**

JDK提供了两种解决方案

1.  线程上下文加载器

   ```
     public static <S> ServiceLoader<S> load(Class<S> service) {
           ClassLoader cl = Thread.currentThread().getContextClassLoader();
           return ServiceLoader.load(service, cl);
       }
   ```

    在sun.misc.Launcher 初始化的时候，会获取AppClassLoader，然后将其设置为上下文类加载器，所以线程上下文类加载器默认情况下就是系统加载器

   ```
   public Launcher() {
       ...
       try {
           this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
       } catch (IOException var9) {
           throw new InternalError("Could not create application class loader", var9);
       }
       Thread.currentThread().setContextClassLoader(this.loader);
       ...
   }
   ```

 2. systemClassLoader  

    ```
     private ServiceLoader(Class<S> svc, ClassLoader cl) {
            service = Objects.requireNonNull(svc, "Service interface cannot be null");
            loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
            acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
            reload();
        }
    ```

    systemClassLoader  是在initSystemClassLoader方法中写入的，获取的是sun.misc.Launcher的加载器，即AppClassLoader

    ```
    private static synchronized void initSystemClassLoader() {
        if (!sclSet) {
            if (scl != null)
                throw new IllegalStateException("recursive invocation");
            sun.misc.Launcher l = sun.misc.Launcher.getLauncher();
            if (l != null) {
                Throwable oops = null;
                scl = l.getClassLoader();
                try {
                    scl = AccessController.doPrivileged(
                        new SystemClassLoaderAction(scl));
                } catch (PrivilegedActionException pae) {
                    oops = pae.getCause();
                    if (oops instanceof InvocationTargetException) {
                        oops = oops.getCause();
                    }
                }
                if (oops != null) {
                    if (oops instanceof Error) {
                        throw (Error) oops;
                    } else {
                        // wrap the exception
                        throw new Error(oops);
                    }
                }
            }
            sclSet = true;
        }
    }
    ```

#### 	参考链接

	1. https://www.jianshu.com/p/09f73af48a98
	1. https://www.cnblogs.com/lyc88/articles/11431383.html
	1. https://www.jianshu.com/p/9cf306550b0a
	1. https://zhuanlan.zhihu.com/p/51374915

 