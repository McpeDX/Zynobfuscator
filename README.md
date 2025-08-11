# OpenMYJ2C

#### Open Source Information
This open source version is the official open source, open source the old version, we will continue to update the paid version! The open source version can be compiled and used directly.

### The bases used in this obfuscation:
[native-obfuscator](https://github.com/radioegor146/native-obfuscator)



#### 介绍
OpenMYJ2C将编译的Java的Class字节码转换为C语言代码。交叉编译（您不用自己配置编译环境，OpenMYJ2C自动完成）可以生成Windows，Linux，Mac系统X86，ARM平台的动态链接库文件后，通过Java Native Interface 重新链接到原始程序。在此过程结束时，包含原始方法的.class文件的字节码中不会保留原始方法的信息。 



编译前
```
public class App {
	public static void main(String args[]) {
		System.out.println("Hello, world!");
	}
}
```
编译后

```
public class App {
	public static native void main(String args[]);
}
```

#### 使用说明

 _运行默认配置_

1.  在命令行切换到OpenMYJ2C.jar所在目录，运行java -jar OpenMYJ2C.jar 待编译jar路径 输出目录或文件路径
```bash
java -jar OpenMYJ2C.jar D:\dev\SnakeGame.jar D:\dev\SnakeGame 
```
2.  自定义配置
```xml
<OpenMYJ2C>
	<targets><!--需要编译动态链接库类型，根据自己的程序运行环境配置，每种配置会编译一个动态链接库，不需要的平台建议不要配置会增加jar包大小-->
		<target>WINDOWS_X86_64</target>
		<target>WINDOWS_AARCH64</target>
		<target>MACOS_X86_64</target>
		<target>MACOS_AARCH64</target>
		<target>LINUX_X86_64</target>
		<target>LINUX_AARCH64</target>
	</targets>
	<include>
		<match className="demo/abc/**" /><!--需要编译的类-->
	</include>
	<exclude>
		<match className="demo/Main" /><!--不要编译的类-->
	</exclude>
</OpenMYJ2C>
```

目前可以编译WINDOWS，LINUX，MACOS系统X86_64和ARM架构动态链接库，具体配置如下：

```bash
    WINDOWS_X86_64
    WINDOWS_AARCH64
    MACOS_X86_64
    MACOS_AARCH64
    LINUX_X86_64
    LINUX_AARCH64
```

 **注意** 
```bash
OpenMYJ2C的代码将比直接在JVM上运行的代码运行得慢得多。这是由于无法避免的本机函数调用的固有开销。
建议您只使用OpenMYJ2C来混淆应用程序中对性能不重要的敏感部分。要合理利用include，exclude配置要混淆方法。
```

关于include

白名单，如果配置白名单，则只会编译白名单匹配到的类和方法

关于exclude

黑名单，配置黑名单，则排除掉黑名单匹配的方法，如果不配置白名单，则匹配排除黑名单外的所有类和方法

关于match

```bash
<match className="demo/abc/*" methodName="main" methodDesc="(\[Ljava/lang/String;)V" />
```

其中

```bash
className 匹配类名，如果只配置className 则会匹配该类的全部方法，该属性支持demo/abc/*或demo.abc.*
methodName 匹配类的方法名，设置该属性则只匹配到类的方法，未匹配的不包含
methodDesc 匹配JVM的方法描述，相同方法名称，只匹配到对应方法描述方法
```

JVM 方法描述
```bash
方法描述格式：(parameterTypes)returnType 类型如下:

    V is the void type, used only for the return value of a method. If a method takes no parameters, parameterTypes should be left blank
    I is the primitive integer type
    J is the primitive long type
    S is the primitive short type
    F is the primitive float type
    D is the primitive integer type
    C is the primitive char type
    B is the primitive byte type
    Z is the primitive boolean type
    Ljava/lang/Object; is the fully qualified class java.lang.Object
    [elementType is an array of elementType. elementType may itself be an array for multidimensional arrays.

Note that [ is a regex special character and needs to be escaped with a \

For example:
Method Signature 	JVM Method Descriptor
void main(String[] args) 	([Ljava/lang/String;)V
String toString() 	()Ljava/lang/String;
void wait(long t, int n) 	(JI)V
boolean compute(int[][] k) 	([[I)Z
```
 _运行自定义配置_ 

在命令行执行如下命令

```bash
java -jar OpenMYJ2C.jar D:\dev\SnakeGame.jar D:\dev\SnakeGame -c config1.xml
```

 _运行注解配置_

1.  请在源码上增加Native和NotNative注解后执行

2.  在命令行执行如下命令

```bash
java -jar OpenMYJ2C.jar D:\dev\SnakeGame.jar D:\dev\SnakeGame -a
```

#### 注意事项

     较旧的Java编译器可能会发出JSR和RET指令，这是Java 7字节码或更新版本中不允许的弃用指令。OpenMYJ2C仅支持Java8字节码及以上版本，因此无法处理JSR/RET指令。如果您的应用程序中有包含Java 6类文件的较旧库，则应将它们排除。 
    Java 11中允许一个称为ConstantDynamic的新特性，它允许在运行时通过引导方法动态初始化常量池条目。包含ConstantDynamic条目的类文件目前与OpenMYJ2C不兼容。应将它们排除。

    JNI接口的限制限制了任何转换方法的性能。特别是，与Java相比，方法调用、字段访问和数组操作速度较慢。在JNI代码中，算术、强制转换、控制流和局部变量访问仍然非常快（甚至可以通过C编译器进行优化），在一些要求性能的方法应将他们排除

#### 编写安全的代码

OpenMYJ2C是一个强大的混淆器，但其有效性可能会受到其翻译的代码的限制。考虑以下几点：

不安全的写法

```bash
public class App {
	public static void main(String[] args) {
		if(!checkLicence()) {
			System.err.println("Invalid licence");
			return;
		}
		initApp();
		runApp();
	}

	private static native boolean checkLicence(); // Protected by OpenMYJ2C

	private static native void initApp(); // Protected by OpenMYJ2C

	private static native void runApp(); // Protected by OpenMYJ2C
}
```

在此示例中，即使checkLicence代码受OpenMYJ2C保护，攻击者也很容易修改主方法，直接返回 true 达到破解目的。

```bash
public class App {
	public static void main(String[] args) {
		if(!checkLicence()) {
			System.err.println("Invalid licence");
			return;
		}
		initApp();
		runApp();
	}

	private static boolean checkLicence() {
	    return true; //这样就可以达到破解目的，绕过正常授权验证方法
	}

	private static native void initApp(); // Protected by OpenMYJ2C

	private static native void runApp(); // Protected by OpenMYJ2C
}
```

比较好的写法

```bash
public class App {
	public static void main(String[] args) {
		checkLicenceAndInitApp();
		runApp();
	}

	private static native void checkLicenceAndInitApp(); // Protected by OpenMYJ2C
}
```


在这里，攻击者即使修改CheckLicensandInApp的方法，跳过授权部分但是也不知道里面要执行哪些初始化功能，修改之后应用程序将无法正常运行（因为它将无法初始化）。

推荐使用OpenMYJ2C保护应用程序的初始化代码，因为它只在运行的时候执行一次，初始化方法不会反复被执行不用担心有性能问题。 


#### 授权信息

1.免费版 只能编译一个类中的一个方法，可以永久免费使用

2.试用版 免费使用，编译后的jar包可以免费使用一周（7天），之后程序将无法使用

3.个人版 可按月按年付费，控制台会打印OpenMYJ2C的编译信息，控制台信息可定制，不能去除，同时有编译数量限制

4.专业版 可按月按年付费，控制台无任何信息输出，没有编译数量限制

#### Frequently Asked Questions

1. Unable to compile the dynamic link library
Answer: This may be caused by Chinese characters or special symbols in the file path, which prevent Zig from compiling. Move the files to a path without Chinese characters or special symbols, then try compiling again.


2. Do I need to compile for all platforms?
Answer: By default, it compiles for Windows, Linux, and Mac systems, supporting both 64-bit and ARM platforms. You can configure the compilation to target only the platforms you need.


3. Will OpenMYJ2C significantly affect my application's performance?
Answer: Many code protection tools involve a trade-off between performance and security. We recommend using it only on sensitive code or code where performance is not critical.


4. Does OpenMYJ2C support lambdas/streams/exceptions/threads/locks/etc.?
Answer: OpenMYJ2C works on compiled Java bytecode and supports any bytecode compiled for Java 8 or newer JVMs. It supports all Java language features, as well as other JVM languages such as Kotlin.


5. What advantages does OpenMYJ2C have over writing JNI methods manually?
Answer: Writing Java Native Interface code is challenging and often harder to debug. OpenMYJ2C allows you to write (and test!) your code in Java, using all Java features. In addition:

OpenMYJ2C can translate the output of Java obfuscators like Zelix, KlassMaster, or Stringer.

It can convert Java API features that have no direct C equivalent, such as lambdas, method references, and streams.

With OpenMYJ2C, you don’t need to know JNI or C, nor write runtime linking code—OpenMYJ2C injects it automatically.

It can translate existing Java code—you don’t need to waste time rewriting parts of an already completed application.



6. Can I apply extra obfuscation before using OpenMYJ2C?
Answer: Yes, this is possible, although we can’t guarantee compatibility with every obfuscation tool. Also, if a Java obfuscator is already used before OpenMYJ2C, further obfuscation might be unnecessary.


7. Can I apply extra obfuscation to the JAR output from OpenMYJ2C?
Answer: Applying name obfuscation to any methods/fields/classes processed by OpenMYJ2C will cause runtime crashes due to unresolved links. However, you are free to use string obfuscation, reference obfuscation, resource encryption, etc.


8. Getting java.security.InvalidKeyException: Illegal key size error
Answer: Replace local_policy.jar and US_export_policy.jar in the jdk/jre/lib/security directory with the ones from jce_policy-8.zip. Download and extract the ZIP to obtain these files.
