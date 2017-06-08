# ϸ˵����
------
###### ʲô�ǵ���ģʽ
>[����ģʽ](http://zh.wikipedia.org/wiki/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F)��Ҳ�е���ģʽ����һ�ֳ��õ�������ģʽ����Ӧ�����ģʽʱ���������������뱣ֻ֤��һ��ʵ�����ڡ�

����ԭ����ǣ�**�������������뱣ֻ֤��һ��ʵ������**

��java����Ҫ�����ֹ�����ʽ
* ������ʽ��ָȫ�ֵĵ���ʵ���ڵ�һ�α�ʹ��ʱ������
* ������ʽ��ָȫ�ֵĵ���ʵ������װ��ʱ������

�򵥵�˵����һ����Ҫ*�ӳٳ�ʼ��*��һ������Ҫ��

�Ƚϼ򵥵Ĺ�����ʽ�У�

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

���ַ�ʽʵ�ּ򵥣�ʵ������װ��ʱ�����������Ҫʵ��һ��ʵ���ڵ�һ�α�ʹ��ʱ����Ӧ����ô����

��һ�ֽ���*˫�ؼ����(double-checked locking)*

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

���ַ���ֻ������JDK5���Ժ�汾(ע�� INSTANCE ������Ϊ volatile)��֮ǰ�İ汾ʹ�á�˫�ؼ�������ᷢ����Ԥ����Ϊ.

�Ƽ��Ķ�:

1.  Effective Java ��71�� �����ӳٳ�ʼ��
2.  Core Java ��һ�� 14.5.8 Volatile ��
3.  [JSL 17.4](http://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4)
4.  [Java ������ʵ��: ��ȷʹ�� Volatile ����](http://www.ibm.com/developerworks/cn/java/j-jtp06197.html)
5.  [˫�ؼ���������ӳٳ�ʼ��](http://ifeve.com/double-checked-locking-with-delay-initialization/)

�ڵ�һ���Ƽ��Ķ����ᵽ����һ��ʵ�ֵ����ķ�ʽ*lazy initialization holder class idiom*

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

JVM����ĳ�ʼ���׶Σ�����Class�����غ��ұ��߳�ʹ��֮ǰ������ִ����ĳ�ʼ������ִ����ĳ�ʼ���ڼ䣬JVM��ȥ��ȡһ���������������ͬ������̶߳�ͬһ����ĳ�ʼ�����������ʵ�ַ�������double-checked locking�ȣ����ü���������ʵ�ִ����Ϊ��࣬���������а汾�ı������ж��ǿ��еġ�

����*static final Singleton INSTANCE*��ķ���Ȩ��Ϊʲôʱ����˽�п����Ķ�:[Initialization On Demand Holder idiom��ʵ��̽��](http://ifeve.com/initialization-on-demand-holder-idiom/)

--- ---

����Ƽ�ʵ����Ϊ����һ�ַ�ʽ: **ʹ��ö��**

���뼫����, ʹ�ü����:

```java
    public enum Singleton {
        INSTANCE;
    }
```

�ع�һ��ǰ�漯�е�����ʵ�ַ�ʽ, ��ֻ�����˳����ȡ�������ֶ�, Ȼ��������ͨ�����л��ͷ�����ƻ�ȡ����.�������ַ�ʽ���ʵ�������л��ӿ�*Serializable*�ͱ�����д*readResolve()*����

```java
    private Object readResolve(){
        return INSTANCE;
    }
```

��ʹ��д�� *readResolve()* ����Ҳ���漰ĳϵ����Ҫ�ؼ��� *transient* ������, �������۲���չ��, ��֮�漰���л�ͦ����. 

���ڷ�ֹ������ʱû�������˽�, ���˽�: ��Ϊ�����ĳЩ�ط��ƹ���java���Ƶ����ƣ�privateֻ�ڱ���ʱ����Ȩ�޵����ƣ�����������ʱ�ǲ���������Ȩ�޵����Ƶ�, �˴������ο�.

����ʹ��enumʵ�ֵĵ����Դ������л�������书��, ��ϸ����ö���෴��������(���ο�)

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

����ʵ�ֵ�ö�ٶ��Ǽ̳��� *java.lang.Enum* ���Կ�����������ʵ��Ҳ��ͨ���ؼ��� *static* ���εľ�̬��ʼ������ʵ��.

��ôΪʲôenum���Է���������...�ܼ�, ��Ϊ����һ�������� *public abstract class Singleton extends Enum* ��ʹ�Ƿ������Ҳ����ʵ������.

��Ϊʲô�ܷ������л���...���Ҫ��javaԴ���ж��ڶ������л��Ĵ���.

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

���Կ���enum�ڱ����л�ʱʱ�������⴦���, �����л��Ľ�����ö�ٵ����ֶ���.���Կ��Բ²�һ�·����еĵĴ���ʵ��

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

�����л�Ҳ������ͨ��name�����˷��� *Enum.valueOf(Class<T> enumType, String name)* ��ȡ��һ��ö��ʵ��, ����ö��Ҳ���Է�ֹͨ�����л������µĵ���.

**���齨��: �����л�ö��ʱҪ�ر�ע��, ö�ٵ�����һ�����ܸı�, �����ڷ����л�ʱ�п��ܻ��׳��쳣!!!**

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
