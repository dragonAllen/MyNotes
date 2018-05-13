# 链接地址

(本文据网上资料与JDK源码整理为个人学习之用,JDK不同版本有所区别,有错漏之处敬请指出)

[Java 7 源码学习系列（一）——String](http://www.hollischuang.com/archives/99)

[五月的仓颉](http://www.cnblogs.com/xrq730/p/4841518.html)

[StringBuffer与 StringBuilder的应用场景](https://www.zhihu.com/question/20101840)

# String

### A. 定义

```java
//String是final类型的,表示该类不能被继承,同时该类实现了三个接口:java.io.Serializable,Comparable<String>,CharSequence
public final class String implements java.io.Serializable,Comparable<String>,CharSequence{
    
}
```



### B. 属性

```Java
//String的内容一旦被初始化了是不能被更改的,是用char[]实现的;
private final char value[];
//缓存字符串的hash Code,默认值为 0
private int hash;

//String实现了Serializable接口,所以支持序列化和反序列化支持。Java的序列化机制是通过在运行时判断类的serialVersionUID来验证版本一致性的;在进行反序列化时,JVM会把传来的字节流中的serialVersionUID与本地相应实体(类)的serialVersionUID进行比较,如果相同就认为是一致的,可以进行反序列化,否则就会出现序列化版本不一致的异常(InvalidCastException);
private static final long serialVersionUID = -6849794470754667710L;
private static final ObjectStreamField[] serialPersistentFields = new ObjectStreamField[0];
```



### C. 构造方法

#### 1. 使用字符数组,字符串构造一个String

```Java
public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }

public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= value.length) {
                this.value = "".value;
                return;
            }
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }
```

当我们使用字符数组创建String的时候,会用到Arrays.copyOf方法和Arrays.copyOfRange方法;

这两个方法是将原有的字符数组中的内容逐一的复制到String中的字符数组中;同样,我们也可以用一个String类型的对象来初始化一个String;这里将直接将`源String`中的`value`和`hash`两个属性直接赋值给`目标String`;



#### 2. 使用字节数组构造一个String

String实例中保存有一个`char[]`字符数组,`char[]`字符数组是以unicode码来存储的,String 和 char 为内存形式,byte是网络传输或存储的序列化形式;

String提供了一系列重载的构造方法来将一个字符数组转化成String,提到byte[]和String之间的相互转换就不得不关注编码问题;String(byte[] bytes, Charset charset)是指通过charset来解码指定的byte数组,将其解码成unicode的char[]数组,够造成新的String;

使用的是构造方法(带有`charsetName`或者`charset`参数)的一种的话,那么就会使用`StringCoding.decode`方法进行解码,使用的解码的字符集就是我们指定的`charsetName`或者`charset`;在使用byte[]构造String的时候,如果没有指明解码使用的字符集的话,那么`StringCoding`的`decode`方法首先调用系统的默认编码格式,如果没有指定编码格式则默认使用**ISO-8859-1**编码格式进行编码操作;



#### 3. 使用StringBuffer和StringBuider构造一个String

这两个构造方法是很少用到的



### D. equals方法

```java
public boolean equals(Object anObject) {
    	//判断要比较的对象和当前对象是不是同一个对象
        if (this == anObject) {
            return true;
        }
    	//判断anObject是不是String类型的
        if (anObject instanceof String) {
            String anotherString = (String) anObject;
            int n = value.length;
            //比较字符数组的时候,先比较了两个数组的长度,不一样直接返回false
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                            return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```



### E. substring方法

```Java
    public String substring(int beginIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        int subLen = value.length - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
    }
```



### F. StringBuilder与StringBuffer

StringBuffer (线程安全)
StringBuilder (非线程安全)

在大部分情况下效率 StringBuilder > StringBuffer;

StringBuilder对象是动态对象,允许扩充它所封装的字符串中字符的数量,当达到容量时,将自动分配新的空间且容量翻倍 int newCapacity = value.length \* 2 + 2;有频繁作字符串附加的需求,使用StringBuilder 类能使效率大大提高;

#### Java中为什么StringBuilder比StringBuffer快?

```java
public StringBuilder append(String str) {
        super.append(str);
        return this;
    }
    
public synchronized StringBuffer append(String str) {
        super.append(str);
        return this;
    }    
```



#### 避免"+"和 concat()的原因?

编译器每次碰到"+"的时候,会new一个StringBuilder出来,接着调用append方法,在调用toString方法,生成新字符串;

由于"+"拼接字符串与String的concat方法拼接字符串的低效,我们才需要使用StringBuilder和StringBuffer来拼接字符串;



#### StringBuilder拼接效率

最早是没有stringbuilder的,sun的人不知处于何种愚蠢的考虑,决定让stringbuffer是线程安全的,然后大约10年之后,人们终于意识到这是一个多么愚蠢的决定,意识到在这10年之中这个愚蠢的决定为java运行速度慢这样的流言贡献了多大的力量,于是,在jdk1.5的时候,终于决定提供一个非线程安全的stringbuffer实现,并命名为stringbuilder;顺便,javac好像大概也是从这个版本开始,把所有用加号连接的string运算都隐式的改写成stringbuilder,也就是说,从jdk1.5开始,用加号拼接字符串已经没有任何性能损失了;

如果没有循环的情况下,单行用加号拼接字符串是没有性能损失的,java编译器会隐式的替换成stringbuilder,但在有循环的情况下,编译器没法做到足够智能的替换,仍然会有不必要的性能损耗,因此,用循环拼接字符串的时候,还是老老实实的用stringbuilder;



### G. 为什么String=String？

在JVM中有一块区域叫做常量池,常量池中的数据是那些在编译期间被确定,并被保存在已编译的.class文件中的一些数据;除了包含所有的8种基本数据类型(char,byte,short,int,long,float,double,boolean)外,还有String及其数组的常量值,另外还有一些以文本形式出现的符号引用;

Java栈的特点是存取速度快(比堆块),但是空间小,数据生命周期固定,只能生存到方法结束;

