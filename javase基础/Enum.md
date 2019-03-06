# Enum

## 枚举类的定义

```
enum Name{
    TYPE1,TYPE2,TYPE3;
}
public class A{
	public static void main(String[] args){
        Name name=Name.TYPE1;  //可以直接引用
	}
}
```

## 枚举类的实现原理

实际上在使用关键字enum创建枚举类型并编译后，编译器会为我们生成一个相关的类，这个类继承了Java API中的java.lang.Enum类，也就是说通过关键字enum创建枚举类型在编译后事实上也是一个类类型而且该类继承自java.lang.Enum类。

利用反编译，可以得到enum.class文件的内容为:

```
Compiled from "A.java"
final class wjj.javalearning.enumtest.Name extends java.lang.Enum<wjj.javalearning.enumtest.Name> {
  public static final wjj.javalearning.enumtest.Name TYPE1;
  public static final wjj.javalearning.enumtest.Name TYPE2;
  public static final wjj.javalearning.enumtest.Name TYPE3;
  private static final wjj.javalearning.enumtest.Name[] ENUM$VALUES;
  static {};
  private wjj.javalearning.enumtest.Name(java.lang.String, int);
  public static wjj.javalearning.enumtest.Name[] values();
  public static wjj.javalearning.enumtest.Name valueOf(java.lang.String);
}
```

可以看到是final类型的，所以不能被继承。该类继承自java.lang.Enum。

## 枚举类的常用方法

| 返回类型   |  方法名称   |    方法说明 |
| ---       | :--        | :---      |
| int  |  compareTo(E o)   |    比较此枚举与指定对象的顺序 |
|boolean|equals(Object other)|当指定对象等于此枚举常量时，返回 true。|
|Class<?>|getDeclaringClass()|返回与此枚举常量的枚举类型相对应的 Class 对象|
|String|name()|返回此枚举常量的名称，在其枚举声明中对其进行声明|
|int|ordinal()|返回枚举常量的序数（它在枚举声明中的位置，其中初始常量序数为零）|
|String|toString()|返回枚举常量的名称，它包含在声明中|

```
public class A {
    public static void main(String[] args) {
//        Name name=Name.TYPE1;  //可以直接引用
        Name[] names = new Name[]{Name.TYPE1, Name.TYPE2, Name.TYPE3};
        for (Name t : names
                ) {
            System.out.println(t.ordinal());
        }
        System.out.println("--------");
        for (Name t : names
                ) {
            System.out.println(t.name());
        }
        System.out.println("--------");
        for (Name t : names
                ) {
            System.out.println(t.toString());
        }
        System.out.println("--------");
        System.out.println(Arrays.toString(Name.values()));
		System.out.println(Name.valueOf("TYPE1"));
    }
}
执行结果：
0
1
2
--------
TYPE1
TYPE2
TYPE3
--------
TYPE1
TYPE2
TYPE3
--------
[TYPE1, TYPE2, TYPE3]
TYPE1
```
values()方法的作用就是获取枚举类中的所有变量，并作为数组返回，而valueOf(String name)方法与Enum类中的valueOf方法的作用类似根据名称获取枚举变量，只不过编译器生成的valueOf方法更简洁些只需传递一个参数。这里我们还必须注意到，由于values()方法是由编译器插入到枚举类中的static方法，所以如果我们将枚举实例向上转型为Enum，那么values()方法将无法被调用，因为Enum类中并没有values()方法，valueOf()方法也是同样的道理，注意是一个参数的。

## 枚举与class对象

| 返回类型   |  方法名称   |    方法说明 |
| ---       | :--        | :---      |
|T[]| getEnumConstants()|返回该枚举类型的所有元素，如果Class对象不是枚举类型，则返回null。|
|boolean|isEnum()|当且仅当该类声明为源代码中的枚举时返回 true|

```
        Enum e = (Enum) Name.TYPE1;
        Class<?> clazz = e.getDeclaringClass();
        if (clazz.isEnum()) {
            Name[] names = (Name[]) clazz.getEnumConstants();
            System.out.println(Arrays.toString(names));
```

## 枚举类的进阶用法

### 向enum类添加方法与自定义构造函数
自定义私有构造函数，在声明枚举实例时传入对应的中文描述。enum类中确实可以像定义常规类一样声明变量或者成员方法。但是我们必须注意到，如果打算在enum类中定义方法，务必在声明完枚举实例后使用分号分开，倘若在枚举实例前定义任何方法，编译器都将会报错，无法编译通过，同时即使自定义了构造函数且enum的定义结束，我们也永远无法手动调用构造函数创建枚举实例，毕竟这事只能由编译器执行。
###可以覆盖enum类方法，如toString()
###enum类中定义抽象方法
与常规抽象类一样，enum类允许我们为其定义抽象方法，然后使每个枚举实例都实现该方法，以便产生不同的行为方式，注意abstract关键字对于枚举类来说并不是必须的。
###enum类与接口
由于Java单继承的原因，enum类并不能再继承其它类，但并不妨碍它实现接口，因此enum类同样是可以实现多接口的。
###枚举与switch
###枚举与单例模式
```
/**
 * 饿汉式（基于classloder机制避免了多线程的同步问题）
 */
public class SingletonHungry {

    private static SingletonHungry instance = new SingletonHungry();

    private SingletonHungry() {
    }

    public static SingletonHungry getInstance() {
        return instance;
    }
}

/**
 * 懒汉式单例模式（适合多线程安全）
 */
public class SingletonLazy {

    private static volatile SingletonLazy instance;

    private SingletonLazy() {
    }

    public static synchronized SingletonLazy getInstance() {
        if (instance == null) {
            instance = new SingletonLazy();
        }
        return instance;
    }
}
/**
 * 双重检测锁（效率高）
 */
public class Singleton {
    private static volatile Singleton singleton = null;

    private Singleton(){}

    public static Singleton getSingleton(){
        if(singleton == null){
            synchronized (Singleton.class){
                if(singleton == null){
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }    
}

/**
 * 静态内部类
 */
public class SingletonInner {
    private static class Holder {
        private static SingletonInner singleton = new SingletonInner();
    }

    private SingletonInner(){}

    public static SingletonInner getSingleton(){
        return Holder.singleton;
    }
}
//反序列化
public class Singleton implements java.io.Serializable {     
   public static Singleton INSTANCE = new Singleton();     

   protected Singleton() {     
   }  

   //反序列时直接返回当前INSTANCE
   private Object readResolve() {     
            return INSTANCE;     
      }    
}  
//破解反射
public static Singleton INSTANCE = new Singleton();     
private static volatile  boolean  flag = true;
private Singleton(){
    if(flag){
    flag = false;   
    }else{
        throw new RuntimeException("The instance  already exists ！");
    }
}


/**
 * 枚举单利
 */
public enum  SingletonEnum {
    INSTANCE;
    private String name;
    public String getName(){
        return name;
    }
    public void setName(String name){
        this.name = name;
    }
}



```

### EnumSet、EnumMap….



[参考博客](https://blog.csdn.net/javazejian/article/details/71333103)