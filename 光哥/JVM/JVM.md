## JVM

请你谈谈对JVM的理解？jdk8虚拟机和之前的变化更新

什么是OOM，什么是栈溢出（stackOverFlow）怎么分析

JVM的常用调优参数

对类加载器的认知



## 2.JVM体系结构

## 3.类加载器



- 类加载指将类的class的字节码读取到内存中，并将其放在**运行时数据区的方法区**内，同时在**堆区创建一个Class类型的对象**，这个Class对象就是类加载的终极目的。
- 类加载的时机-遇到 new（比如new Student()）、getstatic 和 putstatic（读取或设置一个类的静态字段，如下代码，读取被 final修饰并已在编译器把结果放入常量池的静态字段除外）、invokestatic（调用类的静态方法）这四条指令时，如果对应的类没有初始化，则要对对应的类先进行初始化。
  

~~~java
Class.forName();
类.class;
对象.getClass()
~~~



- ![4](IMGS\4.png)
  
  加载 ：通过io读入字节码文件
  “加载”过程主要是靠类加载器实现的，包括用户自定义类加载器。类加载过程的一个阶段，ClassLoader通过一个类的完全限定名查找此类字节码文件，并利用字节码文件创建一个class对象。
  
  ~~~java
  1、通过一个类的全限定名来获取定义此类的二进制字节流（class文件）。在程序运行过程中，当要访问一个类时，若发现这个类尚未被加载，并满足类初始化的条件时，就根据要被初始化的这个类的全限定名找到该类的二进制字节流，开始加载过程
  2、将这个字节流的静态存储结构转化为方法区的运行时数据结构（即Class对象）
  3、在内存中创建一个该类的java.lang.Class对象，作为方法区该类的各种数据的访问入口
  
  ~~~
  
  
  
  验证：
  目的在于确保class文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身的安全，主要包括四种验证：文件格式的验证，元数据的验证，字节码验证，符号引用验证。
  
  class文件用16进制打开后，前八位为ca fe ba ba（magic number）后面16为是jdk的版本号
  
  
  
  
  
  准备：为[static](https://so.csdn.net/so/search?q=static&spm=1001.2101.3001.7020)分配内存并初始化0值。JDK1.7之前在方法区，1.7之后在堆。
  
  为类变量（static修饰的字段变量）分配内存并且设置该类变量的初始值，（如static int i = 5 这里只是将 i 赋值为0，在初始化的阶段再把 i 赋值为5)，这里不包含final修饰的static ，因为final在编译的时候就已经分配了。这里不会为实例变量分配初始化，类变量会分配在方法区中，实例变量会随着对象分配到Java堆中。
  
  
  
  解析：这里主要的任务是把常量池中的符号引用替换成直接引用
  
  
  
  初始化：初始化过程就是调用类初始化方法的过程，完成对static修饰的类变量的手动赋值还有主动调用静态代码块。
  
  这里是类加载的最后阶段，如果该类具有父类就进行对父类进行初始化，执行其静态初始化器（静态代码块）和静态初始化成员变量。（前面已经对static 初始化了默认值，这里我们对它进行赋值，成员变量也将被初始化）
  
  ~~~java
  类记载器的任务是根据类的全限定名来读取此类的二进制字节流到 JVM 中，然后转换成一个与目标类对象的java.lang.Class 对象的实例，在java 虚拟机提供三种类加载器：
  
  1. 启动类加载器(BootStrapClassLoader)-加载支撑JVM运行的位于JAVA_HOME\lib核心类库，比如rt.jar等
  2. 扩展类加载器(ExtClassLoader)-加载JAVA_HOME\lib\ext扩展类库
  3. 应用程序类加载器(AppClassLoader)-加载classpath下的类，主要加载你自己的写的类
  
  4. 自定义加载器-加载用户自定义路径下的类(tomcat)
  ~~~
  
  

## 4.双亲委派机制

~~~java
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }
    //              -----??-----
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
            // 首先，检查是否已经被类加载器加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    // 存在父加载器，递归的交由父加载器
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        // 直到最上面的Bootstrap类加载器
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }
 
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            return c;
    }
~~~



![2](IMGS\2.jpg)



从上图中我们就更容易理解了，当一个Hello.class这样的文件要被加载时。不考虑我们自定义类加载器，首先会在AppClassLoader中检查是否加载过，如果有那就无需再加载了。如果没有，那么会拿到父加载器，然后调用父加载器的loadClass方法。父类中同理也会先检查自己是否已经加载过，如果没有再往上。注意这个类似递归的过程，直到到达Bootstrap classLoader之前，都是在检查是否加载过，并不会选择自己去加载。直到BootstrapClassLoader，已经没有父加载器了，这时候开始考虑自己是否能加载了，如果自己无法加载，会下沉到子加载器去加载，一直到最底层，如果没有任何加载器能加载，就会抛出ClassNotFoundException。





## 5.沙箱安全机制

为什么要设计这种机制
这种设计有个好处是，如果有人想替换系统级别的类：String.java。篡改它的实现，在这种机制下这些系统的类已经被Bootstrap classLoader加载过了（为什么？因为当一个类需要加载的时候，最先去尝试加载的就是BootstrapClassLoader），所以其他类加载器并没有机会再去加载，从一定程度上防止了危险代码的植入



<img src="IMGS\1.jpg" alt="1" style="zoom:150%;" />





## 6.native



通过本地方法接口（Java Native Interface  JNI），调用跟本地方法库(存放通过C或C++操作CPU\内存等方法)，本地方法库中的方法，可以是C，C++,python等其他语言的方法，方法运行产生的数据会保存在**本地方法栈**中。



## 7.PC寄存器（程序计数器）

在JVM规范中，每个线程都有它自己的程序计数器，是线程私有的，生命周期与线程的生命周期保持一致。

任何时间一个线程都只有一个方法在执行，也就是所谓的当前方法。程序计数器会存储当前线程正在执行的Java方法的JVM指令地址；或者，如果是在执行native方法，则是未指定值（undefned）。

它是程序控制流的指示器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。



## 8.方法区

属于共享内存区域，存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。



运行时常量池：属于方法区一部分，用于存放编译期生成的各种字面量和符号引用。编译器和运行期(String 的 intern() )都可以将常量放入池中。内存有限，无法申请时抛出 OutOfMemoryError。



直接内存：非虚拟机运行时数据区的部分

OutOfMemoryError：会受到本机内存限制，如果内存区域总和大于物理内存限制从而导致动态扩展时出现该异常

## 9.三种Jvm

## 10.堆栈，新生代，老年代，永久区

**虚拟机栈**：每一个正在执行的方法，都会在虚拟机栈中被分配一块空间，称之为**栈帧**。每个线程在虚拟机栈都有一个独立的空间。也就是说，虚拟机栈是线程独有（**虚拟机栈，本地方法栈，程序计数器都是线程独有，堆，方法区是线程共享**），使用-Xss进行修改，默认是1m

```
-Xss1m
-Xss1024k
-Xss1048576
```





栈帧：局部变量表，操作数栈，动态链接，返回出口、附加信息





## 