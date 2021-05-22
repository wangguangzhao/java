#### 数据类型

------

- **基本数据类型**

  - byte：8
  - char：16
  - short：16
  - int：32
  - float：32
  - long：64
  - double：64
  - boolean：~

  boolean只有两个值：true、false，可以使用1bit来存储，但是具体大小没有明确规定。JVM会在编译时期将boolean类型的数据转换为int，使用1来表示true，0表示false。JVM支持boolean数组，但是是通过读写byte数组来实现的。

- **包装类型**

  所有的基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成

  ```java
  Integer x = 2; //装箱 调用了Integer.valueOf(2)
  int y = x; //拆箱 调用了x.intValue()
  ```

  

- **缓存池 **

  new Integer(123) 与 Integer.valueOf(123) 的区别在于：

  - new Integer(123) 每次都会新建一个对象

  - Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

    ```java
    Integer x = new Integer(123);
    Integer y = new Integer(123);
    
    System.out.println(x == y); //false
    
    Integer x1 = Integer.valueOf(123);
    Integer y1 = Integer.valueOf(123);
    
    System.out.println(x1 == y1); //true
    ```

    valueOf的方法实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池中的内容

    ```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
    ```

    在java 8 中，Integer缓存池的大小默认为-128~127

    ```java
    static final int low = -128;
    static final int high;
    static final Integer cache[];
    
    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;
    
        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);
    
        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }
    
    ```

    编译器会在自动装箱拆箱过程中调用valueOf（） 方法，因此多个值相且值在缓存池范围内的Integer实力使用自动装箱来创建，那么就会引用相同的对象

    ```java
    Integer m = 123;
    Integer n = 123;
    System.out.println(m==n); // true
    ```

    基本类型对应的缓冲池如下

    - boolean value true and false
    - all byte values
    - Short  values between -128 ~127
    - Int values between -128~127
    - Char in the range \u0000~\u007F

    在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以使用缓冲池中的对象

     在jdk 1.8所有的数值类缓冲池中，Integer的缓冲池IntegerCache很特殊，这个缓冲池的下界是-128，默认上界是127，但是这个上界是可调的，在启动jvm的时候，通过 -xx：AutoBoxCacheMax=<size> 来指定这个缓冲池的大小，该选项在JVM初始化的时候回设定一个名为java.lang.IntegerCache.high系统属性，然后IntegerCache初始化的时候就会读取该系统属性来决定上界。

  #### String

  ------

  **概览**

  String被声明为final，因此他不可以被继承。（Integer等包装类也不能被继承）

  在java8中，String 内部使用char数组存储数据

  ```java
  public final class String
      implements java.io.Serializable, Comparable<String>, CharSequence {
      /** The value is used for character storage. */
      private final char value[];
  }
  
  ```

  在java9之后，String类的实现改用byte数组存储字符串，同时使用coder来表示使用了哪种编码。

  ```java
  public final class String
      implements java.io.Serializable, Comparable<String>, CharSequence {
      /** The value is used for character storage. */
      private final byte[] value;
  
      /** The identifier of the encoding used to encode the bytes in {@code value}. */
      private final byte coder;
  }
  ```

  value数组被声明为final，这意味着value数组初始化之后就能再引用其他数组。并且String内部没后改变value数组的方法，因此可以保证String不可变

  **不可变的好处**

  - 可缓存hash值

    因为String的hash值经常被使用，例如String用作HashMap的key。不可变的特性使得hash值也不可变，因此只需要进行一次计算。

  - String Pool的需要

    如果一个String对象已经被创建过了，那么就会从String Pool中取得引用只有String是不可变的，才能使用String Pool

    ```java
            String a = "123";
            String b = "123";
            System.out.println(a == b);
    ```

  - 安全性

    String经常作为参数，String不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果String是可变的，那么网络连接过程中String被改变，改变String的一方以为现在连接的是其它株机，而实际情况却不一定是

  - 线程安全

    String不可变性天生具备线程安全，可以在多个线程中安全使用

  - **String，StringBuffer and StringBuild**

    - 可变性
      - String 不可变
      - StringBuffer 和StringBuild可变
    - 线程安全性
      - String不可变，因此是线程安全的
      - StringBuild 不是线程安全的
      - StringBuffer 是线程安全的，内部使用synchronized进行同步

    **StringPool** （字符串常量池）

    字符串常量池保存着所有字符串字面量，这些字面量在编译时期就确定。不进如此，还可以使用String的 intern（）方法在运行过程将字符串添加到String Pool中。

    当一个字符串调用 intern() 方法时，如果String Pool 已经存在一个字符串和该字符串值相等（使用equals方法进行确定）,那么就会返回String Pool 中字符串的引用；否则就会在String Pool中添加一个新字符串，并返回这个新字符串的引用。

    下面实力中s1 和 s2 采用 new String（）的方式创建了两个不同的字符串，而s3和s4 是通过s1.intern() 和 s2.intern()方法取得同一个字符串引用。intern() 首先将 “aaa” 放到 String Pool 中，然后返回这个字符串引用，因此s3和s4引用的是同一个字符串

    ```java
     				String s1 = new String("aaa");
            String s2 = new String("aaa");
    
            System.out.println(s1 == s2); // false
    
            String s3 = s1.intern();
            String s4 = s2.intern();
    
            System.out.println(s3 == s4);
    ```

    如果采用 “bbb” 这种字面量的形式创建字符串，会自动将字符串放入String Pool 中

    ```java
    String s5 = "bbb";
    String s6 = "bbb";
    System.out.println(s5 == s6);  // true
    
    ```

    在java7之前，String Pool 被放在运行时常量池中，它属于永久代。而在java7，String Pool 被移到堆中。这时因为永久代的空间有限，在大量使用字符串的场景下会导致OutOfMemoryError错误

    - [StackOverflow : What is String interning?](https://stackoverflow.com/questions/10578984/what-is-string-interning)
    - [深入解析 String#intern](https://tech.meituan.com/in_depth_understanding_string_intern.html)

    **new String(abc)**

    使用这种方式一共会创建两个字符串对象（前提是String Pool中还没有"abc"字符串对象）

    - "abc"属于字符串字面量，因此编译时期会在String Pool中创建一个字符串对象，指向这个“abc”字符串字面量；
    - 而使用new方式会在堆中创建一个字符串对象

  创建一个测试类，其 main 方法中使用这种方式来创建字符串对象。

  ```
  public class NewStringTest {
      public static void main(String[] args) {
          String s = new String("abc");
      }
  }
  ```

  使用 javap -verbose 进行反编译，得到以下内容：

  ```
  // ...
  Constant pool:
  // ...
     #2 = Class              #18            // java/lang/String
     #3 = String             #19            // abc
  // ...
    #18 = Utf8               java/lang/String
    #19 = Utf8               abc
  // ...
  
    public static void main(java.lang.String[]);
      descriptor: ([Ljava/lang/String;)V
      flags: ACC_PUBLIC, ACC_STATIC
      Code:
        stack=3, locals=2, args_size=1
           0: new           #2                  // class java/lang/String
           3: dup
           4: ldc           #3                  // String abc
           6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
           9: astore_1
  // ...
  ```

  在 Constant Pool 中，#19 存储这字符串字面量 "abc"，#3 是 String Pool 的字符串对象，它指向 #19 这个字符串字面量。在 main 方法中，0: 行使用 new #2 在堆中创建一个字符串对象，并且使用 ldc #3 将 String Pool 中的字符串对象作为 String 构造函数的参数。

  以下是 String 构造函数的源码，可以看到，在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 数组内容，而是都会指向同一个 value 数组。

  ```
  public String(String original) {
      this.value = original.value;
      this.hash = original.hash;
  }
  ```

  #### 运算

  ------

  **参数传递**

  java的参数是以值传递的形式传入方法中，而不是引用传递。

  以下代码中的Dog dog 的dog是一个指针，存储的是对象的地址。在将一个参数传入一个方法时，本质上是将对象的地址以值得形式传递给形参中

  ```java
  public class Dog {
  
      String name;
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public Dog(String name) {
          this.name = name;
      }
  
  
      String getObjectAddress() {
  
          return super.toString();
      }
  }
  
  ```

  ```java
  package dataStructure.chaper2;
  
  public class Test {
  
      public static void main(String[] args) {
  
  
          Dog dog = new Dog("A");
          func(dog);
  
          System.out.println(dog.getName()); //B
  
  
      }
  
      private static void func(Dog dog) {
          dog.setName("B");
      }
  
  
  }
  
  ```

  在方法中改变对象的字段值会改变源对象该字段的值，因为引用的是同一个对象

  但是在方法中将指针引用了其他对象，那么此时方法里和方法外的两个指针指向了不同的对象，在一个指针改变其所指对象的内容对另一个指针所指向的对象没有影响。

  ```java
  package dataStructure.chaper2;
  
  public class Test {
  
      public static void main(String[] args) {
  
  
          Dog dog = new Dog("A");
          System.out.println(dog.getObjectAddress()); //dataStructure.chaper2.Dog@61bbe9ba
          func(dog);
  
          System.out.println(dog.getObjectAddress());//dataStructure.chaper2.Dog@61bbe9ba
  
          System.out.println(dog.getName()); //A
  
  
      }
  
      private static void func(Dog dog) {
          System.out.println(dog.getObjectAddress()); //dataStructure.chaper2.Dog@61bbe9ba
  
          dog = new Dog("B");
  
          System.out.println(dog.getObjectAddress());//dataStructure.chaper2.Dog@610455d6
          System.out.println(dog.getName()); //B
      }
  
  
  }
  
  ```

  **Float 与 double**

  java 不能隐式执行向下转型，因为这会使得精度降低。

  ```
  float是单精度类型,精度是8位有效数字，取值范围是10的-38次方到10的38次方，float占用4个字节的存储空间
  double是双精度类型，精度是17位有效数字，取值范围是10的-308次方到10的308次方，double占用8个字节的存储空间
  当你不声明的时候，默认小数都用double来表示，所以如果要用float的话，则应该在其后加上f
  例如：float a=1.3;
  则会提示不能将double转化成float  这成为窄型转化
  如果要用float来修饰的话，则应该使用float a=1.3f
  注意float是8位有效数字，第7位数字将会产生四舍五入
  所以如果一个float变量 这样定义:  float a=1.32344435;   则第7位将产生四舍五入(5及5以下的都将舍去)   
  ```

  

  - 字面量属于double类型，不能直接将1.1直接赋值给float变量，因为这是向下转型

    ```java
    float f = 1.1;
    ```

    1.1f字面量才是float类型

    ```java
    float f = 1.1f;
    ```

  - **隐式类型转换**

    因为字面量 1 是 int 类型，它比 short 类型精度要高，因此不能隐式地将 int 类型向下转型为 short 类型。

    ```
    short s1 = 1;
    // s1 = s1 + 1;
    ```

    但是使用 += 或者 ++ 运算符会执行隐式类型转换。

    ```
    s1 += 1;
    s1++;
    ```

    上面的语句相当于将 s1 + 1 的计算结果进行了向下转型：

    ```
    s1 = (short) (s1 + 1);
    ```

  - **switch**

    从java7 开始，可以在switch条件判断语句中使用String对象

    ```java
    
            String s = "a";
    
            switch (s) {
    
                case "a":
    
                    System.out.println("aaa");
                    break;
    
                case "b":
    
                    System.out.println("bbb");
                    break;
                    
    
            }
    ```

    switch 不支持 long、float、double，是因为 switch 的设计初衷是对那些只有少数几个值的类型进行等值判断，如果值过于复杂，那么还是用 if 比较合适

  #### 关键字

  - final

    - 数据

      声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能改变的常量

      - 对于基本类型，final使数值不变

      - 对于引用类型，final使引用不变，也就不能引用其他对象，但是被引用的对象本身是可以修改的。

        ```java
               final int x = 1;
        //        x = 2; //Cannot assign a value to final variable 'x'
                final Dog y = new Dog("A");
                y.setName("A");
        //         y = new Dog("B");//Cannot assign a value to final variable 'y'
        ```

        

    - 方法

      声明方法不能被子类重写。

      private方法隐式的被指定为final，如果在子类中定义的方法和积累中的一个private方法签名相同，此时子类方法不能重写基类方法，而是在子类中定了一个新的方法

    - 类

      声明类不允许被继承

  - static

    - 静态变量

      - 静态变量：有称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名访问它。静态变量在内存中只存在一份
      - 实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死

      ```java
      public class A{
      	private int x; //实例变量
      	private static int y;//静态变量
        
        public static void main(String[] args){
          // int x = A.x; //Non-static field 'x' cannot be refrenced from a static context
          A a = new A();
          int x = a.x;
          int y = A.y;
        }
      }
      ```

      

    - 静态方法

      静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。

      ```java
      public abstract class A{
        
        public static void func1(){
          
        }
        
        //public abstract static void func2();//Illegal combination of modifiers: 'abstract' and 'static'
      }
      ```

      只能访问所属类的静态字段和静态方法，方法中不能有this和super关键字，因为这两个关键字与具体实现有关。

      ```java
      public class A{
      	private static int x;
      	private int y;
      	
      	public static void func1(){
      		int a = x;
          //int b = y; //Non-static field 'y' cannot be referenced from a static context
          //int b = this.y; //'dataStructure.chaper2.Test.this' cannot be referenced from a static context
      	}
      	
      }
      ```

      

    - 静态语句块

      静态语句块在类初始化时运行一次。

      ```java
      package dataStructure.chaper2;
      
      public class A {
      
          static {
      
              System.out.println("123");
          }
      
          public static void main(String[] args) {
              A a1 = new A();
              A a2 = new A();
      
          }
      }
      
      ```

      ```java
      123
      ```

      

    - 静态内部类

      非静态内部类依赖于外部类的实例，也就是说需要先创建外部类实例，才能用这个实例去创建非静态内部类。而静态内部类不需要。

      ```java
      package dataStructure.chaper2;
      
      public class OuterClass {
      
          class InnerClass {
      
          }
      
          static class StaticInnerClass {
      
          }
      
          public static void main(String[] args) {
      
      //        InnerClass innerClass = new  InnerClass(); //'dataStructure.chaper2.OuterClass.this' cannot be referenced from a static context
              
              OuterClass outerClass = new OuterClass();
              InnerClass innerClass = outerClass.new InnerClass();
      
              StaticInnerClass staticInnerClass = new StaticInnerClass();
          }
      }
      
      ```

      静态内部类不能访问外部类的非静态变量和方法

    - 静态导包

      在使用静态变量和方法时不用再指明ClassName，从而简化代码，但可读性大大降低。

      ```java
      import static com.xx.ClassName.*
      ```

      

    - 初始化顺序

      静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于他们在代码中的顺序

      ```java
      public static String staticField = "静态变量"
      ```

      ```java
      static{
      	System.out.println("静态语句块");
      }
      ```

      ```java
      public String field = "实例变量";
      ```

      ```java
      {
      System.out.println("普通语句块");
      }
      ```

      最后才是构造函数的初始化

      ```java
      public InitialOrderTest(){
      	System.out.println("构造函数");
      }
      ```

      **存在继承的情况下初始化顺序为：**

      - 父类（静态变量，静态语句块）
      - 子类（静态变量，静态语句块）
      - 父类（实例变量、普通语句块）
      - 父类（构造函数）
      - 子类（实例变量、普通语句块）
      - 子类（构造函数）

    #### Object 通用方法

    概览

    ```java
    public native int hashCode();
    public boolean equals(Object obj)
    protected native Object clone() throws CloneNotSupportedException
    public String toString();
    public final native Class<?> getClass();
    protected void finalize() throws Throwable();
    public final native void notify();
    public final native void notifyAll();
    public final native void wait(long timeout) throws InterruptedException
    public final void wait(long timeout,int nanos) throws InterruptedException
    public final void wait() throws InterruptedException
    ```

    **equals()**

    - 等价关系

      两个对象具有等价关系，需要满足以下5个条件

      - 自反性： x.equals(x) ; //true

      - 对称性：x.equals(y) == y.equals(x);

      - 传递性 if(x.equals(y) && y.equals(x)) x.equals(z);

      - 一致性：x.equals(y) == x.equals(y); //多次调用equals方法结果不变

      - 与null比较：对于任何不适null的对象x，调用x.equlas(null)结果都为false

        

    - 等价于相等

      - 对于基本类型，==判断两个值是否相等，基本类型没有equals方法。

      - 对于引用类型，==判断两个变量是否为一个对象，而equals（）判断引用的对象是否等价。

        ```java
         Integer x = new Integer(1);
         Integer y = new Integer(1);
        
        System.out.println(x.equals(y)); //true
        System.out.println(x == y); //false
        
        ```

    - 实现

      - 检查是否为同一个对象的引用，如果是直接返回true；
      - 检查是否为同一个类型，如果不是，直接返回false；
      - 将Object对象进行转型；
      - 判断每个关键域是否相等

      ```java
      package dataStructure.chaper2;
      
      public class EqualExample {
      
          private int x;
          private int y;
          private int z;
      
          public EqualExample(int x, int y, int z) {
              this.x = x;
              this.y = y;
              this.z = z;
          }
      
          @Override
          public boolean equals(Object obj) {
      
              if (this == obj) return true;
              if (obj == null || getClass() != obj.getClass()) return false;
      
              EqualExample that = (EqualExample) obj;
              if (x != that.x) return false;
              if (y != that.y) return false;
      
              return z == that.z;
          }
      }
      
      ```

      

  - hashCode()

    hashCode() 返回哈希值，而equals() 是用来判断两个对象是否等价。等价的两个队形散列值一定相同，但是散列值相同的两个对象不一定等价，因为计算哈希值具有随机性，两个不同的对象可能计算出相同的哈希值。

    在覆盖equals（）方法时，应当总是覆盖hashCode 方法，保证等价的两个对象hash值也相等

    HashSet、HashMap等集合类使用了hashCode（）方法来计算对象应该存储的位置，因此要将对象添加到这些集合类中，需要让对应的类实现hashCode方法

    下面的代码中，新建了两个等价的对象，并将他们添加到HashSet中，我们希望这两个对象当成一样的，只在集合中添加一个对象，。但是EqualExample没有实现hashCode方法，因此这两个对象的hash值是不同的，最终导致集合添加了两个等价的对象

    ```java
    
            EqualExample e1 = new EqualExample(1,1,1);
            EqualExample e2 = new EqualExample(1,1,1);
    
            System.out.println(e1.equals(e2));
    
            HashSet<EqualExample> set = new HashSet<>();
            set.add(e1);
            set.add(e2);
    
            System.out.println(set.size());
    ```

    理想的哈希函数应当具有均匀性，即不相等的对象应当均匀分布到所有可能的哈希值上。这就要求了哈希函数要把所有域的值都考虑进来。可以将每个域都当成R进制的某一位，然后组成一个R进制函数。

    R一般取31，因为它是一个基素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与2想成相当于向左移一位，最左侧的位丢失。并且一个数与31想成可以转换成移位和减法：31*x == (x<<5) -x,编译器会自动进行这个优化。

  - toString()

    默认返回ToStringExample@4554617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。

    ```java
    package dataStructure.chaper2;
    
    public class ToStringExample {
    
        private int number;
    
        public ToStringExample(int number) {
            this.number = number;
        }
    }
    
    ```

    ```java
    package dataStructure.chaper2;
    
    import java.util.HashSet;
    
    public class Test {
    
        public static void main(String[] args) {
    
            ToStringExample toStringExample = new ToStringExample(123);
    
            System.out.println(toStringExample);//dataStructure.chaper2.ToStringExample@61bbe9ba
    
        }
    
    
    }
    
    ```

    

  - clone

    - cloneable

      clone() 是 Object 的 protected 方法，它不是 public，一个类不显式去重写 clone()，其它类就不能直接去调用该类实例的 clone() 方法。

      ```java
      public class CloneExample {
          private int a;
          private int b;
      }
      CloneExample e1 = new CloneExample();
      // CloneExample e2 = e1.clone(); // 'clone()' has protected access in 'java.lang.Object'
      ```

      重写 clone() 得到以下实现：

      ```java
      public class CloneExample {
          private int a;
          private int b;
      
          @Override
          public CloneExample clone() throws CloneNotSupportedException {
              return (CloneExample)super.clone();
          }
      }
      CloneExample e1 = new CloneExample();
      try {
          CloneExample e2 = e1.clone();
      } catch (CloneNotSupportedException e) {
          e.printStackTrace();
      }
      java.lang.CloneNotSupportedException: CloneExample
      ```

      以上抛出了 CloneNotSupportedException，这是因为 CloneExample 没有实现 Cloneable 接口。

      应该注意的是，clone() 方法并不是 Cloneable 接口的方法，而是 Object 的一个 protected 方法。Cloneable 接口只是规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException。

      ```java
      public class CloneExample implements Cloneable {
          private int a;
          private int b;
      
          @Override
          public Object clone() throws CloneNotSupportedException {
              return super.clone();
          }
      }
      ```

      

    - 浅拷贝

      浅拷贝对象和原始对象的引用类型引用同一个对象

      ```java
      package dataStructure.chaper2;
      
      public class ShallowCloneExample implements Cloneable {
      
      
          private int[] arr;
      
          public ShallowCloneExample() {
              arr = new int[10];
      
              for (int i = 0; i < arr.length; i++) {
                  arr[i] = i;
              }
          }
      
      
          public void set(int index, int value) {
              arr[index] = value;
          }
      
          public int get(int index) {
              return arr[index];
          }
      
          @Override
          protected ShallowCloneExample clone() throws CloneNotSupportedException {
              return (ShallowCloneExample) super.clone();
          }
      }
      
      ```

      ```java
      package dataStructure.chaper2;
      
      import java.util.HashSet;
      
      public class Test {
      
          public static void main(String[] args) throws CloneNotSupportedException {
      
      
              ShallowCloneExample e1 = new ShallowCloneExample();
      
              ShallowCloneExample e2 = null;
      
              e2 = e1.clone();
      
              e1.set(2, 222);
      
              System.out.println(e2.get(2));//222
          }
      
      
      }
      
      ```

      

    - 深拷贝

      深拷贝对象和原始对象的引用类型引用不同对象

      ```java
      package dataStructure.chaper2;
      
      public class DeepCloneExample implements Cloneable {
      
          private int[] arr;
      
          public DeepCloneExample() {
              arr = new int[10];
              for (int i = 0; i < arr.length; i++) {
                  arr[i] = i;
              }
          }
      
          public void set(int index, int value) {
              arr[index] = value;
          }
      
          public int get(int index) {
              return arr[index];
          }
      
          @Override
          protected DeepCloneExample clone() throws CloneNotSupportedException {
      
              DeepCloneExample result = (DeepCloneExample) super.clone();
      
              result.arr = new int[arr.length];
              for (int i = 0; i < arr.length; i++) {
                  result.arr[i] = i;
              }
      
              return result;
      
          }
      }
      
      ```

      ```java
      package dataStructure.chaper2;
      
      import java.util.HashSet;
      
      public class Test {
      
          public static void main(String[] args) throws CloneNotSupportedException {
      
      
              DeepCloneExample e1 = new DeepCloneExample();
      
              DeepCloneExample e2 = null;
      
              e2 = e1.clone();
      
              e1.set(2, 222);
      
              System.out.println(e2.get(2));//222
          }
      
      
      }
      
      ```

      

    - clone替代方案

      使用clone方法来拷贝一个对象即复杂又有风险，他会抛出异常，并且还需要转换类型。Effactive java书上讲到，最好不要去使用 clone，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象

      ```java
      package dataStructure.chaper2;
      
      public class CloneConstructorExample {
      
          private int[] arr;
      
          public CloneConstructorExample() {
      
              arr = new int[10];
              for (int i = 0; i < arr.length; i++) {
                  arr[i] = i;
              }
      
          }
      
          public CloneConstructorExample(CloneConstructorExample orginal) {
      
              arr = new int[orginal.arr.length];
              for (int i = 0; i < orginal.arr.length; i++) {
                  arr[i] = orginal.arr[i];
              }
      
          }
      
          public void set(int index, int value) {
              arr[index] = value;
          }
      
          public int get(int index) {
              return arr[index];
          }
      
      }
      
      ```

      ```java
      package dataStructure.chaper2;
      
      import java.util.HashSet;
      
      public class Test {
      
          public static void main(String[] args) throws CloneNotSupportedException {
      
      
              CloneConstructorExample e1 = new CloneConstructorExample();
      
              CloneConstructorExample e2 = new CloneConstructorExample(e1);
      
              e1.set(2, 222);
              System.out.println(e1.get(2)); //222
              System.out.println(e2.get(2)); //2
          }
      
      
      }
      
      ```

      

#### 继承

**访问权限**

java中有三个访问权限修饰符:private、 protected、public，如果不加访问修饰符，标识包级可见。

可以对类或类中的成员（字段和方法）加上访问修饰符。

- 类可见标识其他类可以用这个类创建实例对象
- 成员可见标识其他类可以用这个类的实例对象访问该成员

protected用于修饰成员，标识在继承体系中成员对于子类可见，但是这个访问修饰符对于类没有意义

设计良好的模块会隐藏所有实现细节，把他的API与他的实现清晰的隔离开。模块之间通过它们的API通信，一个模块不需要知道其他模块的内部工作情况，这个概念被称为信息隐藏或封装。因此访问权限应当尽可能的使每个类或者成员不被外界访问。

如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别。这时为了确保可以使用父类实例的地方都可以使用子类实例去替代，也就是确保满足里氏替换原则。

字段绝不能是公开的，因为这么做的话就失去了对这个字段修饰行为的控制，客户端可以对其随意修改，例如下面的例子中，AccessExample拥有id公有字段，如果在某个时刻，我们想要使用int存储id字段，就需要修改所有的客户端代码。

```java
public class AccessExample{
	public String id;
}
```

可以使用共有的getter和setter方法来替换公有字段，这样的话就可以该控制对字段的修改行为。

```java
public class AccessExample{
	private int id;
	public String getId(){
		return id+"";
	}
	public void SetId(String id){
    this.id = Integer.valueOf(id);
  }
}
```

但是也有例外，如果包级私有类或者私有嵌套类，那么直接暴露成员不会有特别大的影响。

```java
public class AccessWithInnerClassExample{
  	private class InnnerClass{
    	int x;  
   
    }
  	private InnerClass innerClass;
  	public AccessWithInnerClassExample(){
      innerClass = new InnerClass();
    }
  	public int getValue(){
      return innerClass.x; //直接访问
    }

}
```

**抽象类与接口**

- 抽象类

  抽象类和抽象方法都使用abstract关键字进行声明。如果一个类中包含抽象方法，那么这个类必须声明为抽象类。

  抽象类和普通类最大的区别是，抽象类不能实例化，只能被继承

  ```java
  package dataStructure.chaper2;
  
  public abstract class AbstractClassExample {
  
      protected int x;
      protected int y;
  
      public abstract void func1();
  
      public void func2() {
          System.out.println("func2");
      }
  
  }
  
  ```

  ```java
  package dataStructure.chaper2;
  
  public class AbstractExtendClassExample extends AbstractClassExample {
      @Override
      public void func1() {
          System.out.println("func1");
      }
  }
  
  ```

  ```java
  //       AbstractClassExample ac1 = new AbstractClassExample();//'AbstractClassExample' is abstract; cannot be instantiated
          
          AbstractClassExample ac2 = new AbstractExtendClassExample();
          ac2.func1();
  ```

  

- 接口

  接口是抽象类的延伸，在java8之前，他可以看成是一个完全抽象的类，也就是说它不能有任何抽象的方法实现。

  从java8开始，接口也可以拥有默认的方法实现，这时因为不支持默认方法的接口维护成本太高了。在java8之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类，让他们都实现新增的方法。

  接口的成员（字段+方法）默认都是public的，并且不允许定义为private或protected。从java9开始，允许将方法定义为private，这样就能定义某些服用的代码又不会把方法暴露出去。

  接口的字段默认都是static和final的

  ```java
  package dataStructure.chaper2;
  
  public interface InterfaceExample {
  
      void func1();
  
      default void func2() {
          System.out.println("func2");
      }
  
      int x = 23;
  //    int y; //Variable 'y' might not have been initialized
  
      public int z = 0;
  //    private int k =0;//Modifier 'private' not allowed here
  
      // protected int l = 0;//Modifier 'protected' not allowed here
  //    private void func3();//Modifier 'private' not allowed here
  
  }
  
  ```

  ```java
  package dataStructure.chaper2;
  
  public class InterfaceImplementExample implements InterfaceExample {
      @Override
      public void func1() {
          System.out.println("func1");
      }
  }
  
  ```

  ```java
  
  //        InterfaceExample ie1 = new InterfaceExample();//'InterfaceExample' is abstract; cannot be instantiated
          InterfaceExample interfaceExample = new InterfaceImplementExample();
  
          interfaceExample.func1();
          interfaceExample.func2();
          System.out.println(InterfaceExample.x);
  ```

  

- 比较

  - 从设计层面上看，抽象类提更了一种IS-A关系，需要满足里氏替换原则，即子类对象必须能够替换掉所有父类对象，而接口更像是一种LIKE-A关系，他只是提供一种方法实现契约，并不要求接口和实现接口的类具有IS-A关系。
  - 从使用上来看，一个类可以实现多个接口，但不能继承多个抽象类。
  - 接口的字段只能是static和final类型的，而抽象类的字段没有这种限制。
  - 接口的成员只能是public的，而抽象类的成员可以有多种访问限制。

- 使用选择

  - 使用接口：

    - 需要让不相关的类都实现一个方法，例如不相关的类都可以实现Comparable接口中的compareTo()方法；
    - 需要使用多重继承

  - 使用抽象类：

    - 需要在几个相关的类中共享代码。
    - 需要能控制继承来的成员访问权限，而不是都为public
    - 需要继承非静态和非常量字段

    在很多情况下，接口优于抽象类。因为接口没有抽象类严格的类层次结构要求，可以灵活地为一个类添加行为。并且从java8开始，接口也可以有默认的方法实现，使得修改接口的成本也变的很低。

- **super**

  - 访问父类的构造函数：可以使用super()访问父类的构造函数，从而委托父类完成一些初始化工作。应该注意到，子类一定会调用父类的构造函数来完成初始化工作，一般是调用父类的默认构造方法，如果子类需要调用父类的其他构造函数，那么就可以使用super（）函数

  - 访问父类的成员：如果子类重写了父类的某个方法，可以通过使用super关键字来引用父类的方法实现。

    ```java
    package dataStructure.chaper2;
    
    public class SuperExample {
    
        protected int x;
        protected int y;
    
        public SuperExample(int x, int y) {
            this.x = x;
            this.y = y;
        }
    
        public void func() {
            System.out.println("SuperExample.func()");
        }
    }
    
    ```

    

```java
package dataStructure.chaper2;

public class SuperExtendExample extends SuperExample {

    private int z;

    public SuperExtendExample(int x, int y, int z) {
        super(x, y);
        this.z = z;

    }

    @Override
    public void func() {
        super.func();
        System.out.println("SuperExtendExample.func()");
    }
}

```

```java
        SuperExample e = new SuperExtendExample(1,2,3);

        e.func();
```

```java
SuperExample.func()
SuperExtendExample.func()
```

**重写与重载**

- 重写

  存在于继承体系中，指子类实现了一个与父类在方法声明上完全相同的一个方法。

  为了满足里氏替换原则，重写有一下三个限制：

  - 子类的方法访问权限必须大于等于父类方法；
  - 子类的返回值类型必须是父类返回值类型或其子类型；
  - 子类抛出的异常必须是父类抛出的异常类型或其子类型；

  使用@Override注解，可以让编辑器帮忙检查是否满足上面的三个限制条件。

  下面实力中，SubClass为SuperClass的子类，SubClass重写了SuperClass的func方法。其中：

  - 子类的访问权限为public，大于父类的protected
  - 子类的返回值类型为Array<Integer>，是父类返回值类型List<Integer>的子类
  - 子类抛出的异常类型为Exception，是父类抛出异常Throwable的子类。
  - 子类重写方法使用@Override注解，从而让编译器自动检查是否满足条件

  ```java
  package dataStructure.chaper2;
  
  import java.util.ArrayList;
  import java.util.List;
  
  public class SuperClass {
  
      protected List<Integer> func() throws Throwable {
  
          return new ArrayList<>();
      }
  }
  
  ```

  ```java
  package dataStructure.chaper2;
  
  import java.util.ArrayList;
  
  public class SubClass extends SuperClass {
  
      @Override
      protected ArrayList<Integer> func() throws Exception {
  
  
          return new ArrayList<>();
      }
  }
  
  ```

  在调用一个方法时，先从本类中查找看是否有对应的方法，如果没有再到父类中查看，看是否从父类继承来。否则就要对参数进行转型，转成弗雷之后看是否有对应的方法。总的来说，方法调用的优先级为：

  - this.func(this)
  - Super.func(this)
  - this.func(super)
  - Super.func(super)

  ```java
  package dataStructure.chaper2;
  
  public class ABCD {
  
      public static void main(String[] args) {
  
          A a = new A();
          B b = new B();
          C c = new C();
          D d = new D();
  
          //在A中存在方法 show（A obj）,直接调用
          a.show(a);//A.show(A)
          //在A中不存在方法show（B obj）,将B转型寸其父类A
          a.show(b);//A.show(A)
  
          //在B中不存在从A中继承来的show（C obj），直接调用
          b.show(c);//A.show(C)
          //在B中不存在，但是存在从A中继承来的show（C obj）,将D转型成其父类C
          b.show(d);//A.show(C)
  
          //引用的还是B对象，所以ba和b的调用结果一样
          A ba = new B();
          ba.show(c); //A.show(C)
          ba.show(d);//A.show(C)
  
      }
  }
  
  
  class A {
      public void show(A obj) {
  
          System.out.println("A.show(A)");
      }
  
      public void show(C obj) {
          System.out.println("A.show(C)");
      }
  }
  
  class B extends A {
  
      @Override
      public void show(A obj) {
          System.out.println("B.show(A)");
      }
  }
  
  class C extends B {
  
  }
  
  class D extends C {
  
  }
  ```

  

- 重载

  存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是参数类型、个数、顺序至少有一个不同

  应该注意的是，返回值不同，其他都相同的不算是重载

  ```java
  package dataStructure.chaper2;
  
  public class OverloadingExample {
  
      public void show(int x) {
          System.out.println(x);
      }
  
      public void show(int x, int y) {
          System.out.println(x + "  " + y);
      }
  
  
      public static void main(String[] args) {
          OverloadingExample example = new OverloadingExample();
  
          example.show(1);
          example.show(1, 2);
      }
  }
  
  ```

  

#### 反射

------

每个类都有一个Class对象，包含了与类有关的信息。当编译一个新类时，会产生一个同名的.class文件，该文件内保存着Class对象。

类加载相当于Class对象的加载，类在第一次使用时才动态加载到JVM中。也可以使用Class.forName("com.mysql.jdbc.Driver")这种方式来控制类的加载，该方法会返回一个Class对象。

反射可以提供运行时的类信息，并且这个类可以在运行时才加载进来，甚至在编译时期的.class不存在也可以加载进来。

Class和java.lang.reflect一起对反射提供了支持，java.lang.reflect类库主要包含以下三个类：

- Field：可以使用get()和set() 方法读取和修改Field对象关联的字段
- Method：可以使用 invoke()方法调用与Method对象关联的方法
- Constructor：可以使用Constructor的newInstance()创建新的对象

**反射的优点：**

- 可扩展性：应用程序可以使用全限定名创建可拓展对象的实例，来使用来自外部用户自定义类
- 类浏览器和可视化开发环境：一个类浏览器可以枚举的成员。可视化环境（如IDE）可以利用从犯射中可用的类型信息中受益，以帮助程序员编写正确的代码
- 调试和测试工具：调试器需要能够检查一个类里的私有成员。测试工具可以利用反射来自动的调用类里定义的可被发现的API定义，以确保一组测试中有较高的代码覆盖率。

**反射的缺点：**

尽管反射非常强大，但也不能滥用。如故宫一个功能可以不用反射完成，那么最好就不用。在我们使用反射技术时，下面几条内容应该牢记于心。

- 性能开销：反射设计了动态类型解析，所以JVM无法对这些代码进行优化。因此反射操作的效率要比哪些非反射操作低的多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射
- 安全限制：使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在安全限制的环境中运行。如 Applet，那么这就是问题了。
- 内部暴露：由于反射允许代码执行一些在正常情况下不被允许的操作（比如私有的属性和方法），所有使用反射可能会导致意料之外的副作用，这可能导致代码功能失调并破坏可移植性。反射代码破坏了抽象性，因此平台发生改变是，代码的行为有可能也随着变化
- [Trail: The Reflection API](https://docs.oracle.com/javase/tutorial/reflect/index.html)
- [深入解析 Java 反射（1）- 基础](http://www.sczyh30.com/posts/Java/java-reflection-1/)

#### 异常

------

Throwable可以用来表示任何可以作为异常抛出的类，分为两种：Error和Exception。其中Error用来表示JVM无法处理的错误，Exception分类两种：

- 受检异常：需要用try...catch...语句捕获并进行处理，并且可以从异常中恢复
- 非受检异常：是程序运行时错误，例如除0会引发Arithmetic Exception，此时程序崩溃并无法恢复。
- [Java Exception Interview Questions and Answers](https://www.journaldev.com/2167/java-exception-interview-questions-and-answersl)
- [Java提高篇——Java 异常处理](https://www.cnblogs.com/Qian123/p/5715402.html)
- ![img](https://camo.githubusercontent.com/8b2dfb0bfaeaf7ea0f0ae387ce2cbb0da1ec8d88a23929623cafc6bcbff044af/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f50506a77502e706e67)

#### 泛型

------

```java
public class Box<T> {
    // T stands for "Type"
    private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

- [Java 泛型详解](https://www.cnblogs.com/Blue-Keroro/p/8875898.html)
- [10 道 Java 泛型面试题](https://cloud.tencent.com/developer/article/1033693)



#### 注解

------

Java注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用

[注解 Annotation 实现原理与自定义注解例子](https://www.cnblogs.com/acm-bingzi/p/javaAnnotation.html)

#### 特性

------

### Java 各版本的新特性

**New highlights in Java SE 8**

1. Lambda Expressions
2. Pipelines and Streams
3. Date and Time API
4. Default Methods
5. Type Annotations
6. Nashhorn JavaScript Engine
7. Concurrent Accumulators
8. Parallel operations
9. PermGen Error Removed

**New highlights in Java SE 7**

1. Strings in Switch Statement
2. Type Inference for Generic Instance Creation
3. Multiple Exception Handling
4. Support for Dynamic Languages
5. Try with Resources
6. Java nio Package
7. Binary Literals, Underscore in literals
8. Diamond Syntax

- [Difference between Java 1.8 and Java 1.7?](http://www.selfgrowth.com/articles/difference-between-java-18-and-java-17)
- [Java 8 特性](http://www.importnew.com/19345.html)

**java与C++的区别**

-  java是纯粹的面向对象语言，所有对象都继承自java.lang.Object，C++为了兼容C即支持面向对象也支持面向过程
- java通过虚拟机从而实现跨平台特性，但是C++依赖于特定的平台
- java没有指针，它的引用可以理解为安全指针，而C++具有和C一样的指针
- java支持自动垃圾回收，而C++需要手动回收
- java不支持多重继承，只能通过实现多个接口来达到相同目的，而C++支持多重继承
- java不支持操作符重载，虽然可以对两个String对象执行加法运算，但是这是语言内置支持的操作，不属于操作符重载，而C++可以。
- java的goto是保留至，但是不可用，C++可以使用goto

[What are the main differences between Java and C++?](http://cs-fundamentals.com/tech-interview/java/differences-between-java-and-cpp.php)



**JRE or JDK**

- JRE：java运行环境的简称，为java的运行提供了所属的环境。他是一个JVM程序，主要包括了JVM的标准实现和一些java基本类库
- JDK：java开发工具包，提供了java的开发及运行环境。JDK是java开发的核心，集成了JRE及一些其他的工具，比如编译java源代码的编译器javac等

