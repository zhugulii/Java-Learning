# Java 基础

## 1. 数据类型

### 1.1 基本类型

|  类型   |   大小   |
| :-----: | :------: |
|  byte   | 1 个字节 |
|  char   | 2 个字节 |
|  short  | 2 个字节 |
|   int   | 4 个字节 |
|  long   | 8 个字节 |
|  float  | 4 个字节 |
| double  | 8 个字节 |
| boolean |    ~     |

- boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。
- JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。
- JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的。

### 1.2 包装类型

- 基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。

	```java
	Integer x = 2;  // 自动装箱
	int y = x;      // 自动拆箱
	```

### 1.3 缓存池

- new Integer(123) 与 Integer.valueOf(123) 的区别在于：

	- new Integer(123) 每次都会新建一个对象；

	- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

		```java
		Integer x = new Integer(123);
		Integer y = new Integer(123);
		System.out.println(x == y);          // false
		
		Integer m = Integer.valueOf(123);
		Integer n = Integer.valueOf(123);
		System.out.println(m == n);          // true
		```

- valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。

	```java
	public static Integer valueOf(int i) { 
	    if (i >= IntegerCache.low && i <= IntegerCache.high) 
	      	return IntegerCache.cache[i + (-IntegerCache.low)]; 
	    return new Integer(i); 
	}
	```

- 在 Java 8 中，Integer 缓存池的大小默认为 -128~127。 

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

- 编译器会在自动装箱过程调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象。

	```java
	Integer m = 123;
	Integer n = 123;
	System.out.println(m == n);  // true
	```

- 基本类型对应的缓冲池如下：

	- boolean values true and false
	- all byte values
	- short values between -128 and 127
	- int values between -128 and 127
	- char in the range \u0000 to \u007F

- 在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以直接使用缓冲池中的对象。

- 在 jdk 1.8 所有的数值类缓冲池中，Integer 的缓冲池 IntegerCache 很特殊，这个缓冲池的下界是 - 128，上界默认是 127，但是这个上界是可调的，在启动 jvm 的时候，通过 -XX:AutoBoxCacheMax=\<size> 来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界。

## 2. String

### 2.1 概览

- String 被声明为 final，因此它不可被继承。

- 在 Java 8 中，String 内部使用 char 数组存储数据。

	```java
	public final class String implements java.io.Serializable, Comparable<String>, CharSequence { 
	    /** The value is used for character storage. */ 
	    private final char value[]; 
	}
	```

- 在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 coder 来标识使用了哪种编码。

	```java
	public final class String implements java.io.Serializable, Comparable<String>, CharSequence { 
	    /** The value is used for character storage. */ 
	    private final byte[] value; 
	    /** The identifier of the encoding used to encode the bytes in {@code value}. */ 
	    private final byte coder; 
	}
	```

- value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

### 2.2 不可变的好处

- **可以缓存 hash 值**

	- 因为 String 的 hash 值经常被使用，例如 String 用作 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

- **String Pool 的需要**

	- 如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

		<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214143506.png" alt="image-20200918173853626" style="zoom:33%;" />

- **安全性**

	- String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

- **线程安全**

	- String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

### 2.3 String、StringBuffer、StringBuilder

- **可变性**
	- String 不可变。
	- StringBuffer 和 StringBuilder 可变。
- **线程安全**
	- String 不可变，因此是线程安全的。
	- StringBuilder 不是线程安全的。
	- StringBuffer 是线程安全的，内部使用 synchronized 进行同步。

### 2.4 String Pool

- 字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程中将字符串添加到 String Pool 中。

- 当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

- 下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同字符串，而 s3 和 s4 是通过 s1.intern() 方法取得一个字符串引用。intern() 首先把 s1 引用的字符串放到 String Pool 中，然后返回这个字符串引用。因此 s3 和 s4 引用的是同一个字符串。

	```java
	String s1 = new String("aaa"); 
	String s2 = new String("aaa"); 
	System.out.println(s1 == s2); // false 
	String s3 = s1.intern(); 
	String s4 = s1.intern(); 
	System.out.println(s3 == s4); // true
	```

- 如果是采用 "bbb" 这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。

	```java
	String s5 = "bbb"; 
	String s6 = "bbb"; 
	System.out.println(s5 == s6); // true
	```

- 在 Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7，String Pool 被移到堆中。这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。

### 2.5 new String("abc")

- 使用这种方式一共会创建两个字符串对象（前提是 String Pool 中还没有 "abc" 字符串对象）。

	- "abc" 属于字符串字面量，因此编译时期会在 String Pool 中创建一个字符串对象，指向这个 "abc" 字符串字面量；

	- 而使用 new 的方式会在堆中创建一个字符串对象。

	- 以下是 String 构造函数的源码，可以看到，在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 数组内容，而是都会指向同一个 value 数组。

		```java
		public String(String original) { 
		  	this.value = original.value; 
		  	this.hash = original.hash; 
		}
		```

	- 测试

		```java
		String s = "abc";
		String s1 = new String("abc");
		String s2 = new String("abc");
		System.out.println(s == "abc");      // true
		System.out.println(s == s1);         // false
		System.out.println(s1 == s2);        // false
		```

## 3. 运算

### 3.1 参数传递

- Java 的参数是以值传递的形式传入方法中，而不是引用传递。
- 如果在方法中改变对象的字段值会改变原对象该字段值，因为改变的是同一个地址指向的内容。

### 3.2 float 与 double

- Java 不能隐式执行向下转型，因为这会使得精度降低。
- 1.1 字面量属于 double 类型，不能直接将 1.1 直接赋值给 float 变量，因为这是向下转型。
- 1.1f 字面量才是 float 类型。

### 3.3 隐式类型转换

- 因为字面量 1 是 int 类型，它比 short 类型精度要高，因此不能隐式地将 int 类型下转型为 short 类型。

	```java
	short num = 1;
	// num = num + 1;  不能向下转型
	```

- 但是使用 += 或者 ++ 运算符可以执行隐式类型转换。

	```java
	num += 1;
	num++;
	```

- 上面的语句相当于将 s1 + 1 的计算结果进行了向下转型：

	```java
	num = (short) (num + 1);
	```

### 3.4 switch

- 从 Java 7 开始，可以在 switch 条件判断语句中使用 String 对象。
- switch 不支持 long，是因为 switch 的设计初衷是对那些只有少数的几个值进行等值判断，如果值过于复杂，那么还是用 if 比较合适。

## 4. 继承

### 4.1 访问权限

- Java 中有三个访问权限修饰符：private、protected 以及 public，如果不加访问修饰符，表示包级可见。
- 可以对类或类中的成员（字段以及方法）加上访问修饰符。
	- 类可见表示其它类可以用这个类创建实例对象。
	- 成员可见表示其它类可以用这个类的实例对象访问到该成员。
- protected 用于修饰成员，表示在继承体系中成员对于子类可见，但是这个访问修饰符对于类没有意义。
- 一个模块不需要知道其他模块的内部工作情况，这个概念被称为信息隐藏或封装。因此访问权限应当尽可能地使每个类或者成员不被外界访问。
- 如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别。这是为了确保可以使用父类实例的地方都可以使用子类实例。
- 字段决不能是公有的，因为这么做的话就失去了对这个字段修改行为的控制，客户端可以对其随意修改。

| 访问权限  | 本类 | 本包 | 子类 | 非子类的外包类 |
| --------- | ---- | ---- | ---- | -------------- |
| public    | 是   | 是   | 是   | 是             |
| protected | 是   | 是   | 是   | 否             |
| default   | 是   | 是   | 否   | 否             |
| private   | 是   | 否   | 否   | 否             |

### 4.2 抽象类与接口

- **抽象类**
	- 抽象类和抽象方法都使用 abstract 关键字进行声明。如果一个类中包含抽象方法，那么这个类必须声明为抽象类。
	- 抽象类和普通类最大的区别是，抽象类不能被实例化，需要继承抽象类才能实例化其子类。
- **接口**
	- 接口是抽象类的延伸，在 Java 8 之前，它可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现。
	- 从 Java 8 开始，接口也可以拥有默认的方法实现，这是因为不支持默认方法的接口的维护成本太高了。在 Java 8 之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类。
	- 接口的成员（字段 + 方法）默认都是 public 的，并且不允许定义为 private 或者 protected。
	- 接口的字段默认都是 static 和 final 的。

### 4.3 super

- 访问父类的构造函数：可以使用 super() 函数访问父类的构造函数，从而委托父类完成一些初始化的工作。
- 访问父类的成员：如果子类重写了父类的某个方法，可以通过使用 super 关键字来引用父类的方法实现。

### 4.4 重写与重载

- **重写**
	- 存在于继承体系中，指子类实现了一个与父类在方法声明上完全相同的一个方法。
	- 重写有以下三个限制：
		- 子类方法的访问权限必须大于等于父类方法；
		- 子类方法的返回类型必须是父类方法返回类型或为其子类型。
		- 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型。
	- 使用 @Override 注解，可以让编译器帮忙检查是否满足上面的三个限制条件。
	- 在调用一个方法时，先从本类中查找看是否有对应的方法，如果没有查找到再到父类中查看，看是否有继承来的方法。否则就要对参数进行转型，转成父类之后看是否有对应的方法。总的来说，方法调用的优先级为：
		- this.func(this)
		- super.func(this)
		- this.func(super)
		- super.func(super)
- **重载**
	- 存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是参数类型、个数、顺序至少有一个不同。
	- 应该注意的是，返回值不同，其它都相同不算是重载。

## 5. Object 通用方法

### 5.1 概览

```java
public native int hashCode() 
public boolean equals(Object obj) 
protected native Object clone() throws CloneNotSupportedException 
public String toString() 
public final native Class<?> getClass() 
protected void finalize() throws Throwable {} 
public final native void notify()
public final native void notifyAll() 
public final native void wait(long timeout) throws InterruptedException 
public final void wait(long timeout, int nanos) throws InterruptedException 
public final void wait() throws InterruptedException
```

### 5.2 equals

- **自反性**

	```java
	x.equals(x);   // true
	```

- **对称性**

	```java
	x.equals(y) == y.equals(x);   // true
	```

- **传递性**

	```java
	if (x.equals(y) && y.equals(z)) 
	  	x.equals(z);   // true
	```

- **一致性**

	- 多次调用 equals() 方法结果不变

		```java
		x.equals(y) == x.equals(y);   // true
		```

- **与 null 的比较**

	- 对任何不是 null 的对象 x 调用 x.equals(null) 结果都是 false。

		```java
		x.equals(null);   // false
		```

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。

- 对于引用类型，== 判断两个变量是否引用同一个对象，而 equals() 判断引用的对象是否等价。

	```java
	Integer x = new Integer(1);
	Integer y = new Integer(1);
	System.out.println(x.equals(y));  // true
	System.out.println(x == y);       // false
	```

- **实现时注意**

	- 检查是否为同一个对象的引用，如果是直接返回 true；
	- 检查是否是同一个类型，如果不是，直接返回 false；
	- 将 Object 对象进行转型；
	- 判断每个关键域是否相等。

	```java
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
	        if (obj == null || this.getClass() != obj.getClass()) return false;
	        
	        EqualExample that = (EqualExample) obj;
	        
	        if (that.x != this.x) return false;
	        if (that.y != this.y) return false;
	        return this.z == that.z;
	    }
	    
	}
	```

### 5.3 hashCode()

- hashCode() 返回散列值，而 equals() 是用来判断两个对象是否等价。等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价。

- 在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象散列值也相等。

- 下面的代码中，新建了两个等价的对象，并将它们添加到 HashSet 中。我们希望将这两个对象当成一样的，只在集合中添加一个对象，但是因为 EqualExample 没有实现 hasCode() 方法，因此这两个对象的散列值是不同的，最终导致集合添加了两个等价的对象。

	```java
	EqualExample e1 = new EqualExample(1, 1, 1); 
	EqualExample e2 = new EqualExample(1, 1, 1); 
	System.out.println(e1.equals(e2)); // true 
	HashSet<EqualExample> set = new HashSet<>(); 
	set.add(e1); 
	set.add(e2); 
	System.out.println(set.size()); // 2
	```

- 理想的散列函数应当具有均匀性，即不相等的对象应当均匀分布到所有可能的散列值上。这就要求了散列函数要把所有域的值都考虑进来。可以将每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。R 一般取 31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位。

- 一个数与 31 相乘可以转换成移位和减法： 31*x == (x<<5)-x ，编译器会自动进行这个优化。

	```java
	@Override public int hashCode() { 
	    int result = 17; 
	    result = 31 * result + x; 
	    result = 31 * result + y; 
	    result = 31 * result + z; 
	    return result; 
	}
	```

### 5.4 toString()

- 默认返回 ToStringExample@4554617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。

### 5.5 clone()

- **cloneable**

	- clone() 是 Object 的 protected 方法，它不是 public，一个类不显式去重写 clone()，其它类就不能直接去调用该类实例的 clone() 方法。

		```java
		public class CloneExample { 
		  	private int a; 
		  	private int b; 
		}
		```

		```java
		CloneExample e1 = new CloneExample(); 
		// CloneExample e2 = e1.clone(); // 'clone()' has protected access in 'java.lang.Object'
		```

	- 重写 clone() 得到以下实现：

		```java
		public class CloneExample { 
		    private int a; 
		    private int b; 
		
		    @Override public CloneExample clone() throws CloneNotSupportedException { 
		      	return (CloneExample)super.clone(); 
		    } 
		}
		```

		```java
		CloneExample e1 = new CloneExample(); 
		try {
		  	CloneExample e2 = e1.clone(); 
		} catch (CloneNotSupportedException e) { 
		  	e.printStackTrace(); 
		}
		```

		```java
		java.lang.CloneNotSupportedException: CloneExample
		```

	- 以上抛出了 CloneNotSupportedException，这是因为 CloneExample 没有实现 Cloneable 接口。

	- 应该注意的是，clone() 方法并不是 Cloneable 接口的方法，而是 Object 的一个 protected 方法。Cloneable 接口只是规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException。 

		```java
		public class CloneExample implements Cloneable { 
		    private int a; 
		    private int b; 
		
		    @Override public CloneExample clone() throws CloneNotSupportedException { 
		      	return (CloneExample)super.clone(); 
		    } 
		}
		```

- **浅拷贝**

	- 拷贝对象和原始对象的引用类型引用同一个对象。

- **深拷贝**

	- 拷贝对象和原始对象的引用类型引用不同对象。

- **clone() 的替代方案**

	- 使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

## 6. 关键字

### 6.1 final

- **数据**
	- 声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。
		- 对于基本类型，final 使数值不变；
		- 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。
- **方法**
	- 声明方法不能被子类重写。
	- private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。
- **类**
	- 声明类不允许被继承。

### 6.2 static

- **静态变量**

	- 静态变量：又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它。静态变量在内存中只存在一份。
	- 实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死。

- **静态方法**

	- 静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。
	- 只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字。

- **静态语句块**

	- 静态语句块在类初始化时运行一次。

- **静态内部类**

	- 非静态内部类依赖于外部类的实例，而静态内部类不需要。

		```java
		public class OuterClass { 
		    class InnerClass { }
		    static class StaticInnerClass { }
		
		    public static void main(String[] args) {
		      	// 'OuterClass.this' cannot be referenced from a static context 
		        // InnerClass innerClass = new InnerClass(); 
		        OuterClass outerClass = new OuterClass(); 
		        InnerClass innerClass = outerClass.new InnerClass(); 
		        StaticInnerClass staticInnerClass = new StaticInnerClass(); 
		    } 
		}
		```

	- 静态内部类不能访问外部类的非静态的变量和方法。

- **静态导包**

	- 在使用静态变量和方法时不用再指明 ClassName，从而简化代码，但可读性大大降低。

		```java
		import static com.xxx.ClassName.*
		```

- **初始化顺序**

	- 静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序。
	- 存在继承的情况下，初始化顺序为：
		- 父类（静态变量、静态语句块）
		- 子类（静态变量、静态语句块）
		- 父类（实例变量、普通语句块）
		- 父类（构造函数）
		- 子类（实例变量、普通语句块）
		- 子类（构造函数）

## 7. 反射

- 每个类都有一个 **Class** 对象，包含了与类有关的信息。当编译一个新类时，会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。
- 类加载相当于 Class 对象的加载，类在第一次使用时才动态加载到 JVM 中。也可以使用 Class.forName("com.mysql.jdbc.Driver")  这种方式来控制类的加载，该方法会返回一个 Class 对象。
- 反射可以提供运行时的类信息，并且这个类可以在运行时才加载进来，甚至在编译时期该类的 .class 不存在也可以加载进来。
- Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：
	- **Field** ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
	- **Method** ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
	- **Constructor** ：可以用 Constructor 创建新的对象。
- **反射的优点：**
	- **可扩展性** ：应用程序可以利用全限定名创建可扩展对象的实例，来使用来自外部的用户自定义类。
	- **类浏览器和可视化开发环境** ：一个类浏览器需要可以枚举类的成员。可视化开发环境（如 IDE）可以从利用反射中可用的类型信息中受益，以帮助程序员编写正确的代码。
	- **调试器和测试工具** ： 调试器需要能够检查一个类里的私有成员。测试工具可以利用反射来自动地调用类里定义的可被发现的 API 定义，以确保一组测试中有较高的代码覆盖率。
- **反射的缺点：**
	- 尽管反射非常强大，但也不能滥用。如果一个功能可以不用反射完成，那么最好就不用。在我们使用反射技术时，下面几条内容应该牢记于心。
		- **性能开销** ：反射涉及了动态类型的解析，所以 JVM 无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。
		- **安全限制** ：使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如 Applet，那么这就是个问题了。
		- **内部暴露** ：由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用，这可能导致代码功能失调并破坏可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。

## 8. 异常

- Throwable 可以用来表示任何可以作为异常抛出的类，分为两种： **Error** 和 **Exception**。其中 Error 用来表示 JVM 无法处理的错误，Exception 分为两种：

	- **受检异常** ：需要用 try...catch... 语句捕获并进行处理，并且可以从异常中恢复；
	- **非受检异常** ：是程序运行时错误，例如除 0 会引发 Arithmetic Exception，此时程序崩溃并且无法恢复。

	<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214143531.png" alt="image-20200918222627796" style="zoom:33%;" />

## 9. 泛型

```java
public class Box<T> { 
    // T stands for "Type" 
    private T t; 
    public void set(T t) { 
      	this.t = t; 
    } 
    public T get() { 
     		sreturn t; 
    } 
}
```

## 10. 注解

- Java 注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。

- 

## 11. 一些问题

### 11.1 ⾯向对象和⾯向过程的区别

-  **面向过程**：面向过程性能比面向对象高。因为类调用时需要实例化，开销比较大，比较消耗资源。但是，面向过程没有面向对象易维 护、易复用、易扩展。 
-  **面向对象**：面向对象易维护、易复用、易扩展。因为面向对象有封装、继承、多态性的特性，所以可以设计出低耦合的系统，使系统更加灵活、更加易于维护。但是，面向对象性能比面向过程低。

### 11.2 Java 语言有哪些特点？

- 面向对象：封装、继承、多态。
- 平台无关性。
- 支持多线程。
- 编译与解释共存。

### 11.3 JVM、JDK、JRE

- **JVM**
	- Java 虚拟机是运行 Java 字节码的虚拟机。JVM 有针对不同系统的特定实现，目的是使用相同的字节码，他们都会给出相同的结果。字节码和不同系统的 JVM 实现是 Java 语言 ”一次编译，随处可以运行“ 的关键所在。
- **JDK**
	- JDK是 Java Development Kit，它是功能齐全的 Java SDK。它拥有 JRE 所拥有的一切，还有编译器 ( Javac ) 和工具 (如 javadoc 和 jdb )。它能够创建和编译程序。
- **JRE**
	- JRE 是 Java 运行时环境。它是运行已编译 Java 程序的所需的所有内容的集合，包括 JVM，Java 类库，Java 命令和其他的一些基础组件。但是，它不能用于创建新程序。

### 11.4 字符型常量和字符串常量的区别？

- **形式上：** 字符常量是单引号引起的一个字符；字符串常量是双引号引起的若干个字符。
- **含义上：** 字符常量相当于一个整型值，可以参加表达式运算；字符串常量代表一个地址值。
- **占内存大小：** 字符常量只占 2 个字节；字符串常量占若干个字节。

### 11.5 构造器是否可被重写？

- constructor 不能被 override，但是可以 overload。

### 11.6 重载与重写的区别？

- **重载：**
	- 发生在同一个类中，方法明必须相同，参数类型不同、个数不同、顺序不同，方法返回值和访问修饰符可以不同。
- **重写：**
	- 重写是子类对父类允许访问的方法的实现过程进行重新编写，发生在子类中，方法名、参数列表必须相同，返回值类型小于等于父类，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类。另外，如果父类方法访问修饰符为 private 则子类就不能重写该方法。**也就是说方法提供的行为改变，而方法的外貌并没有改变。**

### 11.7 Java 面向对象编程三大特性：封装、继承、多态

- **封装**
	- 封装将一个对象的属性私有化，同时提供一些可以被外界访问的属性方法，如果属性不想被外界访问，就可以不提供给外界访问。
- **继承**
	-  继承是在已存在的类的基础上建立新类的技术，新类的定乂可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承能够非常方便地复用以前的代码。
	-  注意：
		-  子类拥有父类对象所有的属性和方法(包括私有属性和私有方法)，但是父类中的私有属性和方法子类是无法访问,只是拥有。
		-  子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
		-  子类可以重写父类的方法。
- **多态**
	- 所谓多态就是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量到底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。 
	- 在 Java 中有两种形式可以实现多态：继承(多个子类对同一方法的重写)和接口(实现接口并覆盖接口中同一方法)。

### 11.8 String、StringBuffer 和 StringBuilder 的区别是什么？String 为什么是不可变的？

- **可变性**
	- String 类中使用 final 修饰字符数组来保存字符串，private final char value[] ，所以 String 对象是不可变的。在 Java 9 及之后，String 类的实现改用 byte 数组存储字符串  private final byte[] value ，所以 String 对象是不可变的。
	- String Builder 与 String Buffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串char[] value 但是没有用 final 修饰，所以这两种对象都是可变的。
- **线程安全性**
	- String 中的对象是不可变的，也就可以理解为常量，线程安全。
	- StringBuffer 对方法增加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。
	- StringBuilder 并没有对方法加同步锁，所以是非线程安全的。
- **性能**
	- 每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象。 
	- StringBuffer 每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。
	- 相同情况下使用 StringBuilder 相比使用 StringBuffer 仅能获得10%~15% 左右的性能提升，但却要冒多线程不安全的风险。
- **总结**
	- 操作少量的数据：适用 String。 
	- 单线程操作字符串缓冲区下操作大量数据：适用 StringBuilder。
	- 多线程操作字符串缓冲区下操作大量数据：适用 StringBuffer。

### 11.9 自动装箱与拆箱

- **装箱：** 将基本类型用它们对应的引用类型包装起来。
- **拆箱：** 将包装类型转换为基本数据类型。

### 11.10 在一个静态方法内调用一个非静态方法为什么是非法的？

-  由于静态方法可以不通过对象进行调用,因此在静态方法里，不能调用其他非静态变量，也不可以访问非静态变量成员。

### 11.11 在 Java 中定义一个不做事且没有参数的构造方法的作用？

-  Java 程序在执行子类的构造方法之前，如果没有用 super() 来调用父类特定的构造方法，则会调用父类中 “没有参数的构造方法”。因此，如果父类中只定义了有参数的构造方法，而在子类的构造方法中又没有用 super() 来调用父类中特定的构造方法，则编译时将发生错误，因为 Java 程序在父类中找不到没有参数的构造方法可供执行。解决办法是在父类里加上一个不做事且没有参数的构造方法。

### 11.12 接口和抽象类的区别是什么？

- 接口的方法默认是 public，所有方法在接口中不能有实现 ( Java8开始接口方法可以有默认实现)，而抽象类可以有非抽象的方法。 
- 接口中除了 static、 final 变量，不能有其他变量，而抽象类中则不一定。 
- 一个类可以实现多个接口，但只能实现一个抽象类。接口自己本身可以通过 extends 关键字扩展多个接口。 
- 接口方法默认修饰符是 public，抽象方法可以有 public、 protected和 default 这些修饰符(抽象方法就是为了被重写所以不能使用 private关键字修饰!)。 
- 从设计层面来说,抽象是对类的抽象，是一种模板设计，而接口是对行为的抽象是一种行为的规范。
- 注意：
	- 在 JDK8 中，接口也可以定义静态方法，可以直接用接口名调用。实现类和实现是不可以调用的。如果同时实现了两个接口，接口中定义了一样的默认方法，则必须重写，不然会报错。
	- JDK9 的接口被允许定义私有方法。

### 11.13 成员变量与局部变量的区别有哪些？

- 成员变量属于类，局部变量是在方法中定义的变量或是方法的参数。
- 成员变量可以被 public、private、static 等修饰符修饰，局部变量不可以。但是都可以被 final 修饰。
- 成员变量存在于堆内存，局部变量存在于栈内存。
- 成员变量随着对象的创建而存在，而局部变量随着方法的调用而自动消失。
- 成员变量如果没有被赋初值，则会自动以类型的默认值而赋值 (例外：被 final 修饰的成员变量必须显示的赋值) ，而局部变量则不会自动赋值。

### 11.14 创建一个对象用什么运算符？对象实体与对象引用有何不同？

- new 运算符。
- 对象实例在堆内存中，对象引用指向对象实例，对象引用存放在栈内存中。
- 一个对象引用可以指向 0 个或 1 个对象。一个对象可以有 n 个引用指向它。

### 11.15 什么是方法的返回值？返回值在类的方法里的作用是什么？

- 方法的返回值是指获取到某个方法体中的代码执行后产生的结果。
- 作用：接收结果，使得它可以用于其他操作。

### 11.16 一个类的构造方法的作用是什么？若一个类没有声明构造方法，该程序能正确执行吗？为什么？

-  主要作用是完成对类对象的初始化工作。可以执行，因为一个类即使没有声明构造方法也会有默认的不带参数的构造方法。

### 11.17 构造方法有哪些特性？

- 名字与类名相同。
- 没有返回值，但不能用 void 声明构造函数。
- 生成类的对象时自动执行，无需调用。

### 11.18 静态方法和实例方法有何不同？

-  在外部调用静态方法时，可以使用 "类名方法名" 的方式，也可以使用 "对象名方法名" 的方式。而实例方法只有后面这种方式。也就是说，调用静态方法可以无需创建对象。 
-  静态方法在访问本类的成员时，只允许访问静态成员(即静态成员变量和静态方法)，而不允许访问实例成员变量和实例方法；实例方法则无此限制。

### 11.19 对象的相等与指向它们的引用相等，两者有什么不同？

-  对象的相等，比的是内存中存放的内容是否相等。而引用相等，比较的是他们指向的内存地址是否相等。

### 11.20 在调用子类构造器方法之前会先调用父类没有参数的构造方法，其目的是？

- 帮助子类做初始化工作。

### 11.21 == 与 equals

- 基本数据类型：== 比较的是值。
- 引用数据类型：== 比较的是内存地址。 
- **equals**
	- 类没有覆盖 equals 方法，则通过 equals 比较类的两个对象时，等价于通过 == 比较这两个对象。
	- 类覆盖了 equals 方法。比较两个对象的内容是否相等，若它们的内容相等，则返回 true 。

### 11.22 hashCode 与 equals

- **hashCode()**
	-  hashCode 的作用是获取哈希码，也称为散列码；它实际上是返回一个 int 整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。 hashCode 定义在 JDK 的 Object java 中，这就意味着 Java 中的任何类都包含有 hashCode 函数。
- **为什么要有 hashCode**
	-  以 HashSet 如何检查重复为例子来说明为什么要有 hashCode: 当把对象加入 HashSet 时, HashSet 会先计算对象的 hashcode值来判断对象加入的位置，同时也会与该位置其他已经加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 equals() 方法来检查 hashcode 相等的对象是否真的相同。如果两者相同， Hashset 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。这样就大大减少了 equals 的次数，相应就大大提高了执行速度。

- hashCode 与 equals 的相关规定
	- 如果两个对象相等，则 hashcode 一定也是相同的。 
	- 两个对象相等，对两个对象分别调用 equals 方法都返回 true 。
	- 两个对象有相同的 hashcode 值，它们也不一定是相等的。 
	- 因此， equals方法被覆盖过，则 hashCode 方法也必须被覆盖。
	- hashcode 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等。

### 11.23 简述线程、程序、进程的基本概念。以及它们之间的关系是什么？

- **线程**与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以生多个线程。与进程不同的是同类的多个线程共享同一块内存空间和一组系统资源，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。
- **程序**是含有指令和数据的文件，被存储在磁盘或其他的数据存储设备中，程序是静态的代码。
- **进程**是程序的一次执行过程，是系统运行程序的基本单位，进程是动态的。

### 11.24 线程有哪些基本状态？

<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214143551.png" alt="image-20200920162352886" style="zoom:33%;" />

### 11.25 关于 final 关键字的总结

- final 主要用于三个地方：类、方法、变量。
	- 对于一个 final 变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。 
	- 当用 final 修饰一个类时，表明这个类不能被继承。 final 类中的所有成员方法都会被隐式地指定为 final 方法。 
	- 使用 final 方法的是把方法锁定，以防任何继承类修改它的含义。
	- 类中所有的 private 方法都隐式地指定为 final。

### 11.26 Java 中的异常处理

- **Java 异常类层次结构图**

	<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214143611.png" alt="image-20200920162828285" style="zoom:33%;" />

	- 在 Java 中，所有的异常都有一个共同的祖先 java. lang 包中的 Throwable 类。 
	- Throwable：有两个重要的子类， Exception (异常)和 Error (错误)。 二者都是 Java 异常处理的重要子类，各自都包含大量子类。
	- 异常和错误的区别：异常能被程序本身处理，错误无法处理。

- **Throwable 类常用方法**

	- public string getMessage()：返回异常发生时的简要描述。
	- public string toString()：返回异常发生时的详细信息。
	- public string getLocalizedMessage()：返回异常对象的本地化信息。如果没有覆盖则与 getMessage() 返回的结果相同。
	- public void printStackTrace()：在控制台打印 Throwable 对象封装的异常信息。

- **异常处理总结**

	- try 块：用于捕获异常。其后可接零个或多个 catch 块，如果没有 catch 块，则必须跟一个 finally 块。
	- catch 块：用于处理 try 捕获的异常。
	- finally 块：无论是否捕获或处理异常，finally 块里的语句都会被执行。当在 try 块或 catch 块中遇到 return 语句时，finally 语句块将在方法返回之前被执行。

- 以下情况下，finally 不会被执行：

	- 在 finally 语句块第一行发生了异常。
	- 在前面的代码中用了 System. exit(int) 已退出程序。 
	- 程序所在的线程死亡。
	- 关闭 CPU。

- 注意：当 try 语句和 finally 语句中都有 return 语句时，在方法返回之前， finally 语句的内容将被执行，并且 finally 语句的返回值将会覆盖原始的返回值。

### 11.27 Java 序列化中如果有些字段不想进行序列化，怎么办？

- 对于不想进行序列化的变量，使用 transient 关键字修饰。 
- transient 关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复。 transient 只能修饰变量，不能修饰类和方法。

### 11.28 获取用键盘输入常用的两种方法

- 通过 Scanner

	```java
	Scanner sc = new Scanner(System.in);
	String s = sc.readLine();
	sc.close;
	```

- 通过 BufferedReader

	```java
	BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
	String s = br.readLine();
	```

### 11.29 Java 中 IO 流

#### 11.29.1 Java 中 IO 流分为几种？

- 按照流的流向分，可以分为输入流和输出流。

- 按照操作单元划分，可以划分为字节流和字符流。

- 按照流的角色划分为节点流和处理流。

- Java IO 流共涉及 40 多个类，这些类看上去很杂乱，但实际上很有规则，而且彼此之间存在非常紧密的联系， Java IO 流的 40 多个类都是从如下 4 个抽象类基类中派生出来的。

	- InputStream/Reader：所有的输入流的基类，前者是字节输入流，后者是字符输入流。 
	- OutputStream/Writer：所有输岀流的基类，前者是字节输岀流，后者是字符输岀流。

- 按操作方式分类结构图：

	<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214143645.png" alt="image-20200920172019800" style="zoom:33%;" />

- 按操作对象分类结构图：

	<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214143746.png" alt="image-20200920172110087" style="zoom:33%;" />

#### 11.29.2 既然有了字节流，为什么还要有字符流？

- 字符流是由 java 虚拟机将字节转换得到的，问题就岀在这个过程还算是非常耗时，并且，如果我们不知道编码类型就很容易出现乱码问题。所以，I/O流就干脆提供了个直接操作字符的接口，方便我们平时对字符进行流操作。如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的话使用字符流比较好。


#### 11.29.3 BIO、NIO、AIO 有什么区别？

- **BIO：** **Blocking IO** 同步阻塞 IO 模式，数据的读取写入必须阻塞在一个线程内等待其完成。
- **NIO：** **Non-blocking IO** 同步非阻塞 IO 模型。
- **AIO：** **Asynchronous IO**  异步非阻塞的 IO 模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

### 11.30 深拷贝、浅拷贝

<img src="https://raw.githubusercontent.com/zhugulii/picBed/master/20201214143102.png" alt="image-20200920174109143" style="zoom:33%;" />