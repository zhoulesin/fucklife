# Android数据序列化

数据序列化在Android应用开发中占据着举足轻重的位置,无论是进程间通信,本地数据存储还是网络数据传输等.都离不开序列化的支持..

序列化是将数据结构或者对象转换成可用于存储或者传输的数据格式的过程,在序列化期间,数据结构或者对象将其状态信息写入到临时或者持久性存储区中;反序列化时将序列化过程中生成的数据还原成数据结构或对象的过程.

## 1.Serializable

Serializable是Java语言的特性,它是最简单的也是使用最广泛的序列化方案之一,只有实现了Serializable接口的Java对象才可以实现序列化.这种类型的序列化是将Java对象转换成字节序列的过程,而反序列化时将字节序列恢复成Java对象的过程.

Serializable接口是一种标识接口,也就是无需实现方法,Java便会对这个对象进行序列化操作.它的缺点是使用反射机制,在序列化的过程中会创建很多临时对象,容易触发垃圾回收,序列化的过程比较慢,对于性能要求很严格的场景不建议使用这种方案.

首先定义Java对象类,他必须实现Serializable接口.序列化过程中会使用名为SerialVersionUID的版本号和序列化的类相关联,serialVersionUID在反序列化过程中用于验证序列化对象的发送者和接收者是否为该对象加载了与序列化兼容的类对象,如果接收者加载的对象的serialVersionUID和发送者加载的对象的serialVersionUID取值不同,则反序列化过程会出现InvalidClassException异常,这种情况在网络通信两个节点间的序列化特别需要注意.最佳实践是显式指定serialVersionUID的值

```java
public class WenkuBanner implements Serializable{
    private static final long serialVersionUID = 941513132453123453L;
    
    public int mAccessTime;
    
    public ArrayList<Image> mContentList;
    
    public class Image implements Serializable{
        private static final long serialVersionUID = 845345386453453454L;
        public int mType;
        public String mIconUrl;
        public String mValue;
    }
    
    /**
     *	默认构造函数,作为空对象使用
     */
    public WenkuBanner(){
        mAccessTime = 0;
        mContentList = new ArrayList<>();
    }
}
```

如果想要序列化一个对象,首先需要创建某种类型的OutputStream,例如ByteArrayOutputStream,FileOutputStream等,夹着将这些OutputStream