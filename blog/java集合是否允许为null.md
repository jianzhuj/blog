## List

### 	ArrayList

   底层采用数组存储 可以为null

### 	LinkedList

​	底层采用双向联表存储,也可以为null

###  vector

​	底层采用数组存储,可以存null

## 	Set

### 	hashSet

​	底层是hashmap存储,可以存null,但最多1个,多的会被覆盖

### 	treeSet

​	***底层使用treeMap,不能为null,因为需要实现comparator接口***

## 	Map

### 	hashMap

​	key和value都能为null

### 	treeMap

​	***key默认不能为null,因为treeMap实现了排序,必须要实现comparator接口,***

***但是如果构造treemap的时候自定义排序方法,其实key是可以存null的***

```
  integerIntegerHashMap.put(null,null);
        TreeMap<Integer, Integer> integerIntegerTreeMap = new TreeMap<>((a,b) ->{
            if (a == null){
                return 0 ;
            }
            if (b==null){
                return 0;
            }
            if (a<b){
                return -1;
            }else{
                return 1;
            }
        });
        integerIntegerTreeMap.put(null,1);
        integerIntegerTreeMap.put(null,2);
```

### 	hashTable

​	 key 和value 都不能为null，hashTable是一个古老的类,原因是作者希望所有的key都实现hashcode 和equals方法.而后续hashMap就允许null为key

总结：只要实现了自然排序的集合,key都不能为null, 比如treeSet treeMap,而hashTable不允许key和value都null。

在自定义排序方式的情况下是treeSet  treeMap是可以用null做key的 

