## 类加载机制是什么？

在 Java 中，类加载机制是 Java 虚拟机（JVM）将 `.class` 文件加载到内存并转化为可以运行的 Class 对象的过程。简单来说，类加载机制是让“代码变为现实”的第一步！

你可能会问，**为什么需要类加载机制？** 因为 Java 是一门 **动态语言**，类可以在运行时加载、链接和初始化，这种灵活性让 Java 能够实现跨平台运行、高效的内存管理和模块化架构。

## 类加载的三个阶段

根据《Java 虚拟机规范》，类的生命周期包括以下三个主要阶段：**加载**、**链接** 和 **初始化**。

而其中链接又分为三个子阶段：验证（Verification）、准备（Preparation）、解析（Resolution）。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202412012155784.png)

我们逐一拆解这些阶段的工作原理和流程。

### 加载（Loading）

> Chaya：类加载阶段作用是什么？非要加载吗？

主要是使用 "类加载器" 将本地或者远程网络中的字节码文件，通过读字节流的方式加载到 Java 虚拟机内存中。在加载阶段中 Java 虚拟机主要完成以下三件事情:

- **①** 通过一个类的全限定名称来获取定义此类的二进制字节流。
- **②** 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
- **③** 在内存中生成一个代表这个类的 `java.lang.Class` 对象，作为方法区中这个类的各种数据的访问入口。

**加载**是类加载的第一步，JVM 需要完成以下任务：

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202412012224688.png)

1. **读取 Class 文件**：通过类的全限定名找到对应的 `.class` 文件。
2. **转换为 JVM 可识别的结构**：将 Class 文件的二进制数据转换为 JVM 的运行时数据结构。
3. **创建 Class 对象**：在内存中创建 `java.lang.Class` 对象，作为该类的入口。

示例。

```java
Class<?> clazz = Class.forName("com.example.MyClass");

```

这段代码会触发 `MyClass` 的加载，将其 `.class` 文件读取到内存中，并生成 `Class` 对象。

## 链接（Linking）

**链接** 是将 Class 文件中的符号引用解析为直接引用的过程，分为以下三个子阶段：

1. **验证（Verification）**
   确保 Class 文件的字节码格式和内容符合 JVM 的规范。
   - **验证文件格式**：Class 文件是否以 `0xCAFEBABE` 开头。
   - **验证字节码**：指令是否符合 JVM 规范，数据类型是否匹配。
2. **准备（Preparation）**
   为类的静态变量分配内存，并设置默认值。
   - 例如：`static int a = 10;` 在准备阶段，`a` 的初始值是 `0`。
3. **解析（Resolution）**
   将符号引用替换为内存地址的直接引用。
   - **符号引用**：`java.lang.String`
   - **直接引用**：指向 `String` 类在内存中的地址。

### 验证阶段 (Verification)

验证阶段的主要目的是对字节码字节流进行校验，判断其内容是否符合当前虚拟机的规范，以确保被加载的代码运行后不会对虚拟机造成损害。

大多数虚拟机大致都会对 `文件格式`、`元数据`、`字节码`、`符号引用` 几项内容进行校验。

**文件格式验证**

文件格式验证主要是对 `字节流格式` 进行校验，判断其是否符合字节码文件格式规范，并且还要判断其是否可以运行在当前版本的虚拟机中。比如:

| 序号 |                             描述                             |
| :--: | :----------------------------------------------------------: |
|  1   |                  验证是否以 0XCAFEBABE 开头                  |
|  2   |    验证主、次版本号，是否包含在当前虚拟机支持的版本范围内    |
|  3   |      验证字节码常量池中的常量类型，是否都被虚拟机所支持      |
|  4   | 验证指向常量的各种索引值，是否有指向不存在的常量或不符合类型的常量 |
|  5   | 验证 CONSTANT_Utf8_info 类型常量中，是否有不符合 UTF-8 编码的数据 |
|  6   | 验证字节码文件中各个部分及文件本身，是否有被删除或附加的其他信息 |

文件格式验证的主要目的其实就是为了保证加载的字节码可以被正确地解析并存储在方法区内。

**元数据验证**

元数据验证主要是对 `字节码` 中的 `元数据信息` 进行语法校验，避免存在不符合 Java 语法规范的元数据信息。比如:

| 序号 |                             描述                             |
| :--: | :----------------------------------------------------------: |
|  1   | 验证当前类的父类是否继承了不允许被继承的类，比如被 final 修饰的类 |
|  2   | 验证当前类是否有父类，一般情况下除了 java.lang.Object 外，所有的类都应当有父类 |
|  3   | 验证如果当前类不是抽象类，则当前类是否实现了其父类或接口之中要求实现的所有方法 |
|  4   | 验证当前类中的字段或方法是否与父类有冲突，比如当前类覆盖了父类的 final 字段，或者当前类实现的方法参数都一致，但返回值的类型却不同，导致不符合方法重载规则等情况 |

**字节码验证**

字节码验证主要是对 `数据流` 和 `控制流` 进行分析，以确保其语法合规且符合逻辑。

**符号引用验证**

符号引用验证主要对 `字节码常量池` 中 `常量` 的各种 `符号引用` 进行校验，确保当前类引用到的其它类或者方法是真实存在且有权限访问的。如果符号引用中关联的类无法在系统中查找到，就会抛出 `NoClassDefFoundError` 错误，如果符号引用中关联的方法无法找到，则会抛出 `NoSuchMethodError` 错误。

### 准备阶段 (Preparation)

准备阶段主要是用于对类或接口中的 "静态变量" 分配内存空间，以及对变量设置默认的初始值。

准备阶段和初始化阶段，这两个阶段都是用于对静态变量设置值，概念上容易混淆，所以这里需要特别说明一下，准备阶段只是对静态变量设置初始默认值，而真正赋值操作是在初始化阶段完成的。

例如，下面示例代码在执行时:

```java
public class A {
    static int test = 999; 
}
```

- 准备阶段会对变量 **test** 设置默认值 `0`；
- 初始化阶段会对变量 **test** 赋予初始值 `999`；

### 解析阶段 (Resolution)

解析阶段主要是用于将 `字节码常量池` 中的 `符号引用` 替换为 `直接引用` 的过程。

- **符号引用 (Symbolic References):** 符号引用就是用于描述引用目标的一组符号，它可以是任何形式的字面量 (只要符合 Java 虚拟机规范)。
- **直接引用 (Direct References):** 直接引用可以是直接指向目标的指针、相对偏移量，或者是一个能间接定位到目标的句柄。

## 初始化（Initialization）

1. **初始化阶段**是类加载的最后一步，也是最重要的阶段。此阶段会执行静态变量的赋值操作和静态代码块。

   **初始化的触发条件**：

   1. **使用 `new` 关键字实例化对象时**。
   2. **访问类的静态字段或静态方法时**。
   3. **使用反射调用类时**。

   **类的初始化顺序**

   1. **先初始化父类**。
   2. **再初始化当前类的静态变量和静态代码块**。

> 唐二婷：初始化阶段有啥用？可以谈恋爱吗？

初始化阶段主要是执行 `类构造器` 方法 `<clinit>()`，该方法不需要定义，代码在经过 Javac 编译器编译时，会自动收集类中的所有 `类变量` 的赋值动作和 `静态代码块` 中的语句，对这些代码进行合并，形成类构造器 `<clinit>()` 。

在执行类构造器 `<clinit>()` 时，会对类中的 `类变量` 和 `静态代码块` 进行初始化赋值操作，如果该类存在父类，则会先执行父类中的类构造器 `<clinit>()`，对父类中的 `类变量` 和 `静态代码块` 进行初始化。

示例如下。

```java
public class FatherCLass {

    public static int number;

    static {
        System.out.println(number);
        System.out.println("父类 static{} 初始化");
    }

}
```

子类:

```java
public class SubInitialization extends FatherCLass {

    static{
        // number 属于父类的属性，这里要能执行成功，说明父类已经加载
        number = 100;
        System.out.println("子类 static{} 初始化");
    }

    public static void main(String[] args) {
        System.out.println(number);
    }

}
```

执行时输出如下:

```bash
0
父类 static{} 初始化
子类 static{} 初始化
100
```

## 类加载器分类

在 Java 中，类的初始化分为几个阶段: `加载`、`链接`（包括验证、准备和解析）和 `初始化`。

而 `类加载器`（Class Loader）则是加载阶段中，负责将本地或网络中的指定类的二进制流，加载到 Java 虚拟机中的工具。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202412012246584.png)

### 引导类加载器 BootstrapClassLoader

引导类加载器 BootstrapClassLoader：引导类加载器是使用 C++ 语言实现的，嵌入在 JVM 中。用于加载 Java 中的核心类库的，不继承自 `java.lang.ClassLoader`，在 Java 程序中通常返回 `null`。

一般会加载 `JAVA_HOME` 目录下的 `/jre/lib` 文件夹下的 jar 和配置。

```java
ClassLoader loader = String.class.getClassLoader();
System.out.println(loader); // 输出 null，因为 String 是由引导类加载器加载的

```



### 扩展类加载器 ExtClassLoader

扩展类加载器主要负责加载 Java 的扩展类库，一般会加载 `JAVA_HOME` 目录下的 `/jre/lib/ext` 文件夹下的 jar。

继承自 `java.lang.ClassLoader`，是用户可以访问的第一个类加载器。

```java
ClassLoader extLoader = ClassLoader.getSystemClassLoader().getParent();
System.out.println(extLoader); // 输出 sun.misc.Launcher$ExtClassLoader

```



###应用类加载器（Application ClassLoader）

应用类加载器是应用程序中默认的类加载器，可以加载 `CLASSPATH` 变量指定目录下的 jar，由 `sun.misc.Launcher$AppClassLoader` 实现。

并且一般情况下，我们编写的 Java 应用的类，都是使用该类加载器完成加载的。

```java
ClassLoader appLoader = ClassLoader.getSystemClassLoader();
System.out.println(appLoader); // 输出 sun.misc.Launcher$AppClassLoader

```

### 类加载器抽象类 `ClassLoader`

在 Java 中存在一个类加载器抽象类 `ClassLoader`，大多数类加载器都是通过继承这个类来实现的类加载功能。以下是 ClassLoader 类的关键部分代码:

```java
public abstract class ClassLoader {

    /*
     * 类加载器的父加载器
     */
    private final ClassLoader parent;

    /**
     * 根据类的全限定名加载类
     * 
     * @param name 类名称 
     * @return     加载的Class对象
     * @throws ClassNotFoundException 没有发现指定类异常
     */
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        // 调用loadClass方法加载类，其中设置resolve=false，表示不立即解析类
        return loadClass(name, false);
    }

    /**
     * 根据类的全限定名加载类
     * 
     * @param name    类名称
     * @param resolve 是否解析这个类，true=解析，false=不解析 
     * @return 加载的Class对象
     * @throws ClassNotFoundException 没有发现指定类异常
     */
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // 检查类是否已经被加载
            Class<?> c = findLoadedClass(name);
            // 如果没有加载过
            if (c == null) {
                // 如果有父类加载器，则委托给父加载器去加载
                // 如果没有父类加载器，则判断 Bootstrap 类加载器是否加载过
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
                // 如果父类加载器都加载失败，则当前类加载器尝试自行加载
                if (c == null) {
                    c = findClass(name);
                }
            }
            // 据 resolve 参数决定是否解析类
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

    /**
     * 查找并加载指定名称的类
     * 
     * @param name 类名称 
     * @return Class对象
     * @throws ClassNotFoundException 没有发现指定类异常
     */
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        //1. 根据传入的类名，到在特定目录下去寻找类文件，把字节码文件读入内存
        // ...
        //2. 调用 defineClass 将字节数组转成 Class 对象
        return defineClass(buf, off, len)；
    }

    /**
     * 将一个 byte[] 转换为 Class 类的实例
     * 
     * @param name 类名称，如果不知道此名称，则该参数为 null
     * @param b    组成类数据的字节数组
     * @param off  类数据的起始偏移量
     * @param len  类数据的长度 
     * @return Class对象
     * @throws ClassFormatError 类格式化异常
     */
    protected final Class<?> defineClass(byte[] b, int off, int len) throws ClassFormatError {
        ...
    }

}

```

**类中定义的常用的类加载相关的方法:**

|                       方法名称                       |                          描述                           |
| :--------------------------------------------------: | :-----------------------------------------------------: |
|                     getParent()                      |               返回该类加载器的父类加载器                |
|                loadClass(String name)                |       加载指定名称的类，返回 java.lang.Class 实例       |
|                findClass(String name)                |       查找指定名称的类，返回 java.lang.Class 实例       |
|             findLoadedClass(String name)             |   查找已加载的指定名称的类，返回 java.lang.Class 实例   |
| defineClass(String name, byte[] b, int off, int len) | 将字节数组转换为一个 Java 类，返回 java.lang.Class 实例 |
|                resolveClass(Class c)                 |                   连接指定的 Java 类                    |

## 双亲委派模型（Parent Delegation Model）

**双亲委派模型** 是类加载器的设计模式，其核心思想是：**类加载请求由子类加载器向父类加载器逐层委派，直到引导类加载器。** 

如果父类加载器无法加载，子类加载器才会尝试加载。

如果子类加载器也无法加载该类，就会抛出一个 `ClassNotFoundException` 异常。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/redis/202412012256145.png)

### 双亲委派机制的作用

我们试想一下，如果不使用这种委托模式，那我们就可以随时使用自定义的 String 类来动态替代 Java 核心 API 中定义的类型，这样会存在非常大的安全隐患。

而双亲委托的方式，就可以避免这种情况，因为 String 已经在启动时就被引导类加载器 (BootstrcpClassLoader) 加载，所以用户自定义的 ClassLoader 永远也无法加载一个用户自己自定义的 String 类，除非你改变 JDK 中 ClassLoader 搜索类的默认算法。

该机制的作用如下。

- 防止重复加载字节码文件: 将类加载请求先委托给父类，父类加载后子类就不会重复加载该类。所以，双亲委派机制可以防止对某个类重复加载；
- 防止核心字节码文件被篡改: 一般情况下引导类加载器会先加载 JVM 核心类库，然后其它加载器才会执行，如果其它加载器要加载一个被篡改的核心字节码文件，会将该文件委托给父类加载器，当委托到引导类加载器时，加载器已经加载过该类，就不会对该类进行重复加载。而且就算能被加载，那么加载它的肯定不是相同的类加载器 (不会是引导类加载器)，Java 虚拟机中只认可核心类加载器加载的核心类库，所以，双亲委派机制可以防止核心字节码文件被篡改。
- 简化加载逻辑: 通过委派模式，每个类加载器只需要关注自己负责的那部分类加载逻辑，而不必关心其他类加载器的加载细节，简化了类加载器的实现，降低了系统的复杂度。

## 自定义类加载器

在某些场景下，标准的类加载器无法满足需求，例如：

1. **热部署**：在 Web 服务器中动态加载或更新类。
2. **模块隔离**：在同一个 JVM 中加载不同版本的类。
3. **加密解密**：加载经过加密的 Class 文件。

默认的类加载器只能加载指定目录下的 Jar 和 Class 文件。

如果需要加载指定位置的类文件并实现一些自定义逻辑，就需要自定义类加载器。

> Chaya：如何实现自定义类加载器？

**步骤**：

1. 继承 `java.lang.ClassLoader` 类。
2. 重写 `findClass()` 方法，通过字节流读取 Class 文件并转换为 `Class` 对象。

```java
import java.io.*;

public class MyClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = loadClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        }
        return defineClass(name, classData, 0, classData.length);
    }

    private byte[] loadClassData(String name) {
        String fileName = name.replace('.', '/') + ".class";
        try (InputStream is = new FileInputStream(fileName);
             ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            int buffer;
            while ((buffer = is.read()) != -1) {
                baos.write(buffer);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }
}

```

**示例说明**

- `findClass()`：从文件系统加载 Class 文件，并将其定义为 `Class` 对象。
- `defineClass()`：将字节数组转换为 JVM 可执行的 `Class` 对象。

为了为保证类加载器都正确实现双亲委派机制，在开发自己的类加载器时，只需要重写 `findClass()` 方法即可。

当然，如果不想使用双亲委派机制时，就需要重写 `loadClass()` 方法。

### 打破双亲委派模型

有时为了实现特殊功能，我们需要**打破双亲委派模型**，例如：

1. **热部署框架**：Tomcat、Spring Boot 使用自定义类加载器加载和卸载 Web 应用。
2. **SPI（Service Provider Interface）机制**：JDBC 驱动等需要通过 `线程上下文类加载器` 来加载用户实现的接口。

## 类加载的时机与触发条件

Java 虚拟机（JVM）中，类的加载并不是随意发生的，而是由特定的**触发条件**决定的。**什么时候加载？什么时候初始化？** 

这是我们必须要搞清楚的问题，尤其在复杂的应用中，弄懂类加载的时机能帮助我们避免一些潜在的性能问题和运行时错误。

在本节中，我们将详细探讨类加载的时机、主动和被动引用的区别，以及常见的类加载触发条件。

### 类加载生命周期

类加载的生命周期包括：**加载（Loading）**、**链接（Linking）** 和 **初始化（Initialization）**。而其中，**初始化阶段**是决定类是否被真正加载的关键。

JVM 在什么时候启动类加载过程呢？主要分为**主动引用**和**被动引用**两种情况。

### 主动引用：触发类加载的条件

**主动引用**是指程序显式地使用某个类，从而触发类的加载和初始化。根据《Java 虚拟机规范》，以下六种情况会触发类的主动引用。

#### 创建类的实例

当你使用 `new` 关键字创建一个类的实例时，JVM 会立即加载并初始化该类。

```java
MyClass obj = new MyClass();  // 触发 MyClass 的加载和初始化

```

**初始化流程**：

1. 分配内存给 `MyClass` 的实例对象。
2. 加载 `MyClass` 类的字节码，并执行静态代码块和静态变量赋值操作。

#### 访问类的静态字段或静态方法

访问类的静态字段或静态方法时，也会触发类的加载和初始化。

```java
System.out.println(MyClass.staticVar);  // 触发 MyClass 的加载
MyClass.staticMethod();                // 触发 MyClass 的加载

```

**常量不会触发类加载**：如果静态字段是 `final` 修饰的常量，它在编译期已存入常量池，因此不会触发类加载。

```java
System.out.println(MyClass.FINAL_CONSTANT);  // 不触发类加载

```

#### **反射**

通过反射调用类时，也会触发类加载。

```java
Class<?> clazz = Class.forName("com.example.MyClass");  // 触发 MyClass 的加载

```

####  初始化类的子类时，先初始化父类

当初始化一个类时，如果它的父类尚未初始化，JVM 会先初始化父类。

```java
public class Parent {
    static {
        System.out.println("父类初始化");
    }
}

public class Child extends Parent {
    static {
        System.out.println("子类初始化");
    }
}

Child obj = new Child();  // 先输出"父类初始化"，再输出"子类初始化"

```

#### 拟机启动时，初始化 `main` 方法所在的类

虚拟机启动时，`main` 方法所在的类是程序的入口类，会被优先加载和初始化。

```java
public static void main(String[] args) {
    System.out.println("主类加载");
}

```

#### 动态语言支持

在 Java 7 引入的 `java.lang.invoke` 包中，当 `MethodHandle` 最终指向的类需要初始化时，也会触发类的加载。

```java
MethodHandle handle = MethodHandles.lookup().findStatic(MyClass.class, "staticMethod", MethodType.methodType(void.class));
handle.invoke();  // 可能触发 MyClass 的加载

```

### 被动引用：不会触发类加载的场景

与主动引用相对，**被动引用**是指访问类的某些特性时不会触发类的加载和初始化。以下是几种典型的被动引用场景。

#### 通过子类引用父类的静态字段

如果子类只引用父类的静态字段，JVM 只会初始化父类，而不会初始化子类。

**示例**

```java
System.out.println(Child.staticVar);  // 只触发 Parent 的加载，不触发 Child 的加载

```

#### 访问编译期常量

访问 `final` 修饰的编译期常量，不会触发类的加载。

```java
System.out.println(MyClass.FINAL_CONSTANT);  // 不触发 MyClass 的加载

```

#### 通过数组定义类引用

通过数组引用一个类，不会触发该类的加载。

```java
MyClass[] array = new MyClass[10];  // 不触发 MyClass 的加载

```

> 为什么需要关注类加载的时机？

1. **避免类的过早加载**：过早加载可能导致不必要的内存消耗，尤其在大型应用中。
2. **延迟加载（Lazy Loading）**：通过延迟加载，可以在真正需要时才加载类，减少启动时间。
3. **减少类加载冲突**：在模块化或插件化的应用中，合理安排类加载顺序有助于避免类冲突和类加载死锁问题。

