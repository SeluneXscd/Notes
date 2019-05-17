[TOC]
- [一、数据类型](#%E4%B8%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)
	- [基本数据类型](#%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)
	- [引用类型](#%E5%BC%95%E7%94%A8%E7%B1%BB%E5%9E%8B)
	- [常量](#%E5%B8%B8%E9%87%8F)
	- [类型转换](#%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2)
	- [包装类型](#%E5%8C%85%E8%A3%85%E7%B1%BB%E5%9E%8B)
- [二、String](#%E4%BA%8Cstring)
	- [String 被声明为 final，因此它不可被继承。](#string-%E8%A2%AB%E5%A3%B0%E6%98%8E%E4%B8%BA-final%E5%9B%A0%E6%AD%A4%E5%AE%83%E4%B8%8D%E5%8F%AF%E8%A2%AB%E7%BB%A7%E6%89%BF)
	- [不可变的好处](#%E4%B8%8D%E5%8F%AF%E5%8F%98%E7%9A%84%E5%A5%BD%E5%A4%84)
	- [String, StringBuilder, StringBuffer](#string-stringbuilder-stringbuffer)
	- [String str = "i" 与 String str = new String("i")](#string-str-string-str-new-string)
	- [String Pool 字符串常量池](#string-pool-%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%B8%B8%E9%87%8F%E6%B1%A0)
	- [字符串反转](#%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%8F%8D%E8%BD%AC)
	- [String 类的常用方法](#string-%E7%B1%BB%E7%9A%84%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95)
	- [== 和 equals() 的区别](#%E5%92%8C-equals-%E7%9A%84%E5%8C%BA%E5%88%AB)	

[TOC]



# 一、数据类型

## 基本数据类型

|类型|缺省值|长度|数的范围|
|:-:|:-:|:-:|:-:|
|byte|0|8位|-128~127|
|short|0|16位|-32,768~32,767|
|int|0|32位|-2,147,483,648~2,147,483,647|
|long|0|64位|-9,223,372,036,854,775,808~<br>9,233,372,036,853,775,807|
|char|'\u0000'|16位|只能一个字符|
|float|0.0|32位|3.4E-038~3.4E+038|
|double|0.0|64位|1.7E-308~1.7E+308|
|boolean|false|1位|false, true|

boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示false。JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的。

## 引用类型

- 在Java中，引用类型的变量非常类似于C/C++的指针。引用类型指向一个对象，指向对象的变量是引用变量。这些变量在声明时被指定为一个特定的类型，比如 Employee、Puppy 等。变量一旦声明后，类型就不能被改变了。
- 对象、数组都是引用数据类型。
- 所有引用类型的默认值都是null。
- 一个引用变量可以用来引用任何与之兼容的类型。
- 例子：String str = new String("abc");

## 常量

常量在程序运行时是不能被修改的。
在 `Java` 中使用 `final` 关键字来修饰常量，声明方式和变量类似：

```java
final int i = 1;
```

## 类型转换

- 低精度向高精度转换，是正常转换的

```java
long l = 50;
int i = 50;
l = i;
```

- 高精度向低精度转换，需要强制转换

```java
byte b = 5;
int i1 = 10;
int i2 = 300;
b = (byte) i1;
b = (byte) i2;
```

## 包装类型

所有的基本类型，都有对应的类类型(包装类型)

- 基本类型转换为包装类，叫做装箱
- 包装类转换为基本类型，叫做拆箱

```java
Integer x = 2;     // 装箱
int y = x;         // 拆箱
```

int 的最大值可以通过对应的包装类`Integer,MAX_VALUE`获取

```java
System.out.println(Integer.MAX_VALUE);
```

# 二、String

## String 被声明为 final，因此它不可被继承。

在`java8`中，String内部使用`char`数组来存储数据

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
	/** The value is used for character storage. */
	private final char value[];
}
```

value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

## 不可变的好处

**1. 可以缓存 hash 值**
因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

**2. String Pool 的需要**
如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有String 是不可变的，才可能使用 String Pool。

**3. 安全性**
String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

**4. 线程安全**
String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

## String, StringBuilder, StringBuffer

- `String`声明的是不可变的对象，每次操作都会生成新的`String`对象，然后将指针指向新的`String`对象；而`StringBuffer`，`StringBuilder`可以在原有的对象基础上进行操作，所以在经常要改变字符串内容的情况下，不推荐使用`String`。

- `StringBuffer`和`StringBuilder`最大的区别在于，`StringBuffer`是线程安全的，而`StringBuilder`是非线程安全的，但是`StringBuilder`的性能高于`StringBuffer`，所以在单线程下推荐使用`StringBuilder`，多线程下推荐使用`StringBuffer`。

- 总而言之
  1. 可变性
      String 不可变
      StringBuffer 和 StringBuilder 可变
  2. 线程安全
      String 不可变，因此是线程安全的
      StringBuilder 不是线程安全的
      StringBuffer 是线程安全的，内部使用 synchronized 进行同步

## String str = "i" 与 String str = new String("i")

两者不同，`String str = "i"`， JVM将其分配到常量池中，`String str = new String("i")`，JVM将其分配到堆内存中。

## String Pool 字符串常量池

字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程中将字符串添加到 String Pool 中。

当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同字符串，而 s3 和 s4　是通过 s1.intern() 方法取得一个字符串引用。intern() 首先把 s1 引用的字符串放到　String Pool 中，然后返回这个字符串引用。因此 s3 和 s4 引用的是同一个字符串。

```java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
String s4 = s1.intern();
System.out.println(s3 == s4);           // true
```

如果是采用 "bbb" 这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。

```java
String s5 = "bbb";
String s6 = "bbb";
System.out.println(s5 == s6);  // true
```

## 字符串反转

使用 StringBuffer 或者 StringBuilder 的 `reverse()` 方法

```java
// StringBuffer reverse
StringBuffer stringBuffer = new StringBuffer();
stringBuffer.append("abcdefg");
System.out.println(stringBuffer.reverse());  //gfedcba

// StringBuilder reverse
StringBuilder stringBuilder = new StringBuilder();
stringBuilder.append("abcdefg");
System.out.println(stringBuilder.reverse());  //gfedcba
```

## String 类的常用方法

```java
indexOf()		// 返回指定字符的索引
charAt()		// 返回指定索引处的字符
replace()		// 字符串替换
trim()			// 去除字符串两端空白
split()			// 分割字符串，返回一个分割后的字符串
getBytes()		// 返回字符串的byte类型数组
length()		// 返回字符串长度
toLowCase()		// 将字符串转换为小写
toUpperCase()		// 将字符串转换成大写
equals()		// 字符串比较
```

## == 和 equals() 的区别

```java
String str1 = "str";
String str2 = "str";
String str3 = new String("str");
System.out.println(str1 == str2); // true
System.out.println(str1 == str3); // false
System.out.println(str1.euqals(str2)); // true
System.out.println(str1.equals(str3)); // true
```

因为str1 和 str2 指向的是同一个引用，所以 == 为 true，而 new String() 方法开辟了新的内存空间，所以 == 为 false，而 equals 比较的一直是值，所以结果都为 true.

**1. ==**
对于基本类型和引用类型，==的作用效果不同

- 基本类型：比较值是否相同
- 引用类型：比较引用是否相同

**2. equals()**

equals() 本质上就是 == ，只不过 String 和 Integer 等重写了 equals() 方法，把他变
成了值比较。

```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```
String 重写了 Object 的 equals 方法

```java
public boolean equals(Object anObject) {
	if (this == anObject) {
		return true;
	}
	if (anObject instanceof String) {
		String anotherString = (String)anObject;
		int n = value.length;
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

**3. 总结**

- == 
	- 基本类型：值比较
	- 引用类型：引用比较
- equals()
  - 默认是引用比较
  - 很多类重写 equals() 方法为值比较，比如String，Integer 等，一般情况下是值比较
