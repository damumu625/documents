# 细说单例
------
###### 什么是单例模式
>[单例模式](http://zh.wikipedia.org/wiki/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F)，也叫单子模式，是一种常用的软件设计模式。在应用这个模式时，单例对象的类必须保证只有一个实例存在。

中心原则就是：**单例对象的类必须保证只有一个实例存在**

在java中主要有两种构建方式
* 懒汉方式。指全局的单例实例在第一次被使用时构建。
* 饿汉方式。指全局的单例实例在类装载时构建。

简单的说就是一个需要*延迟初始化*，一个则不需要。

比较简单的构建方式有：

```java
    public class Singleton {
        private final static Singleton INSTANCE = new Singleton();

        // Private constructor suppresses   
        private Singleton() {
        }

        // default public constructor
        public static Singleton getInstance() {
            return INSTANCE;
        }
    }
```

这种方式实现简单，实例在类装载时构建，如果想要实现一种实例在第一次被使用时构建应该怎么做？

有一种叫做*双重检查锁(double-checked locking)*

```java
    public class Singleton {
        private static volatile Singleton INSTANCE = null;

        // Private constructor suppresses 
        // default public constructor
        private Singleton() {
        }

        //thread safe and performance  promote 
        public static Singleton getInstance() {
            if (INSTANCE == null) {
                synchronized (Singleton.class) {
                    //when more than two threads run into the first null check same time, 
                    //to avoid instanced more than one time, it needs to be checked again.
                    if (INSTANCE == null) {
                        INSTANCE = new Singleton();
                    }
                }
            }
            return INSTANCE;
        }
    }
```

此种方法只能用在JDK5及以后版本(注意 INSTANCE 被声明为 volatile)，之前的版本使用“双重检查锁”会发生非预期行为.

推荐阅读:

1.  Effective Java 第71条 慎用延迟初始化
2.  Core Java 第一卷 14.5.8 Volatile 域
3.  [JSL 17.4](http://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4)
4.  [Java 理论与实践: 正确使用 Volatile 变量](http://www.ibm.com/developerworks/cn/java/j-jtp06197.html)
5.  [双重检查锁定与延迟初始化](http://ifeve.com/double-checked-locking-with-delay-initialization/)

在第一条推荐阅读里提到了另一种实现单例的方式*lazy initialization holder class idiom*

```java
    public class Singleton {

        // Private constructor suppresses   
        private Singleton() {
        }

        private static class LazyHolder {
            static final Singleton INSTANCE = new Singleton();
        }

        public static Singleton getInstance() {
            return LazyHolder.INSTANCE;
        }
    }
```

JVM在类的初始化阶段（即在Class被加载后，且被线程使用之前），会执行类的初始化。在执行类的初始化期间，JVM会去获取一个锁。这个锁可以同步多个线程对同一个类的初始化。相比其他实现方案（如double-checked locking等），该技术方案的实现代码较为简洁，并且在所有版本的编译器中都是可行的。

关于*static final Singleton INSTANCE*域的访问权限为什么时包级私有可以阅读:[Initialization On Demand Holder idiom的实现探讨](http://ifeve.com/initialization-on-demand-holder-idiom/)

--- ---

最后推荐实现最为简洁的一种方式: **使用枚举**

代码极其简洁, 使用极其简单:

```java
    public enum Singleton {
        INSTANCE;
    }
```

回顾一下前面集中单例的实现方式, 都只考虑了常规获取类对象的手段, 然而还可以通过序列化和反射机制获取对象.上面两种方式如果实现了序列化接口*Serializable*就必须重写*readResolve()*方法

```java
    private Object readResolve(){
        return INSTANCE;
    }
```

即使重写了 *readResolve()* 方法也会涉及某系域需要关键字 *transient* 的修饰, 具体讨论不再展开, 总之涉及序列化挺蛋疼. 

关于防止反射暂时没有深入了解, 据了解: 因为反射的某些地方绕过了java机制的限制，private只在编译时进行权限的限制，但是在运行时是不存在这种权限的限制的, 此处仅供参考.

但是使用enum实现的单例自带防序列化与防反射功能, 详细参照枚举类反编译后代码(供参考)

```java
    public abstract class Singleton extends Enum
    {

        private Singleton(String s, int i)
        {
            super(s, i);
        }

        public static Singleton[] values()
        {
            Singleton asingleton[];
            int i;
            Singleton asingleton1[];
            System.arraycopy(asingleton = ENUM$VALUES, 0, asingleton1 = new Singleton[i = asingleton.length], 0, i);
            return asingleton1;
        }

        public static Singleton valueOf(String s)
        {
            return (Singleton)Enum.valueOf(singleton/Singleton, s);
        }

        Singleton(String s, int i, Singleton singleton)
        {
            this(s, i);
        }

        public static final Singleton INSTANCE;
        private static final Singleton ENUM$VALUES[];

        static 
        {
            INSTANCE = new Singleton("INSTANCE", 0) ;
            ENUM$VALUES = (new Singleton[] {
                INSTANCE
            });
        }
    }
```

我们实现的枚举都是继承了 *java.lang.Enum* 可以看出来单例的实现也是通过关键字 *static* 修饰的静态初始化块来实现.

那么为什么enum可以防御反射呢...很简单, 因为它是一个抽象类 *public abstract class Singleton extends Enum* 即使是反射机制也不能实例化了.

有为什么能防御序列化呢...这个要看java源码中对于对象序列化的处理.

```java
    java.io.ObjectOutputStream

    private void writeObject0(Object obj, boolean unshared){
        ...
        // remaining cases
        if (obj instanceof String) {
            writeString((String) obj, unshared);
        } else if (cl.isArray()) {
            writeArray(obj, desc, unshared);
        } else if (obj instanceof Enum) {
            writeEnum((Enum) obj, desc, unshared);
        } else if (obj instanceof Serializable) {
            writeOrdinaryObject(obj, desc, unshared);
        } else {
            if (extendedDebugInfo) {
                throw new NotSerializableException(cl.getName() + "\n" + debugInfoStack.toString());
            } else {
                throw new NotSerializableException(cl.getName());
            }
        }
    }

    /**
    * Writes given enum constant to stream.
    */
    private void writeEnum(Enum en, ObjectStreamClass desc, boolean unshared) throws IOException {
        bout.writeByte(TC_ENUM);
        ObjectStreamClass sdesc = desc.getSuperDesc();
        writeClassDesc((sdesc.forClass() == Enum.class) ? desc : sdesc, false);
        handles.assign(unshared ? null : en);
        writeString(en.name(), false);
    }
```

可以看出enum在被序列化时时经过特殊处理的, 被序列化的仅仅是枚举的名字而已.所以可以猜测一下反序列的的代码实现

```java
     java.io.ObjectOutputStream
    private Object readObject0(boolean unshared) throws IOException {
        ...
        switch (tc) {
            ...
            case TC_ENUM:
            return checkResolve(readEnum(unshared));
            ...
        }
        ...
    }

    /**
     * Reads in and returns enum constant, or null if enum type is
     * unresolvable.  Sets passHandle to enum constant's assigned handle.
     */
    private Enum readEnum(boolean unshared) throws IOException {
        ...
        if (cl != null) {
            try {
                en = Enum.valueOf(cl, name);
            } catch (IllegalArgumentException ex) {
                throw (IOException) new InvalidObjectException(
                    "enum constant " + name + " does not exist in " +
                    cl).initCause(ex);
            }
            if (!unshared) {
                handles.setObject(enumHandle, en);
            }
        }
        ...
        return en;
    }
```

反序列化也仅仅是通过name调用了方法 *Enum.valueOf(Class<T> enumType, String name)* 获取了一个枚举实例, 所以枚举也可以防止通过序列化产生新的单例.

**友情建议: 在序列化枚举时要特别注意, 枚举的名称一定不能改变, 否则在反序列化时有可能会抛出异常!!!**

```java
    public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                                String name) {
        T result = enumType.enumConstantDirectory().get(name);
        if (result != null)
            return result;
        if (name == null)
            throw new NullPointerException("Name is null");
        throw new IllegalArgumentException(
            "No enum constant " + enumType.getCanonicalName() + "." + name);
    }
```
