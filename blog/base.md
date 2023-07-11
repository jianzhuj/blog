方法重写是可以改变访问权限的,但是子类的重写的方法的访问权限一定要大于或等于父类

final 修饰的变量的引用对象不能改变,但变量的内部的值是可以修改的

```java
  final int a =11 ;
  a =12; //Cannot assign a value to final variable 'a'
  final List<Integer> list = Arrays.asList(1,2,3,4,5);
  list.add(6);
        
```

泛型的擦拭法

<? extend Number>  接受Number及其子类 上界通配符 可读不可写

< ? super Integer>     接收Integer及其父类  下界通配符 可写不可读  

```
 Collections.copy的源码
 public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        int srcSize = src.size();
        if (srcSize > dest.size())
            throw new IndexOutOfBoundsException("Source does not fit in dest");

        if (srcSize < COPY_THRESHOLD ||
            (src instanceof RandomAccess && dest instanceof RandomAccess)) {
            for (int i=0; i<srcSize; i++)
                dest.set(i, src.get(i));
        } else {
            ListIterator<? super T> di=dest.listIterator();
            ListIterator<? extends T> si=src.listIterator();
            for (int i=0; i<srcSize; i++) {
                di.next();
                di.set(si.next());
            }
        }
    }
```

```
Properties 本质是 hashtable 历史遗留原因

Properties properties = new Properties();
/开头表示读取classpath下的文件
properties.load(test1.class.getResourceAsStream("/**"));
```
