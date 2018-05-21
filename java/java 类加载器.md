# Java 类加载器 #

## 前言 ##

	public class ClassLoaderTest {
    
	    public static void main(String[] args) {
	        ClassLoader loader = Thread.currentThread().getContextClassLoader();
	        System.out.println(loader);
	        System.out.println(loader.getParent());
	        System.out.println(loader.getParent().getParent());
	    }

	}

运行后，输出结果：

	sun.misc.Launcher$AppClassLoader@18b4aac2
	sun.misc.Launcher$ExtClassLoader@255316f2
	null

从上面的结果可以看出，并没有获取到 ExtClassLoader的父Loader，原因是 BootstrapLoader（引导类加载器）是用C语言实现的，找不到一个确定的返回父Loader的方式，于是就返回null。

![](https://github.com/longtian2/cc3/blob/master/images/java-class-loader.png)

站在Java虚拟机的角度来讲，只存在两种不同的类加载器：

**启动类加载器**：它使用C++实现（这里仅限于Hotspot，也就是JDK1.5之后默认的虚拟机，有很多其他的虚拟机是用Java语言实现的），是虚拟机自身的一部分。

**所有其它的类加载器**：这些类加载器都由Java语言实现，独立于虚拟机之外，并且全部继承自抽象类 java.lang.ClassLoader，这些类加载器需要由启动类加载器加载到内存中之后才能去加载其他的类。

站在Java开发人员的角度来看，类加载器可以大致划分为以下三类：

**启动类加载器**： BootstrapClassLoader，负责加载存放在 JDK\jre\lib(JDK代表JDK的安装目录，下同)下，或被 -Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库（如rt.jar，所有的java.开头的类均被 BootstrapClassLoader加载）。启动类加载器是无法被Java程序直接引用的。

**扩展类加载器**： ExtensionClassLoader，该加载器由 sun.misc.Launcher$ExtClassLoader实现，它负责加载 JDK\jre\lib\ext目录中，或者由 java.ext.dirs系统变量指定的路径中的所有类库（如javax.开头的类），开发者可以直接使用扩展类加载器。

**应用程序类加载器**： ApplicationClassLoader，该类加载器由 sun.misc.Launcher$AppClassLoader来实现，它负责加载用户类路径（ClassPath）所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

应用程序都是由这三种类加载器互相配合进行加载的，如果有必要，我们还可以加入自定义的类加载器。因为JVM自带的ClassLoader只是懂得从本地文件系统加载标准的java class文件，因此如果编写了自己的ClassLoader，便可以做到如下几点：

1、在执行非置信代码之前，自动验证数字签名。

2、动态地创建符合用户特定需要的定制化构建类。

3、从特定的场所取得java class，例如数据库中和网络中。

## JVM类加载机制 ##

全盘负责，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入

父类委托，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类

缓存机制，缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效


类加载有三种方式：

**1、命令行启动应用时候由JVM初始化加载**

**2、通过Class.forName()方法动态加载**

**3、通过ClassLoader.loadClass()方法动态加载**


**Class.forName()和ClassLoader.loadClass()区别**

Class.forName()：除了将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块；

ClassLoader.loadClass()：只干一件事情，就是将.class文件加载到jvm中，不会执行static中的内容,只有在newInstance才会去执行static块。

Class.forName(name,initialize,loader)带参函数也可控制是否加载static块。并且只有调用了newInstance()方法采用调用构造函数，创建类的对象 。

## 双亲委派模型 ##

双亲委派模型的工作流程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

**双亲委派机制**:

1、当 AppClassLoader加载一个class时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器ExtClassLoader去完成。

2、当 ExtClassLoader加载一个class时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给BootStrapClassLoader```去完成。

3、如果 BootStrapClassLoader加载失败（例如在 $JAVA_HOME/jre/lib里未查找到该class），会使用 ExtClassLoader来尝试加载；

4、若ExtClassLoader也加载失败，则会使用 AppClassLoader来加载，如果 AppClassLoader也加载失败，则会报出异常 ClassNotFoundException。

双亲委派模型意义：

系统类防止内存中出现多份同样的字节码

保证Java程序安全稳定运行

**自定义类加载器**

通常情况下，我们都是直接使用系统类加载器。但是，有的时候，我们也需要自定义类加载器。比如应用是通过网络来传输 Java类的字节码，为保证安全性，这些字节码经过了加密处理，这时系统类加载器就无法对其进行加载，这样则需要自定义类加载器来实现。自定义类加载器一般都是继承自 ClassLoader类，从上面对 loadClass方法来分析来看，我们只需要重写 findClass 方法即可。下面我们通过一个示例来演示自定义类加载器的流程：

	public class MyClassLoader extends ClassLoader {
	    private String root;
	
	    public static void main(String[] args) {
	        MyClassLoader classLoader = new MyClassLoader();
	        classLoader.setRoot("E:\\temp");
	        Class<?> testClass = null;
	        try {
	            testClass = classLoader.loadClass("com.neo.classloader.Test2");
	            Object object = testClass.newInstance();
	            System.out.println(object.getClass().getClassLoader());
	        } catch (ClassNotFoundException e) {
	            e.printStackTrace();
	        } catch (InstantiationException e) {
	            e.printStackTrace();
	        } catch (IllegalAccessException e) {
	            e.printStackTrace();
	        }
	    }
	
	    protected Class<?> findClass(String name) throws ClassNotFoundException {
	
	        byte[] classData = loadClassData(name);
	
	        if (classData == null) {
	            throw new ClassNotFoundException();
	        } else {
	            return defineClass(name, classData, 0, classData.length);
	        }
	
	    }
	
	    private byte[] loadClassData(String className) {
	        String fileName = root + File.separatorChar + className.replace('.', File.separatorChar) + ".class";
	
	        try {
	            InputStream ins = new FileInputStream(fileName);
	            ByteArrayOutputStream baos = new ByteArrayOutputStream();
	            int bufferSize = 1024;
	            byte[] buffer = new byte[bufferSize];
	            int length = 0;
	
	            while ((length = ins.read(buffer)) != -1) {
	                baos.write(buffer, 0, length);
	            }
	
	            return baos.toByteArray();
	        } catch (IOException e) {
	            e.printStackTrace();
	
	        }
	
	        return null;
	
	    }
	
	    public String getRoot() {
	        return root;
	    }
	
	    public void setRoot(String root) {
	        this.root = root;
	
	    }
	
	}

自定义类加载器的核心在于对字节码文件的获取，如果是加密的字节码则需要在该类中对文件进行解密。由于这里只是演示，我并未对class文件进行加密，因此没有解密的过程。这里有几点需要注意：

1、这里传递的文件名需要是类的全限定性名称，即 com.paddx.test.classloading.Test格式的，因为 defineClass 方法是按这种格式进行处理的。

2、最好不要重写loadClass方法，因为这样容易破坏双亲委托模式。

3、这类Test 类本身可以被 AppClassLoader类加载，因此我们不能把 com/paddx/test/classloading/Test.class放在类路径下。否则，由于双亲委托机制的存在，会直接导致该类由 AppClassLoader加载，而不会通过我们自定义类加载器来加载。


## OSGi类加载流程 ##

OSGI 是当前业界“事实上”的 java 模块化标准，而 OSGI 实现模块化热部署的关键是它自定义的类加载器机制的实现。每个程序模块（OSGI 中称为 Bundle）都有一个自己的类加载器，当需要更换一个Bundle 时，就把 Bundle 连同类加载器一起换掉以实现代码的热替换。在 OSGI 环境下，类加载器不再是双亲委派模型中树状结构，而是进一步发展为网状结构。

1、OSGi为每个bundle提供一个类加载器，该加载器能够看到bundle Jar文件内部的类和资源

2、为了让bundle能互相协作，可以基于依赖关系，从一个bundle类加载器委托到另一个bundle类加载器

![](https://github.com/longtian2/cc3/blob/master/images/osgi-demo.png)

例如，bundleA、B都依赖于bundleC，当他们访问bundleC中的类时，就会委托给bundleC的类加载器，由它来查找类；如果它发现还要依赖bundleE中的类，就会再委托给bundleE的类加载器。

![](https://github.com/longtian2/cc3/blob/master/images/osgi-class-loader.png)

![](https://github.com/longtian2/cc3/blob/master/images/osgi-class-loader-2.png)

**Step 1: 检查是否java.*，或者在bootdelegation中定义**

当bundle类加载器需要加载一个类时，首先检查包名是否以java.*开头，或者是否在一个特定的配置文件（org.osgi.framework.bootdelegation）中定义。如果是，则bundle类加载器立即委托给父类加载器（通常是Application类加载器）。

这么做有两个原因：

唯一能够定义java.*包的类加载器是bootstrap类加载器，这个规则是JVM要求的。如果OSGI bundle类加载器试图加载这种类，则会抛Security Exception。

一些JVM错误地假设父加载器委托永远会发生，内部VM类就能够通过任何类加载器找到特定的其他内部类。所以OSGi提供了org.osgi.framework.bootdelegation属性，允许对特定的包（即那些内部VM类）使用父加载器委托。

**Step 2: 检查是否在Import-Package中声明**

检查是否在Import-Package中声明。如果是，则找到导出包的bundle，将类加载请求委托给该bundle的类加载器。如此往复。

**Step 3: 检查是否在Require-Bundle中声明**

检查是否在Require-Bundle中声明。如果是，则将类加载请求委托给required bundle的类加载器。

**Step 4: 检查是否bundle内部类**

检查是否是该bundle内部的类，即当前JAR文件中的类。

**Step5:  检查fragment**

搜索可能附加在当前bundle上的fragment中的内部类。

什么是fragment？

Fragment bundle是OSGi 4引入的概念，它是一种不完整的bundle，必须要附加到一个host bundle上才能工作；fragment能够为host bundle添加类或资源，在运行时，fragment中的类会合并到host bundle的内部classpath中。

fragment有什么作用？

【场景1】bundle中有针对特定平台的代码

假设bundle对不同平台的实现方式稍有不同，Windows和Linux下代码有不同之处，即bundle中有针对特定平台的代码。

我们应该为每个平台提供不同的bundle吗？——显然不能，因为那会造成代码重复。

或者将共同代码放到bundle A中，Windows特定的那部分代码放到bundle Pwin中，Linux特定的那部分代码放到bundle Plinux中。——有问题：Pwin肯定要依赖A中某些包，我们就必须在A中导出这些包，如果只有Pwin用到这些包 岂不破坏封装性。

最好的解决方法是把Pwin作为fragment，附加到A中。这样Pwin就能看到A中的所有包，A也能看到Pwin的所有包。

【场景2】针对不同国家用户提供不同的i18n

GUI程序通常会通过properties文件定义i18n信息，可以将不同的i18n存到不同的fragment中。运行时用户只需要下载host bundle以及特定的i18n fragment即可，不需要把其他国家的i18n也下载下来。

**Step6: 动态类加载**

查找Dynamic Import 列表的 Bundle，委托给对应Bundle 的类加载器加载。

参考文献：

   《深入理解java虚拟机》 周志明

  https://mp.weixin.qq.com/s?__biz=MzI4NDY5Mjc1Mg==&mid=2247483934&idx=1&sn=41c46eceb2add54b7cde9eeb01412a90&chksm=ebf6da61dc81537721d36aadb5d20613b0449762842f9128753e716ce5fefe2b659d8654c4e8&scene=21#wechat_redirect


联系方式：

https://github.com/longtian2

**如有用请不吝打赏**

![](https://github.com/longtian2/cc3/blob/master/images/wechat_pay.png)