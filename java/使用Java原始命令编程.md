# 使用Java命令行工具编程

java工程可以直接通过java平台提供的命令行工具进行编译和运行，实际上IDE也是最终使用java平台的命令行工具。

## 基础Java编译和运行
首先创建一个简单的Java文件：
```java
import java.lang.System;

public class Main {
    public static void main(String[] args) {
        System.out.println("Hello world!");
    }
}
```
假设将该源文件保存在"D:\src\rawjava"目录下，文件创建完毕后，通过javac命令编译该java文件：
```shell
PS D:\src\rawjava> javac Main.java
PS D:\src\rawjava> ls


    目录: D:\src\rawjava


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          2022/7/9     22:51            414 Main.class
-a----          2022/7/9     22:48            148 Main.java
```
在shell中可以看到编译生成了对应的class文件。此时通过java命令执行Main.class文件：
```shell
PS D:\src\rawjava> java Main
错误: 找不到或无法加载主类 Main
```
出现无法加载类的错误，网上五花八门的答案都有，实际上出现以上问题的原因在于，java虚拟机搜索类的路径中，没有找到Main.class。通过指定java虚拟机的classpath可以解决问题：
```shell
PS D:\src\rawjava> java -classpath "." Main
Hello world!
PS D:\src\rawjava> java -classpath "D:\src\rawjava" Main
Hello world!
```
可以简单的将当前路径加入classpath或全路径加入classpath。

前面的java文件中比较简单，可以看到并没有指定包信息。如果我们把包信息加上：
```java
package com.example;

import java.lang.System;

public class Main {
    public static void main(String[] args) {
        System.out.println("Hello world!");
    }
}
```
使用同样的命令编译和运行：
``` shell
PS D:\src\rawjava> javac Main.java
PS D:\src\rawjava> ls


    目录: D:\src\rawjava


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          2022/7/9     23:02            426 Main.class
-a----          2022/7/9     23:02            170 Main.java


PS D:\src\rawjava> java -classpath "D:\src\rawjava" Main
错误: 找不到或无法加载主类 Main
```

出现此问题的原因为：由于java文件声明了包名，那么编译出来的class文件中带有包信息，需要指定class的全名com.example.Main
```
PS D:\src\rawjava> java -classpath "D:\src\rawjava" com.example.Main
错误: 找不到或无法加载主类 com.example.Main
```
但是指定全包名路径仍然无法找到执行类，需要保证class的路径结构跟包名的结构一致：
```
PS D:\src\rawjava> tree /f
文件夹 PATH 列表
卷序列号为 44B1-98F2
D:.
│  Main.java
│
└─com
    └─example
            Main.class

PS D:\src\rawjava> java -classpath "D:\src\rawjava" com.example.Main
Hello world!

```
此时再执行java命令可以正常运行。与源码放置的关系没有关系，只需要保证class文件放置在与包名保持一致的目录结构下即可。但是一般我们会将源码文件的路径也保持包名一样的目录结构：
```
PS D:\src\rawjava> tree /f
文件夹 PATH 列表
卷序列号为 44B1-98F2
D:.
└─com
    └─example
            Main.class
            Main.java

PS D:\src\rawjava> javac .\com\example\Main.java
PS D:\src\rawjava> java -classpath "D:\src\rawjava" com.example.Main
Hello world!
```

## 将class文件打包为jar
可以将编译出来的java字节码文件打包为jar包，打包的方法通过jar命令行工具。

### 多个Java文件的编译
首先针对前面的Java文件进行调整，在同一级包名目录下新增一个Calculator的java文件：
```java
package com.example;

public class Calculator {
    public static int sum(int op1, int op2) {
        return op1 + op2;
    }
}
```
同时对Main.java做修改，调用Calculator中的函数：
```java
package com.example;
import java.lang.System;

public class Main {
    public static void main(String[] args) {
        System.out.println("Hello world!");
        System.out.println("1 + 2 = " + Calculator.sum(1, 2));
    }
}
```
然后分别对文件进行编译：
```
PS D:\src\rawjava> javac -sourcepath "D:\src\rawjava" .\com\example\Calculator.java
PS D:\src\rawjava> javac -sourcepath "D:\src\rawjava" .\com\example\Main.java
```
注意此处需要给javac指定-sorucepath参数，否则在编译Main的时候发现，其无法找到依赖的Calculator类。
```java
PS D:\src\rawjava> javac .\com\example\Main.java
.\com\example\Main.java:7: 错误: 找不到符号
        System.out.println("1 + 2 = " + Calculator.sum(1, 2));
  位置: 类 Main
```
可以直接编译Main.java，当编译Main.java的时候，发现依赖Calculator类，会自动先将Calculator类编译出来，javac自动处理了编译依赖的事情：
```shell
PS D:\src\rawjava> javac -sourcepath "D:\src\rawjava" .\com\example\Main.java
PS D:\src\rawjava> tree /f
文件夹 PATH 列表
卷序列号为 44B1-98F2
D:.
│  MANIFEST.MF
│
└─com
    └─example
            Calculator.class
            Calculator.java
            Main.class
            Main.java
```

### 多个class文件打包为Jar
如果打包的jar是可执行的jar，那么需要声明jar中的哪个类为main函数入口的类，通过配置一个Manifest文件的方式，Manifest文件为一个键值对，配置的内容如下：
```
Manifest-Version: 1.0
Main-Class: com.example.Main

```
其中主要的信息即为Main-Class信息，指定了jar文件中哪个类中具有main函数。 
根据oracle关于jar命令的文档的描述(https://docs.oracle.com/javase/tutorial/deployment/jar/modman.html)：
```
Warning: The text file from which you are creating the manifest must end with a new line or carriage return. The last line will not be parsed properly if it does not end with a new line or carriage return.
```
**在Manifest的最后一行，一定一定要加上一个空行结尾**，否则打包出来的jar包中的Manifest永远都是默认的信息：
```
Manifest-Version: 1.0
Created-By: 1.8.0_172 (Oracle Corporation)
```

创建完manifest后，通过jar命令将class文件和manifest文件打包：
``` Shell
PS D:\src\rawjava> tree /f
文件夹 PATH 列表
卷序列号为 44B1-98F2
D:.
│  MANIFEST.MF
│
└─com
    └─example
            Calculator.class
            Calculator.java
            Main.class
            Main.java

PS D:\src\rawjava> jar -cvfm main.jar MANIFEST.MF .\com\example\Calculator.class .\com\example\Main.class
已添加清单
正在添加: com/example/Calculator.class(输入 = 262) (输出 = 209)(压缩了 20%)
正在添加: com/example/Main.class(输入 = 711) (输出 = 438)(压缩了 38%)
```
jar的命令选项：
- c表示创建jar包
- v表示输出verbose信息
- f指定输出的文件名
- m表示指定了manifest文件

f和m都是带参数的选项，因此后面依次顺序对应的参数为main.jar和MANIFEST.MF。后面再紧跟着class列表。生成jar包，将原来未打包的class文件删除掉，验证通过jar包执行正常：
```
PS D:\src\rawjava> rm .\com\example\*.class
PS D:\src\rawjava> tree /f
文件夹 PATH 列表
卷序列号为 44B1-98F2
D:.
│  main.jar
│  MANIFEST.MF
│
└─com
    └─example
            Calculator.java
            Main.java

PS D:\src\rawjava> java -jar .\main.jar
Hello world!
1 + 2 = 3
```
可以看到打包到jar中的class可以正常通过java执行。


## 总结
本来以上知识是比较简单基础的知识，但是由于平时习惯使用IDE，很少直接使用java命令来做编译和运行。最近需要使用java命令来做一些基础技术问题，又遇到各种问题。在网上寻找的答案都是很扯淡的解答，不求甚解，甚至有要求不要声明package包定义的答案。  
为方便后续自查问题方便，特将实践过程的内容记录为本文
