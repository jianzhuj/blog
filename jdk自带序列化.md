java 自带的序列化

```
ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("/Users/guanliyuan/user.txt"));
objectOutputStream.writeObject(user);
objectOutputStream.close();
```



```
ObjectInputStream input = new ObjectInputStream(new FileInputStream("object.data"));
MyClass object = (MyClass) input.readObject(); //etc.
input.close();

```

在java中是通过ObjectInputStream进行序列化操作的

在objectOutputStream.writeObject中 会去检测对象是否实现了Serializable接口,没有的话就直接抛异常

并且会去检查对象是否有writeObject方法，writeObject函数定义如下

```
void writeObject(java.io.ObjectOutputStream s)
```

有的话就使用自定义的,没有的话就使用默认的defaultWriteFields

```
  private void writeSerialData(Object obj, ObjectStreamClass desc)
        throws IOException
    {
        ObjectStreamClass.ClassDataSlot[] slots = desc.getClassDataLayout();
        for (int i = 0; i < slots.length; i++) {
            ObjectStreamClass slotDesc = slots[i].desc;
            // 判断obj是否有实现writeObject方法,有就反射调用,没有就走defaultWriteFields方法
            if (slotDesc.hasWriteObjectMethod()) {
                PutFieldImpl oldPut = curPut;
                curPut = null;
                SerialCallbackContext oldContext = curContext;

                if (extendedDebugInfo) {
                    debugInfoStack.push(
                        "custom writeObject data (class \"" +
                        slotDesc.getName() + "\")");
                }
                try {
                    curContext = new SerialCallbackContext(obj, slotDesc);
                    bout.setBlockDataMode(true);
                    slotDesc.invokeWriteObject(obj, this);
                    bout.setBlockDataMode(false);
                    bout.writeByte(TC_ENDBLOCKDATA);
                } finally {
                    curContext.setUsed();
                    curContext = oldContext;
                    if (extendedDebugInfo) {
                        debugInfoStack.pop();
                    }
                }

                curPut = oldPut;
            } else {
                defaultWriteFields(obj, slotDesc);
            }
        }
    }
```

readObject也是同理

例如 ArrayList就自己实现了writeObject,readObject

serialVersionUID的作用主要是检验版本一致性

比如定义了一个Class A   serialVersionUID为1,然后将A的一个实例对象a序列化到了磁盘上，然后A类的定义进行了一些修改,将serialVersionUID改成了2，这个时候再将反序列化为A对象时候,就会发现serialVersionUID不一致而报错。

如果我们在序列化中没有显示地声明serialVersionUID，则序列化运行时将会根据该类的各个方面计算该类默认的serialVersionUID值。但是，Java官方强烈建议所有要序列化的类都显示地声明serialVersionUID字段，因为如果高度依赖于JVM默认生成serialVersionUID，可能会导致其与编译器的实现细节耦合，这样可能会导致在反序列化的过程中发生意外的InvalidClassException异常。因此，为了保证跨不同Java编译器实现的serialVersionUID值的一致，实现Serializable接口的必须显示地声明serialVersionUID字段

jdk自带的序列化很少被使用