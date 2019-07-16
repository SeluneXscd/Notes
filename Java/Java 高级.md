# Java 高级

## Java反射-reflect

反射是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；

对于任意一个对象，都能够调用它的任意一个方法和属性；

这种动态获取的信息以及动态调用对象的方法的功能称为 Java 语言的反射机制。

### Class 类

#### 在面向对象的世界里，万物皆对象

  - 类是对象，类是java.lang.Class 类的实例对象
  - There is a class named Class

#### 任何一个类都是Class的实例对象，这个实例对象有三种表示方式

  - 第一种表示方式：任何一个类，都有一个隐含的静态成员变量class

  ```java
  Class c1 = Foo.class;
  ```

  - 第二种表示方式：getClass方法

  ```java
  Class c2 = foo1.getClass();
  ```

  - c1, c2 都表示了Foo类的类类型(class type)，类也是对象，是Class类的实例对象，这个对象我们称之为该类的类类型
  - 不管c1还是c2都表示了Foo类的类类型，一个类只可能是Class类的一个实例对象

  ```java
  System.sout.println(c1 == c2); // true
  ```

  - 第三种表示方式：Class.forName

  ```java
  Class c3 = null;
  try {
      c3 = Class.forName("org.reflect.Foo");
  } catch(ClassNotFoundException e) {
      e.printStackTrace();
  }
  
  System.sout.println(c2 == c3);  // true
  ```

  - 可以通过类的类类型创建该类的类类型，即通过c1，c2，c3创建Foo的实例，还需要强制类型转换

  ```java
  try {
      if (c3 != null) {
          Foo foo1 = (Foo) c3.newInstance();
      }
  } catch (InstantiationException | IllegalAccessException e) {
      e.printStackTrace();
  }
  ```

#### Class.forName("类的全称")

  - 类的全称不仅表示类的类类型，还代表了动态加载类
  - 编译时刻加载类是静态加载类
  - 运行时刻加载类是动态加载类
  - new，创建对象，是静态加载类，编译时刻就需要加载所有可能用到的类，若类不存在，就报错；

```java
public class Office {
    public static void main(String[] args) {
        // Word类和Excel类不存在，编译时就报错
        if ("Word".equals(args[0])) {
            Word w = new Word();
            w.start();
        }
        if ("Excel".equals(args[0])) {
            Excel e = new Excel();
            e.start();
        }
    }
}
```

  - 可以使用动态加载类解决这个问题，运行时才管存不存在

```java
public class OfferBetter {
    public static void main(String[] args) {
        try {
            // 动态加载类，在运行是加载
            Class c = Class.forName(args[0]);
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

- 通过类类型，创建该类的实例对象

```java
public class OfficeBetter {
    public static void main(String[] args) {
        try {
            // 动态加载类，在运行是加载
            Class c = Class.forName(args[0]);
            // 通过类类型，创建该类对象
            OfficeAble oa = (OfficeAble) c.newInstance();
            oa.start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

- 创建一个接口，只要类实现OfficeAble，那么就可以创建对应的类

```java
public interface OfficeAble {
    public void start();
}
```

```shell
> javac Word.java
> java OfficeBetter Word
word...start...
```

```java
public class Word implements OfficeAble {
    public void start() {
        System.out.println("word...starts...");
    }
}
```

```shell
> javac Excel.java
> java OfficeBetter Word
excel...start...
```

```java
public class Excel implements OfficeAble {
    public void start() {
        System.out.println("excel...starts...");
    }
}
```

- 途中，OfficeBetter.java不需要重新编译
- 功能性的类，尽量使用动态加载，而不是使用静态加载

#### 获取类的方法信息

```java
Class c1 = int.class;
System.out.println(c1.getName());  // int
```

```java
/**
 * 打印类的信息，包括类的成员函数，成员变量
 * @param object
 */
public static void printClassMessage(Object object) {
    System.out.println(               "+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++");
    // 获取类的信息，首先获取类的类类型
    Class c = object.getClass();
    // 获取类的名称
    System.out.println("class Name= " + c.getName());
    /*
     * Method类，方法对象
     * 一个成员方法就是一个Method对象
     * getMethods() 方法，获取的是所有的public的函数，包括父类继承的而来的方法，native本地方法，java声明，c/c++实现，java调用
     * getDeclaredMethods() 方法，获取的是所有该类自己声明的方法，不问访问权限，没有父类继承而来的
     */
    Method[] ms = c.getMethods();
    Method[] declaredMethods = c.getDeclaredMethods();

    for (int i = 0; i < ms.length; i++) {
        // 得到返回值类型的类类型
        Class returnType = ms[i].getReturnType();
        System.out.print(returnType.getName() + "= ");
        // 得到方法名
        System.out.print(ms[i].getName() + "{ ");
        // 获取参数类型，得到的是参数列表的类型的类类型
        Class[] paramTypes = ms[i].getParameterTypes();
        for (Class paramType : paramTypes) {
            System.out.print(paramType.getName() + ", ");
        }
        System.out.print("} \n");
    }
}
```

#### 获取类的成员变量的信息

```java
public static void printFieldMessage(Object object) {

    Class c = object.getClass();
    /*
     *成员变量也是对象
     * java.lang.reflect.Field
     * Field类封装了关于成员变量的操作
     * getFields() 方法获取的是所有public的成员变量的信息
     * getDeclaredFields() 方法过去的是该类自己声明的成员变量的信息
     */
    // Field[] fs = c.getFields();
    Field[] fs = c.getDeclaredFields();
    for (Field field : fs) {
        // 得到成员变量的类型的类类型
        Class fieldType = field.getType();
        String typeName = fieldType.getName();
        // 得到成员变量的名称
        String fieldName = field.getName();
        System.out.println("typeName= " + fieldName);
    }
}
```

#### 获取类的构造函数的信息

```java
/**
 * 打印对象的构造函数
 * @param object
 */
public static void printConMessage(Object object) {
    Class c = object.getClass();
    /*
     * 构造函数也是对象
     * java.lang.reflect.Constructor 封装了构造函数的信息
     * getConstructors() 获取了所有的public的构造函数
     * getDeclaredConstructors() 获取了所有的自己声明的构造函数
     */
    // Constructor[] cs = c.getConstructors();
    Constructor[] cs = c.getDeclaredConstructors();
    for (Constructor constructor : cs) {
        System.out.print(constructor.getName() + "{ ");
        // 获取构造函数的参数列表，得到的是参数列表的类类型
        Class[] paramTypes = constructor.getParameterTypes();
        for (Class paramType : paramTypes) {
            System.out.print(paramType.getName() + ", ");
        }
        System.out.println("}");
    }
```

### 反射

#### 方法的反射

- 如何获取某个方法

  方法的名称和方法的参数列表才能唯一确定某个方法

- 方法反射的操作

  method.invoke(对象, 参数列表)

```java
public class MethodDemo1 {

    public static void main(String[] args) {
        /*
         * 获取print(int, int) 方法
         * 1. 获取一个方法就是获取类的信息，获取类的信息，先要获取类的类类型
         */
        A a1 = new A();
        Class c = a1.getClass();
        /*
         * 2. 获取方法，名称，参数列表确定
         * getMethod获取的是public方法
         * getDeclaredMethod获取的是自己声明的方法
         */
        try {
            Method m = c.getMethod("print", int.class, int.class);
            // 方法的反射操作
            // 用m对象来进行方法调用
            // 方法没有返回值，返回null
            // 有返回值，返回具体的返回值
            Object object = null;
            object = m.invoke(a1, 10, 20);
            System.out.println("+++++++++++++++++++++++++++++++++++++++");
            
            Method m1 = c.getMethod("print", String.class, String.class);
            object = m1.invoke(a1, "x", "Y");
            System.out.println("+++++++++++++++++++++++++++++++++++++++");
            
            Method m2 = c.getMethod("print");
            object = m2.invoke(a1);
            
        } catch (NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}

class A {

    public void print() {
        System.out.println("Hello");
    }

    public void print(int a, int b) {
        System.out.println(a + b);
    }

    public void print(String a, String b) {
        System.out.println(a.toUpperCase() + ", " + b.toLowerCase());
    }
}
```

#### 通过反射了解集合泛型的本质

- 反射的操作都是编译之后的操作；

  c1 == c2 为true说明编译之后集合的泛型是去泛型化的

  Java中集合的泛型，是防止错误输入的，只在编译阶段有效，绕过编译就无效了

```java
ArrayList list = new ArrayList();
ArrayList<String> list1 = new ArrayList<String>();

list1.add("Hello");
// list1.add(10); // 错误的

Class c1 = list.getClass();
Class c2 = list1.getClass();
System.out.println(c1 == c2);  // true
```

​		验证：可以通过方法的反射来操作，来绕过编译

​		不能使用foreach遍历，int不能转换成String

```java
try {
    
    Method m = c2.getMethod("add", Object.class);
    m.invoke(list1, 100);  // 绕过编译操作，就染过了泛型
    System.out.println(list1.size());
    System.out.println(list1);
    
} catch (Exception e) {
    e.printStackTrace();
}
```

