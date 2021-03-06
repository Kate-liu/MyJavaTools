

# JVM Advanced

## 为什么我们要学习Java虚拟机？

前不久我参加了一个国外程序员的讲座，讲座的副标题很有趣，叫做：“我如何学会停止恐惧，并且爱上 Java 虚拟机”。

这句话来自一部黑色幽默电影《奇爱博士》，电影描述了冷战时期剑拔弩张的氛围。

程序员之间的语言之争又未尝不是如此。写系统语言的鄙视托管语言低下的执行效率；写托管语言的则取笑系统语言需要手动管理内存；写动态语言的不屑于静态语言那冗余的类型系统；写静态语言的则嘲讽动态语言里面各种光怪陆离的运行时错误。

Java 作为应用最广的语言，自然吸引了不少的攻击，而身为 Java 程序员的你，或许在口水战中落了下风，忿忿于没有足够的知识武装自己；又或许想要深入学习 Java 语言，却又无从下手。甚至是在实践中被 Java 的启动性能、内存耗费所震惊，因此对 Java 语言本身产生了种种的怀疑与顾虑。

别担心，我就是来解答你对 Java 的种种疑虑的。“知其然”也要“知其所以然”，学习 Java 虚拟机的本质，更多是了解 Java 程序是如何被执行且优化的。这样一来，你才可以从内部入手，达到高效编程的目的。与此同时，你也可以为学习更深层级、更为核心的 Java 技术打好基础。

我相信在不少程序员的观念里，Java 虚拟机是透明的。在大家看来，我们仅需知道 Java 核心类库，以及第三方类库里 API 的用法，便可以专注于实现具体业务，并且依赖 Java 虚拟机自动执行乃至优化我们的应用程序。那么，我们还需要了解 Java 虚拟机吗？

我认为是非常有必要的。如果我们把核心类库的 API 比做数学公式的话，那么 Java 虚拟机的知识就好比公式的推导过程。掌握数学公式固然可以应付考试，但是了解背后的推导过程更加有助于记忆和理解。并且，在遇到那些没法套公式的情况下，我们也能知道如何解决。



### 学习Java 虚拟机的好处

具体来说，了解 Java 虚拟机有如下（但不限于）好处。

首先，Java 虚拟机提供了许多**配置参数**，用于满足不同应用场景下，对程序性能的需求。学习 Java 虚拟机，你可以针对自己的应用，最优化匹配运行参数。（你可以用下面这个例子看一下自己虚拟机的参数列表。）

```sh
举例来说，macOS 上的 Java 10 共有近千个配置参数（我的电脑是836个）：
 
$ java -XX:+PrintFlagsFinal -XX:+UnlockDiagnosticVMOptions -version | wc -l
java version "10" 2018-03-20
Java(TM) SE Runtime Environment 18.3 (build 10+46)
Java HotSpot(TM) 64-Bit Server VM 18.3 (build 10+46, mixed mode)
     812
```

其次，Java 虚拟机本身是一种**工程产品**，在实现过程中自然存在不少局限性。学习 Java 虚拟机，可以更好地规避它在使用中的 Bug，也可以更快地识别出 Java 虚拟机中的错误，

再次，Java 虚拟机拥有当前最前沿、最成熟的**垃圾回收算法实现**，以及**即时编译器实现**。学习 Java 虚拟机，我们可以了解背后的设计决策，今后再遇到其他代码托管技术也能触类旁通。

最后，Java 虚拟机发展到了今天，已经脱离 Java 语言，形成了一套**相对独立的、高性能的执行方案**。除了 Java 外，Scala、Clojure、Groovy，以及时下热门的 Kotlin，这些语言都可以运行在 Java 虚拟机之上。学习 Java 虚拟机，便可以了解这些语言的通用机制，甚至于让这些语言共享生态系统。



### 写作专栏初心

说起写作这个专栏的初心，与我个人的经历是分不开的，我现在是甲骨文实验室的高级研究员，工作主要是负责研究如何通过程序分析技术以及动态编译技术**让程序语言跑得更快**。明面上，我是 Graal 编译器的核心开发者之一，在为 HotSpot 虚拟机项目拧螺丝。

这里顺便说明一下，**Graal 编译器**是 Java 10 正式引入的实验性即时编译器，在国内同行口中被戏称为“甲骨文黑科技”。当然，在我看来，我们的工作同样也是分析应用程序的性能瓶颈，寻找优化空间，只不过我们的优化方式对自动化、通用性有更高的要求。

加入甲骨文之前，我在瑞士卢加诺大学攻读博士学位，研究如何更加精准地监控 Java 程序，以便做出更具针对性的优化。这些研究工作均已发表在程序语言方向的顶级会议上，并获得了不少同行的认可（OOPSLA 2015 最佳论文奖）。

在这 7 年的学习工作生涯中，我拜读过许多大神关于 Java 虚拟机的技术博客。在受益匪浅的同时，我发觉不少文章的门槛都比较高，而且过分注重实现细节，这并不是大多数的开发人员可以受益的调优方案。这么一来，许多原本对 Java 虚拟机感兴趣的同学， 也因为过高的门槛，以及短时间内看不到的收益，而放弃了对 Java 虚拟机的学习。

和其他栏目一样，我会用简单通俗的语言，来介绍 Java 虚拟机的实现。具体到每篇文章，我将采用一个**贯穿全文的案例**来阐述知识点，并且给出相应的调优建议。在文章的末尾，我还将附上一个动手实践的环节，帮助你巩固对知识点的理解。



### 专栏模块

整个专栏将分为四大模块。

1. **基本原理**：剖析 Java 虚拟机的运行机制，逐一介绍 Java 虚拟机的设计决策以及工程实现；
2. **高效实现**：探索 Java 编译器，以及内嵌于 Java 虚拟机中的即时编译器，帮助你更好地理解 Java 语言特性，继而写出简洁高效的代码；
3. **代码优化**：介绍如何利用工具定位并解决代码中的问题，以及在已有工具不适用的情况下，如何打造专属轮子；
4. **虚拟机黑科技**：介绍甲骨文实验室近年来的前沿工作之一 GraalVM。包括如何在 JVM 上高效运行其他语言；如何混搭这些语言，实现 Polyglot；如何将这些语言事前编译（Ahead-Of-Time，AOT）成机器指令，单独运行甚至嵌入至数据库中运行。

我希望借由这四个模块 36 个案例，帮助你理解 Java 虚拟机的运行机制，掌握诊断手法和调优方式。最重要的，是激发你学习 Java 虚拟机乃至其他底层工作、前沿工作的热情。





### Java虚拟机知识框架图

![Java虚拟机知识图谱](JVMAdvanced.assets/Java虚拟机知识图谱.jpg)



## Java代码是怎么运行的？

### 海关的Java 和 C++ 故事

我们学院的一位教授之前去美国开会，入境的时候海关官员就问他：既然你会计算机，那你说说你用的都是什么语言吧？

教授随口就答了个 Java。海关一看是懂行的，也就放行了，边敲章还边说他们上学那会学的是 C+。我还特意去查了下，真有叫 C+ 的语言，但是这里海关官员应该指的是 C++。

事后教授告诉我们，他当时差点就问海关，是否知道 **Java 和 C++ 在运行方式上的区别**。但是又担心海关官员拿他的问题来考别人，也就没问出口。那么，下次你去美国，不幸地被海关官员问这个问题，你懂得如何回答吗？

作为一名 Java 程序员，你应该知道，**Java 代码有很多种不同的运行方式**。比如说可以在开发工具中运行，可以双击执行 jar 文件运行，也可以在命令行中运行，甚至可以在网页中运行。当然，这些执行方式都离不开 **JRE**，也就是 Java 运行时环境。

实际上，JRE 仅包含运行 Java 程序的必需组件，包括 Java 虚拟机以及 Java 核心类库等。我们 Java 程序员经常接触到的 JDK（Java 开发工具包）同样包含了 JRE，并且还附带了一系列开发、诊断工具。

> JDK = JRE + 开发调试诊断工具
>
> JRE = JVM + Java标准库 (核心类库)

然而，运行 C++ 代码则无需额外的运行时。我们往往把这些代码直接编译成 CPU 所能理解的代码格式，也就是机器码。

比如下图的中间列，就是用 **C 语言**写的 Helloworld 程序的编译结果。可以看到，C 程序编译而成的机器码就是一个个的字节，它们是给机器读的。那么为了让开发人员也能够理解，我们可以用**反汇编器**将其转换成汇编代码（如下图的最右列所示）。

```sh
# 最左列是偏移；中间列是给机器读的机器码；最右列是给人读的汇编代码

0x00:  55                    push   rbp
0x01:  48 89 e5              mov    rbp,rsp
0x04:  48 83 ec 10           sub    rsp,0x10
0x08:  48 8d 3d 3b 00 00 00  lea    rdi,[rip+0x3b] 
                                    ; 加载 "Hello, World!\n"
0x0f:  c7 45 fc 00 00 00 00  mov    DWORD PTR [rbp-0x4],0x0
0x16:  b0 00                 mov    al,0x0
0x18:  e8 0d 00 00 00        call   0x12
                                    ; 调用 printf 方法
0x1d:  31 c9                 xor    ecx,ecx
0x1f:  89 45 f8              mov    DWORD PTR [rbp-0x8],eax
0x22:  89 c8                 mov    eax,ecx
0x24:  48 83 c4 10           add    rsp,0x10
0x28:  5d                    pop    rbp
0x29:  c3                    ret
```

既然 C++ 的运行方式如此成熟，那么你有没有想过，为什么 Java 要在虚拟机中运行呢，Java 虚拟机具体又是怎样运行 Java 代码的呢，它的运行效率又如何呢？

今天我便从这几个问题入手，和你探讨一下，Java 执行系统的主流实现以及设计决策。



### 为什么 Java 要在虚拟机里运行？

Java 作为一门高级程序语言，它的语法非常复杂，抽象程度也很高。因此，直接在硬件上运行这种复杂的程序并不现实。所以呢，在运行 Java 程序之前，我们需要对其进行一番转换。

这个转换具体是怎么操作的呢？

当前的主流思路是这样子的，设计一个面向 Java 语言特性的虚拟机，并通过编译器将 Java 程序转换成该虚拟机所能识别的指令序列，也称 **Java 字节码**。这里顺便说一句，之所以这么取名，是因为 Java 字节码指令的**操作码（opcode）被固定为一个字节**。

举例来说，下图的中间列，正是用 Java 写的 Helloworld 程序编译而成的字节码。可以看到，它与 C 版本的编译结果一样，都是由一个个字节组成的。

并且，我们同样可以将其反汇编为人类可读的代码格式（如下图的最右列所示）。不同的是，Java 版本的编译结果相对精简一些。这是因为 **Java 虚拟机相对于物理机而言，抽象程度更高**。

```sh
# 最左列是偏移；中间列是给虚拟机读的机器码；最右列是给人读的代码
0x00:  b2 00 02         getstatic java.lang.System.out
0x03:  12 03            ldc "Hello, World!"
0x05:  b6 00 04         invokevirtual java.io.PrintStream.println
0x08:  b1               return
```

[Java 虚拟机可以由硬件实现](https://en.wikipedia.org/wiki/Java_processor) ，但更为常见的是在各个现有平台（如 Windows_x64、Linux_aarch64）上提供软件实现。这么做的意义在于，一旦一个程序被转换成 Java 字节码，那么它便可以在不同平台上的虚拟机实现里运行。这也就是我们经常说的“**一次编写，到处运行**”。

虚拟机的另外一个好处是它带来了一个**托管环境（Managed Runtime）**。这个托管环境能够代替我们处理一些代码中冗长而且容易出错的部分。其中最广为人知的当属**自动内存管理与垃圾回收**，这部分内容甚至催生了一波垃圾回收调优的业务。

除此之外，托管环境还提供了诸如数组越界、动态类型、安全权限等等的**动态检测**，使我们免于书写这些无关业务逻辑的代码。



### Java 虚拟机具体是怎样运行 Java 字节码的？

下面我将以**标准 JDK 中的 HotSpot 虚拟机**为例，从虚拟机以及底层硬件两个角度，给你讲一讲 Java 虚拟机具体是怎么运行 Java 字节码的。



#### 虚拟机视角

从虚拟机视角来看，执行 Java 代码首先需要将它编译而成的 class 文件加载到 Java 虚拟机中。加载后的 Java 类会被存放于Java 虚拟机进程的方法区（Method Area）中。实际运行时，虚拟机会执行方法区内的代码。

如果你熟悉 X86 的话，你会发现这和**段式内存管理**中的代码段类似。而且，Java 虚拟机同样也在内存中划分出**堆和栈来存储运行时数据**。

不同的是，Java 虚拟机会将栈细分为面向 **Java 方法**的 Java 方法栈，面向**本地方法**（用 C++ 写的 native 方法）的本地方法栈，以及存放各个**线程执行位置**的 PC 寄存器。

![1606382043729](JVMAdvanced.assets/1606382043729.png)

在运行过程中，每当调用进入一个 Java 方法，Java 虚拟机会在当前线程的 Java 方法栈中生成一个**栈帧**，用以存放局部变量以及字节码的操作数。这个栈帧的大小是提前计算好的，而且 Java 虚拟机不要求栈帧在内存空间里连续分布。

当退出当前执行的方法时，不管是正常返回还是异常返回，Java 虚拟机均会弹出当前线程的当前栈帧，并将之舍弃。



#### 硬件视角

从硬件视角来看，Java 字节码无法直接执行。因此，Java 虚拟机需要将字节码翻译成机器码。

在 HotSpot 里面，上述翻译过程有两种形式：第一种是**解释执行**，即逐条将字节码翻译成机器码并执行；第二种是**即时编译（Just-In-Time compilation，JIT）**，即将一个方法中包含的所有字节码编译成机器码后再执行。

![1606382076001](JVMAdvanced.assets/1606382076001.png)

前者的优势在于无需等待编译，而后者的优势在于实际运行速度更快。**HotSpot 默认采用混合模式**，综合了解释执行和即时编译两者的优点。它会先解释执行字节码，而后将其中反复执行的**热点代码**，以方法为单位进行即时编译。



### Java 虚拟机的运行效率究竟是怎么样的？

HotSpot 采用了多种技术来提升启动性能以及峰值性能，刚刚提到的即时编译便是其中最重要的技术之一。

即时编译建立在程序符合**二八定律**的假设上，也就是百分之二十的代码占据了百分之八十的计算资源。

对于占据大部分的不常用的代码（八），我们无需耗费时间将其编译成机器码，而是采取解释执行的方式运行；另一方面，对于仅占据小部分的热点代码（二），我们则可以将其编译成机器码，以达到理想的运行速度。

理论上讲，即时编译后的 Java 程序的执行效率，是可能超过 C++ 程序的。这是因为与静态编译相比，即时编译拥有程序的运行时信息，并且能够根据这个信息做出相应的优化。

举个例子，我们知道**虚方法**是用来实现面向对象语言多态性的。对于一个虚方法调用，尽管它有很多个目标方法，但在实际运行过程中它可能只调用其中的一个。

这个信息便可以被即时编译器所利用，来规避虚方法调用的开销，从而达到比静态编译的 C++ 程序更高的性能。



为了满足不同用户场景的需要，HotSpot 内置了多个即时编译器：**C1（Client 编译器）、C2（Server 编译器） 和 Graal**。Graal 是 Java 10 正式引入的实验性即时编译器，在专栏的第四部分我会详细介绍，这里暂不做讨论。

之所以引入多个即时编译器，是为了在编译时间和生成代码的执行效率之间进行取舍。C1 又叫做 Client 编译器，面向的是对**启动性能**有要求的客户端 GUI 程序，采用的优化手段相对简单，因此编译时间较短。

C2 又叫做 Server 编译器，面向的是对**峰值性能**有要求的服务器端程序，采用的优化手段相对复杂，因此编译时间较长，但同时生成代码的执行效率较高。



从 Java 7 开始，HotSpot 默认采用**分层编译**的方式：热点方法首先会被 C1 编译，而后热点方法中的热点会进一步被 C2 编译。

为了不干扰应用的正常运行，HotSpot 的即时编译是放在额外的**编译线程**中进行的。HotSpot 会根据 CPU 的核心数量设置编译线程的数目，并且按 1:2 的比例配置给 C1 及 C2 编译器。

在计算资源充足的情况下，字节码的解释执行和即时编译可同时进行。编译完成后的机器码会在下次调用该方法时启用，以替换原本的解释执行。



### 总结与实践

今天我简单介绍了 Java 代码为何在虚拟机中运行，以及如何在虚拟机中运行。

之所以要在虚拟机中运行，是因为它提供了可移植性。一旦 Java 代码被编译为 Java 字节码，便可以在不同平台上的 Java 虚拟机实现上运行。此外，虚拟机还提供了一个代码托管的环境，代替我们处理部分冗长而且容易出错的事务，例如内存管理。

Java 虚拟机将运行时内存区域划分为五个部分，分别为方法区、堆、PC 寄存器、Java 方法栈和本地方法栈。Java 程序编译而成的 class 文件，需要先加载至方法区中，方能在 Java 虚拟机中运行。

为了提高运行效率，标准 JDK 中的 HotSpot 虚拟机采用的是一种混合执行的策略。

它会解释执行 Java 字节码，然后会将其中反复执行的热点代码，以方法为单位进行即时编译，翻译成机器码后直接运行在底层硬件之上。

HotSpot 装载了多个不同的即时编译器，以便在编译时间和生成代码的执行效率之间做取舍。

下面我给你留一个小作业，通过观察两个条件判断语句的运行结果，来思考 Java 语言和 Java 虚拟机看待 boolean 类型的方式是否不同。

下载[ asmtools.jar ](https://wiki.openjdk.java.net/display/CodeTools/asmtools) or [this link](https://ci.adoptopenjdk.net/view/Dependencies/job/asmtools/lastSuccessfulBuild/artifact/asmtools.jar)，并在命令行中运行下述指令（不包含提示符 $）：

```sh
$ echo '
public class Foo {
 public static void main(String[] args) {
  boolean flag = true;
  if (flag) System.out.println("Hello, Java!");
  if (flag == true) System.out.println("Hello, JVM!");
 }
}' > Foo.java
$ javac Foo.java
$ java Foo  # 此时执行的是 Foo.class
$ java -cp /path/to/asmtools.jar org.openjdk.asmtools.jdis.Main Foo.class > Foo.jasm.1
$ awk 'NR==1,/iconst_1/{sub(/iconst_1/, "iconst_2")} 1' Foo.jasm.1 > Foo.jasm
$ java -cp /path/to/asmtools.jar org.openjdk.asmtools.jasm.Main Foo.jasm
$ java Foo  # 此时执行的是 Foo.jasm
```

对那段 awk 不懂得可参考：
https://blog.csdn.net/jiaobuchong/article/details/83037467

```sh
# 将第一行 到 正则匹配到的第一行中的 in 替换为 on（从记录行的左边开始，只替换一次）
# 怎么理解这个 1 呢？
# 这里有两个模式，awk 读出每行记录都会经过这两个模式的判断
# 1 表示这个模式为真，没有指定模式默认的 action 就是打印整行 
awk 'NR == 1,/in/{sub(/in/, "on")} 1' pattern.txt
```

jvm 把boolean 当做int来处理(在实际的执行的时候，布尔变量是32位的int，不知占一个字节哈！)

flag = iconst_1 = true

awk 把 stackframe（栈帧） 中的 flag 改为iconst_2 

if（flag）比较时 ifeq 指令做是否为零判断，iconst_2 仍为true，打印输出

if（true == flag）比较时 if_cmpne 做整数比较，iconst_1是否等于flag，比较失败，不再打印输出

**asmtools与ASM**，是字节码工程包，就是一个提供了字节码抽象的工具，允许用Java代码来生成或者更改字节码。JDK里也会用到ASM，用来生成一些适配器什么的。代码覆盖工具JaCoCo也是用ASM来实现的。





## Java的基本类型

如果你了解面向对象语言的发展史，那你可能听说过 **Smalltalk** 这门语言。它的影响力之大，以至于之后诞生的面向对象语言，或多或少都借鉴了它的设计和实现。

在 Smalltalk 中，所有的值都是对象。因此，许多人认为它是一门纯粹的面向对象语言。

Java 则不同，它引进了**八个基本类型**，来支持数值计算。Java 这么做的原因主要是工程上的考虑，因为使用基本类型能够在执行效率以及内存使用两方面提升软件性能。

今天，我们就来了解一下基本类型在 Java 虚拟机中的实现。

```java
public class Foo {
  public static void main(String[] args) {
    boolean 吃过饭没 = 2; // 直接编译的话 javac 会报错
    if (吃过饭没) System.out.println(" 吃了 ");
    if (true == 吃过饭没) System.out.println(" 真吃了 ");
  }
}
```

在上一篇结尾的小作业里，我构造了这么一段代码，它将一个 boolean 类型的局部变量赋值为 2。为了方便记忆，我们给这个变量起个名字，就叫“吃过饭没”。

赋值语句后边我设置了两个看似一样的 if 语句。第一个 if 语句，也就是直接判断“吃过饭没”，在它成立的情况下，代码会打印“吃了”。

第二个 if 语句，也就是判断“吃过饭没”和 true 是否相等，在它成立的情况下，代码会打印“真吃了”。

当然，直接编译这段代码，编译器是会报错的。所以，我迂回了一下，采用一个 Java 字节码的汇编工具(ASM tools)，直接对字节码进行更改。

那么问题就来了：当一个 boolean 变量的值是 2 时，它究竟是 true 还是 false 呢？

如果你跑过这段代码，你会发现，问虚拟机“吃过饭没”，它会回答“吃了”，而问虚拟机“真（==）吃过饭没”，虚拟机则不会回答“真吃了”。

那么虚拟机到底吃过没，下面我们来一起分析一下这背后的细节。



### Java 虚拟机的 boolean 类型

首先，我们来看看 Java 语言规范以及 Java 虚拟机规范是怎么定义 boolean 类型的。

在 **Java 语言规范**中，boolean 类型的值只有两种可能，它们分别用符号**“true”和“false”**来表示。显然，这两个符号是不能被虚拟机直接使用的。

在 **Java 虚拟机规范**中，boolean 类型则被映射成 int 类型。具体来说，“true”被映射为整数 1，而“false”被映射为整数 0。这个编码规则约束了 Java 字节码的具体实现。

举个例子，对于存储 boolean 数组的字节码，Java 虚拟机需保证实际存入的值是整数 1 或者 0。

Java 虚拟机规范同时也要求 **Java 编译器**遵守这个编码规则，并且用整数相关的字节码来实现逻辑运算，以及基于 boolean 类型的条件跳转。这样一来，在编译而成的 class 文件中，除了字段和传入参数外，基本看不出 boolean 类型的痕迹了。

```sh
# Foo.main 编译后的字节码
 0: iconst_2       // 我们用 AsmTools 更改了这一指令
 1: istore_1
 2: iload_1
 3: ifeq 14        // 第一个 if 语句，即操作数栈上数值为 0 时跳转
 6: getstatic java.lang.System.out
 9: ldc " 吃了 "
11: invokevirtual java.io.PrintStream.println
14: iload_1
15: iconst_1
16: if_icmpne 27   // 第二个 if 语句，即操作数栈上两个数值不相同时跳转
19: getstatic java.lang.System.out
22: ldc " 真吃了 "
24: invokevirtual java.io.PrintStream.println
27: return
```

在前面的例子中，第一个 if 语句会被编译成**条件跳转字节码 ifeq**，翻译成人话就是说，如果局部变量“吃过饭没”的值为 0，那么跳过打印“吃了”的语句。

而第二个 if 语句则会被编译成条件跳转字节码 if_icmpne，也就是说，如果局部变量的值和整数 1 不相等，那么跳过打印“真吃了”的语句。

可以看到，Java 编译器的确遵守了相同的编码规则。当然，这个约束很容易**绕开**。除了我们小作业中用到的汇编工具 AsmTools 外，还有许多可以修改字节码的 Java 库，比如说 ASM [[1\] ](https://asm.ow2.io/)等。

对于 Java 虚拟机来说，它看到的 boolean 类型，早已被映射为整数类型。因此，将原本声明为 boolean 类型的局部变量，赋值为除了 0、1 之外的整数值，在 Java 虚拟机看来是“合法”的。

> javac不支持将原本声明为 boolean 类型的局部变量，赋值为除了 0、1 之外的整数值的操作，它把boolean是用int实现的这种虚拟机的实现细节给隐藏起来了，从而使得在语言层面没有这种会引起歧义的值。

在我们的例子中，经过编译器编译之后，Java 虚拟机看到的不是在问“吃过饭没”，而是在问“吃过几碗饭”。也就是说，第一个 if 语句变成：你不会一碗饭都没吃吧。第二个 if 语句则变成：你吃过一碗饭了吗。

如果我们约定俗成，每人每顿只吃一碗，那么第二个 if 语句还是有意义的。但如果我们打破常规，吃了两碗，那么较真的 Java 虚拟机就会将第二个 if 语句判定为假了。



### Java 的基本类型

除了上面提到的 boolean 类型外，Java 的基本类型还包括整数类型 byte、short、char、int 和 long，以及浮点类型 float 和 double。

![1606390815444](JVMAdvanced.assets/1606390815444.png)

Java 的基本类型都有对应的值域和默认值。可以看到，byte、short、int、long、float 以及 double 的值域依次扩大，而且前面的值域被后面的值域所包含。因此，从前面的基本类型转换至后面的基本类型，无需强制转换。另外一点值得注意的是，尽管他们的默认值看起来不一样，但**在内存中都是 0**。

在这些基本类型中，boolean 和 char 是唯二的无符号类型。在不考虑违反规范的情况下，boolean 类型的取值范围是 0 或者 1。char 类型的取值范围则是 [0, 65535]。通常我们可以认定 char 类型的值为非负数。这种特性十分有用，比如说作为数组索引等。

在前面的例子中，我们能够将整数 2 存储到一个声明为 boolean 类型的局部变量中。那么，声明为 byte、char 以及 short 的局部变量，是否也**能够存储超出它们取值范围的数值**呢？

答案是可以的。而且，这些超出取值范围的数值同样会带来一些麻烦。比如说，声明为 char 类型的局部变量实际上有可能为负数。当然，在正常使用 Java 编译器的情况下，生成的字节码会遵守 Java 虚拟机规范对编译器的约束，因此你无须过分担心局部变量会超出它们的取值范围。



Java 的浮点类型采用 [IEEE 754 浮点数格式](https://www.h-schmidt.net/FloatConverter/IEEE754.html)。以 float 为例，浮点类型通常有**两个 0**，**+0.0F 以及 -0.0F**。

前者在 Java 里是 0，后者是符号位为 1、其他位均为 0 的浮点数，在内存中等同于十六进制整数 0x8000000（即 -0.0F 可通过 Float.intBitsToFloat(0x8000000) 求得）。尽管它们的内存数值不同，但是在 Java 中 +0.0F == -0.0F 会返回真。

在有了 +0.0F 和 -0.0F 这两个定义后，我们便可以定义浮点数中的**正无穷及负无穷**。正无穷就是任意正浮点数（不包括 +0.0F）除以 +0.0F 得到的值，而负无穷是任意正浮点数除以 -0.0F 得到的值。在 Java 中，正无穷和负无穷是有确切的值，在内存中分别等同于十六进制整数 0x7F800000 和 0xFF800000。

你也许会好奇，既然整数 0x7F800000 等同于正无穷，那么 0x7F800001 又对应什么浮点数呢？

这个数字对应的浮点数是 **NaN（Not-a-Number）**。

不仅如此，[0x7F800001, 0x7FFFFFFF] 和 [0xFF800001, 0xFFFFFFFF] 对应的都是 NaN。当然，一般我们计算得出的 NaN，比如说通过 +0.0F/+0.0F，在内存中应为 0x7FC00000。这个数值，我们称之为**标准的 NaN**，而其他的我们称之为不标准的 NaN。

NaN 有一个有趣的特性：除了“!=”始终返回 true 之外，所有其他比较结果都会返回 false。

举例来说，“NaN<1.0F”返回 false，而“NaN>=1.0F”同样返回 false。对于任意浮点数 f，不管它是 0 还是 NaN，“f!=NaN”始终会返回 true，而“f==NaN”始终会返回 false。

因此，我们在程序里做**浮点数比较**的时候，需要考虑上述特性。在本专栏的第二部分，我会介绍这个特性给向量化比较带来什么麻烦。





### Java 基本类型的大小

在第一篇中我曾经提到，Java 虚拟机每调用一个 Java 方法，便会创建一个栈帧。为了方便理解，这里我只讨论供解释器使用的**解释栈帧（interpreted frame）**。

这种栈帧有两个主要的组成部分，分别是局部变量区，以及字节码的操作数栈。这里的局部变量是广义的，除了普遍意义下的局部变量之外，它还包含实例方法的“this 指针”以及方法所接收的参数。

#### 局部变量区-存储

在 Java 虚拟机规范中，**局部变量区等价于一个数组**，并且可以用正整数来索引。除了 long、double 值需要用两个数组单元来存储之外，其他基本类型以及引用类型的值均占用一个数组单元。

也就是说，boolean、byte、char、short 这四种类型，在栈上占用的空间和 int 是一样的，和引用类型也是一样的。因此，在 32 位的 HotSpot 中，这些类型在栈上将占用 4 个字节；而在 64 位的 HotSpot 中，他们将占 8 个字节。

> 所以，在64位的操作系统中，java程序运行的时候，会比32为操作系统更占内存。

当然，这种情况仅存在于局部变量，而并不会出现在存储于堆中的字段或者数组元素上。对于 byte、char 以及 short 这三种类型的字段或者数组单元，它们在堆上占用的空间分别为一字节、两字节，以及两字节，也就是说，跟这些类型的值域相吻合。



因此，当我们将一个 **int 类型的值**，存储到这些类型的字段或数组时，相当于做了一次**隐式的掩码操作**。举例来说，当我们把 0xFFFFFFFF（-1）存储到一个声明为 char 类型的字段里时，由于该字段仅占两字节，所以高两位的字节便会被截取掉，最终存入“\uFFFF”。

> 把 0xFFFFFFFF（-1）存储到一个声明为 short 类型的字段里时，由于该字段仅占两字节，所以高两位的字节便会被截取掉，最终存入“\uFFFF”。
>
> 把 0xFFFFFFFF（-1）存储到一个声明为 byte 类型的字段里时，由于该字段仅占一字节，所以高三位的字节便会被截取掉，最终存入“\uFF”。

**boolean 字段和 boolean 数组**则比较特殊。在 HotSpot 中，boolean 字段占用一字节，而 boolean 数组则直接用 byte 数组来实现。为了保证堆中的 boolean 值是合法的，HotSpot 在存储时**显式地进行掩码操作**，也就是说，只取最后一位的值存入 boolean 字段或数组中。



#### 操作数栈-加载

讲完了存储，现在我来讲讲加载。Java 虚拟机的算数运算几乎全部依赖于操作数栈。也就是说，我们需要将堆中的 boolean、byte、char 以及 short 加载到操作数栈上，而后将栈上的值**当成 int 类型来运算**。

对于 boolean、char 这两个无符号类型来说，加载伴随着**零扩展**。举个例子，char 的大小为两个字节。在加载时 char 的值会被复制到 int 类型的低二字节，而高二字节则会**用 0 来填充**。

> 零扩展是因为这两种基本数据类型是大于零的，不存在符号

对于 byte、short 这两个类型来说，加载伴随着**符号扩展**。举个例子，short 的大小为两个字节。在加载时 short 的值同样会被复制到 int 类型的低二字节。如果该 short 值为非负数，即最高位为 0，那么该 int 类型的值的高二字节会用 0 来填充，否则用 1 来填充。

> 符号扩展是因为这两种基本数据类型是有符号的，有大于零和小于零的值域





### 总结与实践

今天我介绍了 Java 里的基本类型。

其中，boolean 类型在 Java 虚拟机中被映射为整数类型：“true”被映射为 1，而“false”被映射为 0。Java 代码中的逻辑运算以及条件跳转，都是用整数相关的字节码来实现的。

除 boolean 类型之外，Java 还有另外 7 个基本类型。它们拥有不同的值域，但默认值在内存中均为 0。这些基本类型之中，浮点类型比较特殊。基于它的运算或比较，需要考虑 +0.0F、-0.0F 以及 NaN 的情况。

除 long 和 double 外，其他基本类型与引用类型在解释执行的方法栈帧中占用的大小是一致的，但它们在堆中占用的大小确不同。在将 boolean、byte、char 以及 short 的值存入字段或者数组单元时，Java 虚拟机会进行**掩码**操作。在读取时，Java 虚拟机则会将其扩展为 int 类型。

今天的动手环节，你可以观测一下，将 boolean 类型的值存入字段中时，Java 虚拟机所做的掩码操作。

你可以将下面代码中 boolValue = true 里的 true 换为 2 或者 3，看看打印结果与你的猜测是否相符合。

熟悉 Unsafe 的同学，可以使用 Unsafe.putBoolean 和 Unsafe.putByte 方法，看看还会不会做掩码操作。

>Unsafe就是一些不被虚拟机控制的内存操作的合集。
>
>CAS可以理解为原子性的写操作，这个概念来自于底层CPU指令。Unsafe提供了一些cas的Java接口，在即时编译器中我们会将对这些接口的调用替换成具体的CPU指令。

```java
public class Foo {
  static boolean boolValue;
  public static void main(String[] args) {
    boolValue = true;  // 将这个 true 替换为 2 或者 3，再看看打印结果
    if (boolValue) System.out.println("Hello, Java!");
    if (boolValue == true) System.out.println("Hello, JVM!");
  }
}
```

答案：当替换为2的时候无输出
当替换为3的时候打印HelloJava及HelloJVM

猜测是因为将boolean 保存在静态域中,指定了其类型为'Z',当修改为2（0010）时取低位最后一位为0,当修改为3（0011）时取低位最后一位为1

则说明boolean的掩码处理是**取低位的最后一位**



#### Unsafe.putBoolean 和 Unsafe.putByte 解读

> Unsafe.putBoolean会做掩码，另外方法返回也会对boolean byte char short进行掩码

Unsafe.putBoolean和Unsafe.puByte是native实现

putBoolean和putByte也是通过宏SET_FIELD模板出的函数

\#define SET_FIELD(obj, offset, type_name, x) \
  oop p = JNIHandles::resolve(obj); \
  *(type_name*)index_oop_from_field_offset_long(p, offset) = truncate_##type_name(x)

unsafe.cpp中定义宏做truncate
\#define truncate_jboolean(x) ((x) & 1)
\#define truncate_jbyte(x) (x)
\#define truncate_jshort(x) (x)
\#define truncate_jchar(x) (x)
\#define truncate_jint(x) (x)
\#define truncate_jlong(x) (x)
\#define truncate_jfloat(x) (x)
\#define truncate_jdouble(x) (x)

**综上：unsafe.Put*不会对值做修改**

getBoolean和getByte也是通过宏GET_FIELD模板出的函数

\#define GET_FIELD(obj, offset, type_name, v) \
  oop p = JNIHandles::resolve(obj); \
  type_name v = *(type_name*)index_oop_from_field_offset_long(p, offset)

**综上，unsafe.Get*不会对值做修改**

验证：
unsafe.putByte(foo, addr, (byte)2); // 设置为: 2
System.out.println(unsafe.getByte(foo, addr)); // 打印getByte: 2
System.out.println(unsafe.getBoolean(foo, addr)); // 打印getBoolean: true

unsafe.putByte(foo, addr, (byte)1); // 设置为: 1
System.out.println(unsafe.getByte(foo, addr)); // 打印getByte: 1
System.out.println(unsafe.getBoolean(foo, addr)); // 打印getBoolean: true

疑问：
if(foo.flag)判断，使用getfield    Field flag:"Z"，执行逻辑等于：0 ！= flag
if(foo.getFlag())判断，使用invokevirtual    Method getFlag:"()Z"，执行逻辑等于： 0 != （(flag) & 1）

附getFlag jasm码：
public Method getFlag:"()Z"
    stack 1 locals 1
{
        aload_0;
        getfield    Field flag:"Z";
        ireturn;
}







## Java虚拟机是如何加载Java类的?

听我的意大利同事说，他们那边有个习俗，就是父亲要帮儿子盖栋房子。

这事要放在以前还挺简单，亲朋好友搭把手，盖个小砖房就可以住人了。现在呢，整个过程要耗费好久的时间。首先你要请建筑师出个方案，然后去市政部门报备、验证，通过后才可以开始盖房子。盖好房子还要装修，之后才能住人。

盖房子这个事，和 Java 虚拟机中的类加载还是挺像的。从 class 文件到内存中的类，按先后顺序需要经过**加载、链接以及初始化**三大步骤。其中，链接过程中同样需要**验证**；而内存中的类没有经过初始化，同样不能使用。那么，是否所有的 Java 类都需要经过这几步呢？

![1605597326789](JavaAdvanced.assets/1605597326789.png)

我们知道 Java 语言的类型可以分为两大类：**基本类型（primitive types）**和**引用类型（reference types）**。在上一篇中，我已经详细介绍过了 Java 的基本类型，它们是由 Java 虚拟机预先定义好的。

至于另一大类引用类型，Java 将其细分为四种：**类、接口、数组类和泛型参数**。由于泛型参数会在编译过程中被擦除（我会在专栏的第二部分详细介绍），因此 Java 虚拟机实际上只有前三种。在类、接口和数组类中，数组类是由 Java 虚拟机直接生成的，其他两种则有对应的**字节流**。

说到字节流，最常见的形式要属由 Java 编译器生成的 class 文件。除此之外，我们也可以在程序内部直接生成，或者从网络中获取（例如网页中内嵌的小程序 Java applet）字节流。这些不同形式的字节流，都会被加载到 Java 虚拟机中，成为类或接口。为了叙述方便，下面我就用“类”来统称它们。

无论是直接生成的数组类，还是加载的类，Java 虚拟机都需要对其进行链接和初始化。接下来，我会详细给你介绍一下每个步骤具体都在干些什么。



### 加载

加载，是指查找字节流，并且据此创建类的过程。前面提到，对于数组类来说，它并没有对应的字节流，而是由 Java 虚拟机直接生成的。对于其他的类来说，Java 虚拟机则需要借助**类加载器**来完成查找字节流的过程。

以盖房子为例，村里的 Tony 要盖个房子，那么按照流程他得先找个建筑师，跟他说想要设计一个房型，比如说“一房、一厅、四卫”。你或许已经听出来了，这里的房型相当于类，而建筑师，就相当于类加载器。

村里有许多建筑师，他们等级森严，但有着共同的祖师爷，叫**启动类加载器（bootstrap class loader）**。启动类加载器是由 C++ 实现的，没有对应的 Java 对象，因此在 Java 中只能用 **null** 来指代。换句话说，祖师爷不喜欢像 Tony 这样的小角色来打扰他，所以谁也没有祖师爷的联系方式。

除了启动类加载器之外，其他的类加载器都是 java.lang.ClassLoader 的子类，因此有对应的 Java 对象。这些类加载器需要先由另一个类加载器，比如说启动类加载器，加载至 Java 虚拟机中，方能执行类加载。

村里的建筑师有一个潜规则，就是接到单子自己不能着手干，得先给师傅过过目。师傅不接手的情况下，才能自己来。在 Java 虚拟机中，这个潜规则有个特别的名字，叫**双亲委派模型**(其实是一个单亲)。每当一个类加载器接收到加载请求时，它会先将请求转发给父类加载器。在父类加载器没有找到所请求的类的情况下，该类加载器才会尝试去加载。

> 双亲委派模式，英文中为parent不带s，照理应该翻译为单亲。但既然约定俗成翻译为双亲，就只好这样叫啦！

在 Java 9 之前，启动类加载器负责加载最为基础、最为重要的类，比如存放在 JRE 的 lib 目录下 jar 包中的类（以及由虚拟机参数 -Xbootclasspath 指定的类）。除了启动类加载器之外，另外两个重要的类加载器是**扩展类加载器（extension class loader）**和**应用类加载器（application class loader）**，均由 Java 核心类库提供。

![1605597565964](JavaAdvanced.assets/1605597565964.png)

扩展类加载器的父类加载器是启动类加载器。它负责加载相对次要、但又通用的类，比如存放在 JRE 的 lib/ext 目录下 jar 包中的类（以及由系统变量 java.ext.dirs 指定的类）。

应用类加载器的父类加载器则是扩展类加载器。它负责加载应用程序路径下的类。（这里的应用程序路径，便是指虚拟机参数 -cp/-classpath、系统变量 java.class.path 或环境变量 CLASSPATH 所指定的路径。）默认情况下，应用程序中包含的类便是由应用类加载器加载的。

Java 9 引入了模块系统，并且略微更改了上述的类加载器[1](https://docs.oracle.com/javase/9/migrate/toc.htm#JSMIG-GUID-A868D0B9-026F-4D46-B979-901834343F9E)。扩展类加载器被改名为**平台类加载器（platform class loader）**。Java SE 中除了少数几个关键模块，比如说 java.base 是由启动类加载器加载之外，其他的模块均由平台类加载器所加载。

除了由 Java 核心类库提供的类加载器外，我们还可以加入**自定义的类加载器**，来实现特殊的加载方式。举例来说，我们可以对 class 文件进行加密，加载时再利用自定义的类加载器对其解密。

除了加载功能之外，类加载器还提供了**命名空间的作用**。这个很好理解，打个比方，咱们这个村不讲究版权，如果你剽窃了另一个建筑师的设计作品，那么只要你标上自己的名字，这两个房型就是不同的。

在 Java 虚拟机中，**类的唯一性**是由类加载器实例以及类的全名一同确定的。即便是同一串字节流，经由不同的类加载器加载，也会得到两个不同的类。在大型应用中，我们往往借助这一特性，来运行同一个类的不同版本。



### 链接

链接，是指将创建成的类合并至 Java 虚拟机中，使之能够执行的过程。它可分为**验证、准备以及解析**三个阶段。

**验证阶段**的目的，在于确保被加载类能够满足 Java 虚拟机的约束条件。这就好比 Tony 需要将设计好的房型提交给市政部门审核。只有当审核通过，才能继续下面的建造工作。

通常而言，Java 编译器生成的类文件必然满足 Java 虚拟机的约束条件。因此，这部分我留到讲解字节码注入时再详细介绍。

**准备阶段**的目的，则是为被加载类的静态字段分配内存。Java 代码中对静态字段的具体初始化，则会在稍后的初始化阶段中进行。过了这个阶段，咱们算是盖好了毛坯房。虽然结构已经完整，但是在没有装修之前是不能住人的。

除了分配内存外，部分 Java 虚拟机还会在此阶段构造其他跟类层次相关的数据结构，比如说用来实现虚方法的动态绑定的方法表。

在 class 文件被加载至 Java 虚拟机之前，这个类无法知道其他类及其方法、字段所对应的具体地址，甚至不知道自己方法、字段的地址。因此，每当需要引用这些成员时，Java 编译器会生成一个符号引用。在运行阶段，这个符号引用一般都能够无歧义地定位到具体目标上。

举例来说，对于一个方法调用，编译器会生成一个包含目标方法所在类的名字、目标方法的名字、接收参数类型以及返回值类型的符号引用，来指代所要调用的方法。

**解析阶段**的目的，正是将这些符号引用解析成为实际引用。如果符号引用指向一个未被加载的类，或者未被加载类的字段或方法，那么解析将触发这个类的加载（但未必触发这个类的链接以及初始化。）

> 注意：类的加载不等于类的初始化。

如果将这段话放在盖房子的语境下，那么符号引用就好比“Tony 的房子”这种说法，不管它存在不存在，我们都可以用这种说法来指代 Tony 的房子。实际引用则好比实际的通讯地址，如果我们想要与 Tony 通信，则需要启动盖房子的过程。

Java 虚拟机规范并没有要求在链接过程中完成解析。它仅规定了：如果某些字节码使用了符号引用，那么在执行这些字节码之前，需要完成对这些符号引用的解析。



### 初始化

在 Java 代码中，如果要**初始化一个静态字段**，我们可以在声明时直接赋值，也可以在静态代码块中对其赋值。

如果直接赋值的静态字段被 final 所修饰，并且它的类型是基本类型或字符串时，那么该字段便会被 Java 编译器标记成**常量值（ConstantValue）**，其初始化直接由 Java 虚拟机完成。除此之外的直接赋值操作，以及所有静态代码块中的代码，则会被 Java 编译器置于同一方法中，并把它命名为 **< clinit >**。

>初始化调用<clinit>(即class init)

类加载的最后一步是初始化，便是为标记为常量值的字段赋值，以及执行 < clinit > 方法的过程。Java 虚拟机会通过**加锁**来确保类的 < clinit > 方法仅被执行一次。

只有当初始化完成之后，类才正式成为可执行的状态。这放在我们盖房子的例子中就是，只有当房子装修过后，Tony 才能真正地住进去。

那么，**类的初始化何时会被触发呢？**JVM 规范枚举了下述多种触发情况：

1. 当虚拟机启动时，初始化用户指定的主类；
2. 当遇到用以新建目标类实例的 new 指令时，初始化 new 指令的目标类；
3. 当遇到调用静态方法的指令时，初始化该静态方法所在的类；
4. 当遇到访问静态字段的指令时，初始化该静态字段所在的类；
5. 子类的初始化会触发父类的初始化；
6. 如果一个接口定义了 default 方法，那么直接实现或者间接实现该接口的类的初始化，会触发该接口的初始化；
7. 使用反射 API 对某个类进行反射调用时，初始化这个类；
8. 当初次调用 MethodHandle 实例时，初始化该 MethodHandle 指向的方法所在的类。

```java
public class Singleton {
  private Singleton() {}
  private static class LazyHolder {
    static final Singleton INSTANCE = new Singleton();
  }
  public static Singleton getInstance() {
    return LazyHolder.INSTANCE;
  }
}
```

我在文章中贴了一段代码，这段代码是在著名的**单例延迟初始化例子**中[2](https://en.wikipedia.org/wiki/Initialization-on-demand_holder_idiom)，只有当调用 Singleton.getInstance 时，程序才会访问 LazyHolder.INSTANCE，才会触发对 LazyHolder 的初始化（对应第 4 种情况），继而新建一个 Singleton 的实例。

由于类初始化是线程安全的，并且仅被执行一次，因此程序可以确保多线程环境下有且仅有一个 Singleton 实例。



### 总结与实践

今天我介绍了 Java 虚拟机将字节流转化为 Java 类的过程。这个过程可分为加载、链接以及初始化三大步骤。

加载是指查找字节流，并且据此创建类的过程。加载需要借助类加载器，在 Java 虚拟机中，类加载器使用了双亲委派模型，即接收到加载请求时，会先将请求转发给父类加载器。

链接，是指将创建成的类合并至 Java 虚拟机中，使之能够执行的过程。链接还分验证、准备和解析三个阶段。其中，解析阶段为非必须的。

初始化，则是为标记为常量值的字段赋值，以及执行 < clinit > 方法的过程。类的初始化仅会被执行一次，这个特性被用来实现单例的延迟初始化。

今天的实践环节，你可以来验证一下本篇中的理论知识。

通过 JVM 参数 -verbose:class 来打印类加载的先后顺序，并且在 LazyHolder 的初始化方法中打印特定字样。在命令行中运行下述指令（不包含提示符 $）：

```java
$ echo '
public class Singleton {
  private Singleton() {}
  private static class LazyHolder {
    static final Singleton INSTANCE = new Singleton();
    static {
      System.out.println("LazyHolder.<clinit>");
    }
  }
  public static Object getInstance(boolean flag) {
    if (flag) return new LazyHolder[2];
    return LazyHolder.INSTANCE;
  }
  public static void main(String[] args) {
    getInstance(true);
    System.out.println("----");
    getInstance(false);
  }
}' > Singleton.java
$ javac Singleton.java
$ java -verbose:class Singleton
```

问题 1：新建数组（第 11 行）会导致 LazyHolder 的加载吗？会导致它的初始化吗？

```java
// 从下面可以看出：
// getInstance(true); 新建数组，只是触发了 LazyHolderd 的加载，并没有触发类的初始化
// getInstance(false); 触发了类的初始化
// java 虚拟机必须知道（加载）有这个类，才能创建这个类的数组（容器），但是这个类并没有被使用到（没有达到初始化的条件），所以不会初始化。

[Loaded Singleton from file:/Users/rmliu/Desktop/]
[Loaded sun.launcher.LauncherHelper$FXHelper from /Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Class$MethodArray from /Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Void from /Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded Singleton$LazyHolder from file:/Users/rmliu/Desktop/]
----
LazyHolder.<clinit>
```



在命令行中运行下述指令（不包含提示符 $）：

```sh
$ java -cp /path/to/asmtools.jar org.openjdk.asmtools.jdis.Main Singleton\$LazyHolder.class > Singleton\$LazyHolder.jasm.1
$ awk 'NR==1,/stack 1/{sub(/stack 1/, "stack 0")} 1' Singleton\$LazyHolder.jasm.1 > Singleton\$LazyHolder.jasm
$ java -cp /path/to/asmtools.jar org.openjdk.asmtools.jasm.Main Singleton\$LazyHolder.jasm
$ java -verbose:class Singleton
```

问题 2：新建数组会导致 LazyHolder 的链接吗？

```java
// 从下面可以看出：
// getInstance(true); 新建数组不会链接 LazyHolderd , 链接的第一步：验证字节码，awk把字节码改的不符合jvm规范
// getInstance(false); 真正链接和初始化
// 新建数组的时候并不是要使用这个类（只是定义了放这个类的容器），所以不会被链接，调用getInstance(false)的时候约等于告诉虚拟机，我要使用这个类了，你把这个类造好（链接），然后把static修饰的字符赋予变量（初始化）。

[Loaded Singleton from file:/Users/rmliu/Desktop/]
[Loaded sun.launcher.LauncherHelper$FXHelper from /Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Class$MethodArray from /Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded java.lang.Void from /Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/jre/lib/rt.jar]
[Loaded Singleton$LazyHolder from file:/Users/rmliu/Desktop/]
----
LazyHolder.<clinit>
```





## JVM是如何执行方法调用的？（上）

前不久在写代码的时候，我不小心踩到一个**可变长参数的坑**。

你或许已经猜到了，它正是可变长参数方法的**重载**造成的。（注：官方文档建议避免重载可变长参数方法，见 [[1]](https://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html) 的最后一段。）（Generally speaking, you should not overload a varargs method, or it will be difficult for programmers to figure out which overloading gets called.）

我把踩坑的过程放在了文稿里，你可以点击查看。

```java
void invoke(Object obj, Object... args) { ... }
void invoke(String s, Object obj, Object... args) { ... }
 
invoke(null, 1);    // 调用第二个 invoke 方法 
				  // 由于 String 是 Object 的子类，因此 Java 编译器会认为第二个方法更为贴切。
				  // 选择一个最为贴切的方法
invoke(null, 1, 2); // 调用第二个 invoke 方法
invoke(null, new Object[]{1}); // 调用第一个 invoke 方法
							// 只有手动绕开可变长参数的语法糖，才能调用第一个 invoke 方法
```

当时情况是这样子的，某个 API 定义了两个同名的重载方法。其中，第一个接收一个 Object，以及声明为 Object…的变长参数；而第二个则接收一个 String、一个 Object，以及声明为 Object…的变长参数。

这里我想调用第一个方法，传入的参数为 (null, 1)。也就是说，声明为 Object 的形式参数所对应的实际参数为 null，而变长参数则对应 1。

通常来说，之所以不提倡可变长参数方法的重载，是因为 Java 编译器可能无法决定应该调用哪个目标方法。

在这种情况下，**编译器会报错**，并且提示这个方法调用有**二义性**。然而，Java 编译器直接将我的方法调用识别为调用第二个方法，这究竟是为什么呢？

带着这个问题，我们来看一看 Java 虚拟机是怎么识别目标方法的。



### 重载与重写

在 Java 程序里，如果同一个类中出现多个名字相同，并且参数类型相同的方法，那么它无法通过编译。

也就是说，在正常情况下，如果我们想要在同一个类中定义名字相同的方法，那么它们的参数类型必须不同。这些方法之间的关系，我们称之为**重载**。

```sh
# 小知识：这个限制可以通过字节码工具绕开。也就是说，在编译完成之后，我们可以再向 class 文件中添加方法名和参数类型相同，而返回类型不同的方法。
# 当这种包括多个方法名相同、参数类型相同，而返回类型不同的方法的类，出现在 Java 编译器的用户类路径上时，它是怎么确定需要调用哪个方法的呢？
# 当前版本的 Java 编译器会直接选取第一个方法名以及参数类型匹配的方法。并且，它会根据所选取方法的返回类型来决定可不可以通过编译，以及需不需要进行值转换等。
```

重载的方法在编译过程中即可完成识别。

具体到每一个方法调用，Java 编译器会**根据所传入参数的声明类型**（注意与实际类型区分）**来选取重载方法**。选取的过程共分为**三个阶段**：

1. 在不考虑对基本类型自动装拆箱（auto-boxing，auto-unboxing），以及可变长参数的情况下选取重载方法；
2. 如果在第 1 个阶段中没有找到适配的方法，那么在允许自动装拆箱，但不允许可变长参数的情况下选取重载方法；
3. 如果在第 2 个阶段中没有找到适配的方法，那么在允许自动装拆箱以及可变长参数的情况下选取重载方法。

如果 Java 编译器在同一个阶段中找到了多个适配的方法，那么它会在其中选择一个最为贴切的，而决定**贴切程度**的一个关键就是**形式参数类型的继承关系**。



在**开头的例子**中，当传入 null 时，它既可以匹配第一个方法中声明为 Object 的形式参数，也可以匹配第二个方法中声明为 String 的形式参数。由于 String 是 Object 的子类，因此 Java 编译器会认为第二个方法更为贴切。



除了同一个类中的方法，重载也可以作用于这个类所继承而来的方法。也就是说，如果子类定义了与父类中非私有方法同名的方法，而且这两个方法的参数类型不同，那么在子类中，这两个方法同样构成了重载。

那么，如果子类定义了与父类中非私有方法同名的方法，而且这两个方法的参数类型相同，那么这两个方法之间又是什么关系呢？

如果这两个方法都是**静态**的，那么子类中的方法**隐藏**了父类中的方法。如果这两个方法都**不是静态**的，且都不是私有的，那么子类的方法**重写**了父类中的方法。



众所周知，Java 是一门面向对象的编程语言，它的一个重要特性便是**多态**。而**方法重写**，正是多态最重要的一种体现方式：它允许子类在继承父类部分功能的同时，拥有自己独特的行为。

打个比方，如果你经常漫游，那么你可能知道，拨打 10086 会根据你当前所在地，连接到当地的客服。重写调用也是如此：它会根据调用者的动态类型，来选取实际的目标方法。





### JVM 的静态绑定和动态绑定

接下来，我们来看看 Java 虚拟机是怎么识别方法的。

Java 虚拟机识别方法的关键在于**类名**、**方法名**以及**方法描述符（method descriptor）**。

前面两个就不做过多的解释了。至于**方法描述符，它是由方法的参数类型以及返回类型所构成**。在同一个类中，如果同时出现多个名字相同且描述符也相同的方法，那么 Java 虚拟机会在类的验证阶段报错。

可以看到，Java 虚拟机与 Java 语言不同，它并不限制名字与参数类型相同，但返回类型不同的方法出现在同一个类中，对于**调用这些方法的字节码**来说，由于**字节码所附带的方法描述符包含了返回类型**，因此 Java 虚拟机能够准确地识别目标方法。



Java 虚拟机中关于方法重写的判定同样基于方法描述符。也就是说，如果子类定义了与父类中非私有、非静态方法同名的方法，那么只有当**这两个方法的参数类型以及返回类型一致**，Java 虚拟机才会判定为**重写**。

对于 Java 语言中重写而 Java 虚拟机中非重写的情况，编译器会通过生成**桥接方法** [[2]](https://docs.oracle.com/javase/tutorial/java/generics/bridgeMethods.html) 来实现 Java 中的重写语义。

由于对重载方法的区分在编译阶段已经完成，我们可以认为 **Java 虚拟机不存在重载**这一概念。

因此，在某些文章中，**重载**也被称为**静态绑定（static binding**），或者**编译时多态（compile-time polymorphism）**；而**重写**则被称为**动态绑定（dynamic binding）**。

这个说法在 Java 虚拟机语境下并非完全正确。这是因为某个类中的重载方法可能被它的子类所重写，因此 Java 编译器会将所有对非私有实例方法的调用编译为需要动态绑定的类型。



确切地说，**Java 虚拟机中的静态绑定**指的是在解析时便能够直接识别目标方法的情况，而**动态绑定**则指的是需要在运行过程中根据调用者的动态类型来识别目标方法的情况。

具体来说，Java 字节码中与**调用相关的指令**共有五种。

1. invokestatic：用于调用静态方法，这是方法调用指令中最快的一个。
2. invokespecial：用于调用私有实例方法、构造器，以及使用 super 关键字调用父类的实例方法或构造器，和所实现接口的默认方法。
3. invokevirtual：用于调用非私有实例方法。
4. invokeinterface：用于调用接口方法。
5. invokedynamic：JDK7 新增加的指令，用于调用动态方法， JDK8 以后支持 lambda 表达式的实现基础。



由于 invokedynamic 指令较为复杂，我将在后面的篇章中单独介绍。这里我们只讨论前四种。

我在文章中贴了一段代码，展示了编译生成这四种调用指令的情况。

```java
interface 客户 {
  boolean isVIP();
}
 
class 商户 {
  public double 折后价格 (double 原价, 客户 某客户) {
    return 原价 * 0.8d;
  }
}
 
class 奸商 extends 商户 {
  @Override
  public double 折后价格 (double 原价, 客户 某客户) {
    if (某客户.isVIP()) {                         // invokeinterface      
      return 原价 * 价格歧视 ();                    // invokestatic
    } else {
      return super. 折后价格 (原价, 某客户);          // invokespecial
    }
  }
  public static double 价格歧视 () {
    // 咱们的杀熟算法太粗暴了，应该将客户城市作为随机数生成器的种子。
    return new Random()                          // invokespecial
           .nextDouble()                         // invokevirtual
           + 0.8d;
  }
}
```

在代码中，“商户”类定义了一个成员方法，叫做“折后价格”，它将接收一个 double 类型的参数，以及一个“客户”类型的参数。这里“客户”是一个接口，它定义了一个接口方法，叫“isVIP”。

我们还定义了另一个叫做“奸商”的类，它继承了“商户”类，并且重写了“折后价格”这个方法。如果客户是 VIP，那么它会被给到一个更低的折扣。

在这个方法中，我们首先会**调用“客户”接口的”isVIP“方法**。该调用会被编译为 invokeinterface 指令。

如果客户是 VIP，那么我们会**调用奸商类的一个名叫“价格歧视”的静态方法**。该调用会被编译为 invokestatic 指令。如果客户不是 VIP，那么我们会**通过 super 关键字调用父类的“折后价格”方法**。该调用会被编译为 invokespecial 指令。

在静态方法“价格歧视”中，我们会**调用 Random 类的构造器**。该调用会被编译为 invokespecial 指令。然后我们会以这个新建的 Random 对象为调用者，**调用 Random 类中的 nextDouble 方法**。该调用会被编译为 invokevirutal 指令。



对于 invokestatic 以及 invokespecial 而言，Java 虚拟机能够直接识别具体的目标方法。

而对于 invokevirtual 以及 invokeinterface 而言，在绝大部分情况下，虚拟机需要在执行过程中，根据调用者的动态类型，来确定具体的目标方法。

**唯一的例外**在于，如果虚拟机能够确定目标方法有且仅有一个，比如说目标方法被标记为 **final** [[3]](https://wiki.openjdk.java.net/display/HotSpot/VirtualCalls) [[4]](https://wiki.openjdk.java.net/display/HotSpot/InterfaceCalls)，那么它可以不通过动态类型，直接确定目标方法。





### 调用指令的符号引用

在编译过程中，我们并不知道目标方法的具体内存地址。因此，Java 编译器会暂时用**符号引用**来表示该目标方法。这一符号引用**包括目标方法所在的类或接口的名字，以及目标方法的方法名和方法描述符**。

符号引用**存储在 class 文件的常量池**之中。

根据目标方法是否为接口方法，这些引用可分为**接口符号引用和非接口符号引用**。我在文章中贴了一个例子，利用“javap -v”打印某个类的常量池，如果你感兴趣的话可以到文章中查看。

```java
// 在奸商.class 的常量池中，#16 为接口符号引用，指向接口方法 " 客户.isVIP()"。而 #22 为非接口符号引用，指向静态方法 " 奸商. 价格歧视 ()"。
// 使用 -v ，输出附加信息，看到更详细的字节码参数:

$ javap -v 奸商.class ...
Constant pool:
...
  #16 = InterfaceMethodref #27.#29        // 客户.isVIP:()Z
...
  #22 = Methodref          #1.#33         // 奸商. 价格歧视:()D
...
```



上一篇中我曾提到过，在执行使用了符号引用的字节码前，Java 虚拟机需要**解析这些符号引用，并替换为实际引用**。



对于**非接口符号引用**，假定该符号引用所指向的类为 C，则 Java 虚拟机会按照如下步骤进行查找。

1. 在 C 中查找符合名字及描述符的方法。
2. 如果没有找到，在 C 的父类中继续搜索，直至 Object 类。
3. 如果没有找到，在 C 所直接实现或间接实现的接口中搜索，这一步搜索得到的目标方法必须是非私有、非静态的。并且，如果目标方法在间接实现的接口中，则需满足 C 与该接口之间没有其他符合条件的目标方法。如果有多个符合条件的目标方法，则任意返回其中一个。

从这个解析算法可以看出，**静态方法也可以通过子类来调用**。此外，子类的静态方法会隐藏（注意与重写区分）父类中的同名、同描述符的静态方法。



对于**接口符号引用**，假定该符号引用所指向的接口为 I，则 Java 虚拟机会按照如下步骤进行查找。

1. 在 I 中查找符合名字及描述符的方法。
2. 如果没有找到，在 Object 类中的公有实例方法中搜索。
3. 如果没有找到，则在 I 的超接口中搜索。这一步的搜索结果的要求与非接口符号引用步骤 3 的要求一致。

经过上述的解析步骤之后，符号引用会被解析成实际引用。对于可以静态绑定的方法调用而言，**实际引用是一个指向方法的指针**。对于需要动态绑定的方法调用而言，**实际引用则是一个方法表的索引**。

具体什么是方法表，我会在下一篇中做出解答。





### 总结与实践

今天我介绍了 **Java 以及 Java 虚拟机是如何识别目标方法**的。

在 Java 中，方法存在重载以及重写的概念，**重载**指的是方法名相同而参数类型不相同的方法之间的关系，**重写**指的是方法名相同并且参数类型也相同的方法之间的关系。

Java 虚拟机识别方法的方式略有不同，除了方法名和参数类型之外，它还会考虑返回类型。

在 Java 虚拟机中，静态绑定指的是在解析时便能够直接识别目标方法的情况，而动态绑定则指的是需要在运行过程中根据调用者的动态类型来识别目标方法的情况。由于 Java 编译器已经区分了重载的方法，因此可以认为 Java 虚拟机中不存在重载。

在 class 文件中，Java 编译器会用符号引用指代目标方法。在执行调用指令前，它所附带的符号引用需要被解析成实际引用。对于可以静态绑定的方法调用而言，实际引用为目标方法的指针。对于需要动态绑定的方法调用而言，实际引用为辅助动态绑定的信息。



在文中我曾提到，**Java 的重写与 Java 虚拟机中的重写**并不一致，但是编译器会通过生成**桥接方法**来弥补。今天的实践环节，我们来看一下**两个生成桥接方法**的例子。你可以通过“javap -v”来查看 class 文件所包含的方法。

1. 重写方法的**返回类型不一致**：

```java
interface Customer {
  boolean isVIP();
}
 
class Merchant {
  public Number actionPrice(double price, Customer customer) {
		//    ...
        return 1;
  }
}
 
class NaiveMerchant extends Merchant {  
  @Override
  public Double actionPrice(double price, Customer customer) {
        //    ...
        return 2.0d;
  }
}

// ------------------------------------------------
// javac org/copydays/thinking/java/jvm/core/technology/override/NaiveMerchant.java
// javap -v org/copydays/thinking/java/jvm/core/technology/override/NaiveMerchant
// 直接 Double 复写了 Number 返回值类型的函数 actionPrice
Constant pool:
...
   #5 = Methodref          #6.#20         // org/copydays/thinking/java/jvm/core/technology/override/NaiveMerchant.actionPrice:(DLorg/copydays/thinking/java/jvm/core/technology/override/Customer;)Ljava/lang/Double;
   #6 = Class              #21            // org/copydays/thinking/java/jvm/core/technology/override/NaiveMerchant
   #7 = Class              #22            // org/copydays/thinking/java/jvm/core/technology/override/Merchant
...
  #12 = Utf8               actionPrice
  #13 = Utf8               (DLorg/copydays/thinking/java/jvm/core/technology/override/Customer;)Ljava/lang/Double;
...
  #20 = NameAndType        #12:#13        // actionPrice:(DLorg/copydays/thinking/java/jvm/core/technology/override/Customer;)Ljava/lang/Double;
  #21 = Utf8               org/copydays/thinking/java/jvm/core/technology/override/NaiveMerchant
  #22 = Utf8               org/copydays/thinking/java/jvm/core/technology/override/Merchant
```

2. 范型参数类型造成的**方法参数类型不一致**：

```java
interface Customer {
  boolean isVIP();
}
 
class Merchant<T extends Customer> {
 	public double actionPrice(double price, T customer) {
        //    ...
        return 3.0d;
 	}
}
 
class VIPOnlyMerchant extends Merchant<VIP> {
	 @Override
 	public double actionPrice(double price, VIP customer) {
        //    ...
        return 2.0d;
 	}
}

// ------------------------------------------------
// javac org/copydays/thinking/java/jvm/core/technology/override/VIPOnlyMerchant.java
// javap -v org/copydays/thinking/java/jvm/core/technology/override/VIPOnlyMerchant
Constant pool:
...
   #4 = Class              #17            // org/copydays/thinking/java/jvm/core/technology/override/VIPOnlyMerchant
   #5 = Class              #18            // org/copydays/thinking/java/jvm/core/technology/override/Merchant
...
    #10 = Utf8               actionPrice
...
   #17 = Utf8               org/copydays/thinking/java/jvm/core/technology/override/VIPOnlyMerchant
   #18 = Utf8               org/copydays/thinking/java/jvm/core/technology/override/Merchant
```





## JVM是如何执行方法调用的？（下）

我在读博士的时候，最怕的事情就是被问有没有**新的 Idea**。有一次我被老板问急了，就随口说了一个。

这个 Idea 究竟是什么呢，我们知道，设计模式大量使用了**虚方法来实现多态**。但是虚方法的性能效率并不高，所以我就说，是否能够在此基础上写篇文章，评估每一种设计模式因为虚方法调用而造成的性能开销，并且在文章中强烈谴责一下？

当时呢，我老板教的是一门高级程序设计的课，其中有好几节课刚好在讲设计模式的各种好处。所以，我说完这个 Idea，就看到老板的神色略有不悦了，脸上写满了“小郑啊，你这是舍本逐末啊”，于是，我就连忙挽尊，说我是开玩笑的。

在这里呢，我犯的错误其实有两个。

- 第一，我不应该因为虚方法的性能效率，而放弃良好的设计。
- 第二，通常来说，Java 虚拟机中虚方法调用的性能开销并不大，有些时候甚至可以完全消除。

第一个错误是原则上的，这里就不展开了。至于第二个错误，我们今天便来聊一聊 **Java 虚拟机中虚方法调用的具体实现**。

首先，我们来看一个**模拟出国边检的小例子**。

```java
abstract class Passenger {
 	abstract void passThroughImmigration();
    
 	@Override
 	public String toString() { ... }
}

class ForeignerPassenger extends Passenger {
	@Override
 	void passThroughImmigration() { /* 进外国人通道 */ }
}

class ChinesePassenger extends Passenger {
 	@Override
 	void passThroughImmigration() { /* 进中国人通道 */ }
    
 	void visitDutyFreeShops() { /* 逛免税店 */ }
}
 
Passenger passenger = ...
passenger.passThroughImmigration();
```

这里我定义了一个抽象类，叫做 Passenger，这个类中有一个名为 passThroughImmigration 的抽象方法，以及重写自 Object 类的 toString 方法。

然后，我将 Passenger 粗暴地分为两种：ChinesePassenger 和 ForeignerPassenger。

两个类分别实现了 passThroughImmigration 这个方法，具体来说，就是中国人走中国人通道，外国人走外国人通道。由于咱们储蓄较多，所以我在 ChinesePassenger 这个类中，还特意添加了一个叫做 visitDutyFreeShops 的方法。

那么在实际运行过程中，Java 虚拟机是如何高效地**确定每个 Passenger 实例应该去哪条通道的呢？**我们一起来看一下。



### 1. 虚方法调用

在上一篇中我曾经提到，Java 里所有非私有实例方法调用都会被编译成 invokevirtual 指令，而接口方法调用都会被编译成 invokeinterface 指令。这两种指令，均属于 **Java 虚拟机中的虚方法调用**。

在绝大多数情况下，Java 虚拟机需要根据调用者的动态类型，来确定虚方法调用的目标方法。这个过程我们称之为**动态绑定**。那么，相对于静态绑定的非虚方法调用来说，**虚方法调用更加耗时**。

在 Java 虚拟机中，**静态绑定**包括用于调用静态方法的 invokestatic 指令，和用于调用构造器、私有实例方法以及超类非私有实例方法的 invokespecial 指令。如果虚方法调用指向一个标记为 final 的方法，那么 Java 虚拟机也可以静态绑定该虚方法调用的目标方法。

Java 虚拟机中采取了一种**用空间换取时间的策略**来实现动态绑定。它为每个类生成一张**方法表**，用以快速定位目标方法。那么方法表具体是怎样实现的呢？





### 2. 方法表

在介绍那篇类加载机制的链接部分中，我曾提到**类加载的准备阶段**，它除了为静态字段分配内存之外，还会构造与该类相关联的方法表。

这个数据结构，便是 Java 虚拟机实现动态绑定的关键所在。

下面我将以 invokevirtual 所使用的**虚方法表（virtual method table，vtable）**为例介绍方法表的用法。invokeinterface 所使用的**接口方法表（interface method table，itable）**稍微复杂些，但是原理其实是类似的。



方法表本质上是一个**数组**，每个数组元素指向一个当前类及其祖先类中非私有的实例方法。

这些方法可能是具体的、可执行的方法，也可能是没有相应字节码的抽象方法。方法表满足**两个特质**：

- 其一，子类方法表中包含父类方法表中的所有方法；
- 其二，子类方法在方法表中的索引值，与它所重写的父类方法的索引值相同。

我们知道，方法调用指令中的**符号引用**会在执行之前解析成**实际引用**。

- 对于静态绑定的方法调用而言，实际引用将指向具体的目标方法。
- 对于动态绑定的方法调用而言，实际引用则是方法表的索引值（实际上并不仅是索引值）。

在执行过程中，Java 虚拟机将获取调用者的**实际类型**，并在该实际类型的虚方法表中，根据索引值获得目标方法。这个过程便是动态绑定。

![1607351800848](JVMAdvanced.assets/1607351800848.png)

在我们的例子中，Passenger 类的方法表包括两个方法：

- toString
- passThroughImmigration，

它们分别对应 0 号和 1 号。之所以方法表**调换了 toString 方法和 passThroughImmigration 方法的位置**，是因为 toString 方法的索引值需要与 Object 类中同名方法的索引值一致。为了保持简洁，这里我就不考虑 Object 类中的其他方法。

ForeignerPassenger 的方法表同样有两行。其中，0 号方法指向继承而来的 Passenger 类的 toString 方法。1 号方法则指向自己重写的 passThroughImmigration 方法。

ChinesePassenger 的方法表则包括三个方法，除了继承而来的 Passenger 类的 toString 方法，自己重写的 passThroughImmigration 方法之外，还包括独有的 visitDutyFreeShops 方法。

```java
Passenger passenger = ...
passenger.passThroughImmigration();
```

这里，Java 虚拟机的工作可以想象为**导航员**。每当来了一个乘客需要出境，导航员会先问是中国人还是外国人（获取动态类型），然后翻出中国人 / 外国人对应的小册子（获取动态类型的方法表），小册子的第 1 页便写着应该到哪条通道办理出境手续（用 1 作为索引来查找方法表所对应的目标方法）。

实际上，使用了方法表的动态绑定与静态绑定相比，仅仅**多出几个内存解引用操作**：访问栈上的调用者，读取调用者的动态类型，读取该类型的方法表，读取方法表中某个索引值所对应的目标方法。相对于创建并初始化 Java 栈帧来说，这几个内存解引用操作的开销简直可以忽略不计。



那么我们是否可以认为虚方法调用对性能没有太大影响呢？

其实是**不能**的，上述优化的效果看上去十分美好，但实际上仅存在于解释执行中，或者即时编译代码的最坏情况中。这是因为即时编译还拥有另外**两种性能更好的优化手段**：**内联缓存（inlining cache）和方法内联（method inlining）**。

下面我便来介绍第一种内联缓存。





### 3. 内联缓存

内联缓存是一种**加快动态绑定**的优化技术。

它能够**缓存**虚方法调用中调用者的动态类型，以及该类型所对应的目标方法。在之后的执行过程中，如果碰到已缓存的类型，内联缓存便会直接调用该类型所对应的目标方法。如果没有碰到已缓存的类型，内联缓存则会退化至使用基于方法表的动态绑定。

在我们的例子中，这相当于导航员记住了上一个出境乘客的国籍和对应的通道，例如中国人，走了左边通道出境。那么下一个乘客想要出境的时候，导航员会先问是不是中国人，是的话就走左边通道。如果不是的话，只好拿出外国人的小册子，翻到第 1 页，再告知查询结果：右边。



在针对多态的优化手段中，我们通常会提及以下**三个术语**。

1. **单态（monomorphic）**指的是仅有一种状态的情况。
2. **多态（polymorphic）**指的是有限数量种状态的情况。二态（bimorphic）是多态的其中一种。
3. **超多态（megamorphic）**指的是更多种状态的情况。通常我们用一个具体数值来区分多态和超多态。在这个数值之下，我们称之为多态。否则，我们称之为超多态。

对于内联缓存来说，我们也有对应的单态内联缓存、多态内联缓存和超多态内联缓存。

- **单态内联缓存**，顾名思义，便是只缓存了一种动态类型以及它所对应的目标方法。它的实现非常简单：比较所缓存的动态类型，如果命中，则直接调用对应的目标方法。
- **多态内联缓存**则缓存了多个动态类型及其目标方法。它需要逐个将所缓存的动态类型与当前动态类型进行比较，如果命中，则调用对应的目标方法。

一般来说，我们会将更加热门的动态类型放在前面。在实践中，大部分的虚方法调用均是单态的，也就是只有一种动态类型。为了节省内存空间，**Java 虚拟机只采用单态内联缓存**。



前面提到，当内联缓存没有命中的情况下，Java 虚拟机需要重新使用方法表进行动态绑定。对于内联缓存中的内容，我们有两种选择。

- 一是**替换单态内联缓存中的纪录**。这种做法就好比 CPU 中的数据缓存，它对数据的局部性有要求，即在替换内联缓存之后的一段时间内，方法调用的调用者的动态类型应当保持一致，从而能够有效地利用内联缓存。
  因此，在最坏情况下，我们用两种不同类型的调用者，轮流执行该方法调用，那么每次进行方法调用都将替换内联缓存。也就是说，只有写缓存的额外开销，而没有用缓存的性能提升。
- 另外一种选择则是**劣化为超多态状态**。这也是 Java 虚拟机的具体实现方式。处于这种状态下的内联缓存，实际上放弃了优化的机会。它将直接访问方法表，来动态绑定目标方法。与替换内联缓存纪录的做法相比，它牺牲了优化的机会，但是节省了写缓存的额外开销。

具体到我们的例子，如果来了一队乘客，其中外国人和中国人依次隔开，那么在重复使用的单态内联缓存中，导航员需要反复记住上个出境的乘客，而且记住的信息在处理下一乘客时又会被替换掉。因此，倒不如一直不记，以此来节省脑细胞。



虽然内联缓存附带内联二字，但是它**并没有内联目标方法**。这里需要明确的是，任何方法调用除非被内联，否则都会有**固定开销**。这些开销来源于保存程序在该方法中的执行位置，以及新建、压入和弹出新方法所使用的栈帧。

对于极其简单的方法而言，比如说 getter/setter，这部分固定开销占据的 CPU 时间甚至超过了方法本身。

此外，在**即时编译中**，方法内联不仅仅能够消除方法调用的固定开销，而且还增加了进一步优化的可能性，我们会在专栏的第二部分详细介绍方法内联的内容。





### 总结与实践

今天我介绍了虚方法调用在 Java 虚拟机中的实现方式。

虚方法调用包括 invokevirtual 指令和 invokeinterface 指令。如果这两种指令所声明的目标方法被标记为 final，那么 Java 虚拟机会采用静态绑定。

否则，Java 虚拟机将采用动态绑定，在运行过程中根据调用者的动态类型，来决定具体的目标方法。

Java 虚拟机的动态绑定是通过方法表这一数据结构来实现的。方法表中每一个重写方法的索引值，与父类方法表中被重写的方法的**索引值一致**。

在解析虚方法调用时，Java 虚拟机会纪录下所声明的目标方法的索引值，并且在运行过程中根据这个索引值查找具体的目标方法。

Java 虚拟机中的**即时编译器**会使用内联缓存来加速动态绑定。**Java 虚拟机所采用的单态内联缓存将纪录调用者的动态类型，以及它所对应的目标方法**。

当碰到新的调用者时，如果其动态类型与缓存中的类型匹配，则直接调用缓存的目标方法。

否则，Java 虚拟机将该内联缓存劣化为超多态内联缓存，在今后的执行过程中直接使用方法表进行动态绑定。



在今天的实践环节，我们来观测一下单态内联缓存和超多态内联缓存的性能差距。为了消除方法内联的影响，请使用如下的命令。

```java
// Run with: java -XX:CompileCommand="dontinline,*.passThroughImmigration" Passenger

public abstract class Passenger {
    abstract void passThroughImmigration();

    public static void main(String[] args) {
        Passenger a = new ChinesePassenger();
        Passenger b = new ForeignerPassenger();
        long current = System.currentTimeMillis();
        for (int i = 1; i <= 2_000_000_000; i++) {
            if (i % 100_000_000 == 0) {
                long temp = System.currentTimeMillis();
                System.out.println(temp - current);
                current = temp;
            }
            Passenger c = (i < 1_000_000_000) ? a : b;
            c.passThroughImmigration();
        }
    }
}

class ChinesePassenger extends Passenger {
    @Override
    void passThroughImmigration() {
    }
}

class ForeignerPassenger extends Passenger {
    @Override
    void passThroughImmigration() {
    }
}
```





## JVM是如何处理异常的？

























































































































