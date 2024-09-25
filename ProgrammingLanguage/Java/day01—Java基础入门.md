## 一、 Java背景知识

### 1.1 Java语言的历史

- **Java是哪家公司的产品？**

  Java是美国Sun（Stanford University Network，斯坦福大学网络公司）公司在1995年推出的一门计算机**高级编程语言**。但是在2009年是Sun公司被Oracle（甲骨文）公司给收购了，所以目前Java语言是Oracle公司所有产品。

- **Java名称的来历？**

  早期这门语言的名字其实不叫Java，当时称为Oak（橡树的意思），为什么叫橡树呢？原因是因为Sun公司的门口种了很多橡树，但是后来由于商标注册时，Oak商标已经其他公司注册了，所以后面改名为Java了。那么有人好奇为什么叫Java呢？Java是印度的一个岛屿，上面盛产咖啡，可能是因为他们公司的程序员喜欢喝咖啡，所以就改名为Java了。

- **Java的创始人是谁？**

- 说完Java名称的来历之后，接下来我们聊聊Java的祖师爷是谁？ Java的联合创始人有很多，但是行业普遍认可的Java的创始人 是**詹姆斯●高斯林**，被称为Java之父

![1660152660273](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252328432.png)

### 1.2 Java能做什么

了解了Java语言的历史之后，接下来，大家比较关心的问题可能是Java到底能做什么了？

![1660141834075](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252328944.png)

其实Java能做的事情非常多，它可以做桌面应用的开发、企业互联网应用开发、移动应用开发、服务器系统开发、大数据开发、游戏开发等等。

```java
1.桌面应用开发：能够在电脑桌面运行的软件
	举例：财务管理软件、编写程序用的IDEA开发工具等，可以用Java语言开发
	
2.企业级应用开发：大型的互联网应用程序
	举例：淘宝、京东、大家每天都用的tlias教学管理系统等

3.移动应用开发：运行的Android手机端的软件
	举例：QQ客户端、抖音APP等

4.服务器系统：应用程序的后台（为客户端程序提供数据）
	举例：服务器系统为用户推荐那你喜爱的视频

5.大数据开发：大数据是一个互联网开发方向
	举例：目前最火的大数据开发平台是Hadoop，就是用Java语言开发的

6.游戏开发：游戏本质上是给用户提供娱乐的软件，有良好的交互感受
	举例：我的世界MineCraft就是用Java语言开发的
```

虽然Java能做的事情非常多，但并不是每一个方向都被市场认可（比如桌面应用使用Java语言开发就不太方便，而使用C#语言是比较推荐的）。**目前Java的主流开发方向是使用Java开发企业级互联网应用程序**（很多公司的OA系统，客户关系管理系统，包括传智播客使用教学实施管理系统都是用Java语言开发的）

### 1.3 Java的技术体系

说完Java语言能做什么之后，接下来我们再给同学们介绍一下Java的技术体系。所谓技术体系，就是Java为了满足不同的应用场景提供了不同的技术版本，主要有三个版本。

- Java SE（Java Standard Edition）：叫做标准版，它是后面两个版本的基础，也就是学习后面两个版本必须先学习JavaSE。**我们基础班现阶段学习的就是这个版本中的技术**。

- Java EE（Java Enterprise Edition）: 叫做企业版，它是为企业级应用开发提供的一套解决方案。**在后面就业班课程中主要学习这个版本中的技术**。
- Java ME（Java Micro Edition）：叫做小型版，它为开发移动设备的应用提供了一套解决方案。**目前已经不被市场认可（淘汰），取而代之的是基于Android系统的应用开发**。

```java
1.Java是什么？
	答：Java是一门高级编程语言
	
2.Java是哪家公司的产品？
	答：Java以前是Sun公司的产品，现在Java是属于Oracle公司的产品
	
3.Java之父是谁？
	答：詹姆斯●高斯林
	
4.Java主流的开发方向是什么？
	答：企业级互联网应用开发
	
5.Java技术平台有哪些？
	答：JavaSE（标准版）、JavaEE（企业版）、JavaME（小型版）
```

## 二、 Java快速入门

这里所说的Java开发环境，实际上就是Java官方提供的一个软件，叫做JDK（全称是Java Develop Kit），翻译过来意思就是Java开发工具包。**我们先要到官网上去下载JDK，然后安装在自己的电脑上，才可以在自己的电脑上使用JDK来开发Java程序**

JDK的版本有很多，下图是JDK版本更新的历程图，有LTS标识的是长期支持版本（意思就是Oracle会不定期更新）。目前公司中用得最多的版本是JDK8版本，在目前这套课程中我们为了将一些新特性会使用JDK17版本。

![1660143538211](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252329254.png)



下面提供了详细的JDK下载和安装过程的截图，大家只需要按照步骤操作就行。

### 2.1 JDK下载和安装

- **JDK的下载**

这是JDK下载的官方网址 https://www.oracle.com/java/technologies/downloads/，你需要把该网址复制到浏览器的地址栏，敲回车

![1660527717279](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252329211.png)

进入网址后，选择JDK17版本，找到Windows标签，选择x64 Installer版本。如下图所示

![1660527981411](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252329940.png)

下载完成之后，在你下载的目录下会出现一个JDK的安装包，如下图所示

![1660528307458](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252329877.png)

到这JDK的下载就完成了，接下来就需要按照下面的步骤完成JDK安装.

-  **JDK的安装**

双击安装包，按照下图引导，点击下一步即可安装。**需要注意的是安装JDK后不像你安装QQ一样会在桌面上显示一个图标，JDK安装后桌面上没有图标！！！**

![1660144855615](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252329772.png)

**如何验证安装成功了呢？**

刚才不是让你记住安装目录吗？你记住了吗？如果你自己修改过目录，就打开你自己修改的目录；如果没有修改安装目录，默认在`C:\Program Files\Java\jdk-17.0.3`目录下。

在文件资源管理器打开JDK的安装目录的bin目录，会发现有两个命令工具 `javac.exe` `java.exe` ，这就是JDK提供给我们使用的**编译工具和运行工具**，如下图所示

![1660145259521](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252329763.png)

我们现在就使用一下 `javac.exe` `java.exe` 这两个工具，测试一下JDK是否可用

1. 第一步：在JDK的bin目录，地址栏输入cmd，回车

![1660529458474](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252330116.png)

输入完cmd回车后，会出现一个黑窗口，专业说法叫**命令行窗口**

![1660529493477](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252330924.png)

2. 第二步：在命令行窗口中输入 `javac -version`回车，然后输入`java -version`回车

   如果出现下面红色框框的提示正确版本号，和我们安装的JDK版本号一致，就说明JDK安装成功

![1660145482256](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252330630.png)

做完以上步骤之后，电脑上就已经有Java的开发环境了，接下来可以开发Java程序了。

### 2.2 cmd常见命令

前面测试JDK是否安装成功，需要在黑窗口中输入`javac -version`和`java -version` 这其实就是JDK查看编译工具和运行工具版本号的命令。

这种输入命令的和电脑交互的方式，称之为命令行交互。也就是说，可以使用命令指挥电脑做事情。接下来我们了解几种Windows系统常见的命令，后面可能会用到。

下面是Windows系统常见的命令以及作用，小伙伴们可以自己试一试。需要注意的是，每敲完一条命令之后，马上敲回车就表示执行这条命名。

```java
E:  //切换到E盘
cd [目录]        //进入指定的目录
cd ..         //退回到上一级目录
cd /         //退回到根目录
dir             //显示当前目录下所有的内容
cls             //清空屏幕
```

### 2.3 Java入门程序

上一节已经安装好了JDK，接下来，我们就正式开始开发第一个入门Java程序。按照国际惯例，学习任何一本编程语言第一个案例都叫做 **Hello World**，意思是向世界问好，从此开用程序和世界沟通的大门。

> **编写Java程序的步骤**

编写一个Java程序需要经过3个步骤：**编写代码，编译代码，运行代码**

![1660145843138](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252330508.png)

- [x] 编写代码：任何一个文本编辑器都可以些代码，如Windows系统自带的记事本
- [x] 编译代码：将人能看懂的源代码（.java文件）转换为Java虚拟机能够执行的字节码文件（.class文件）
- [x] 运行代码：将字节码文件交给Java虚拟机执行

----

> **编写第一个Java入门程序**

按照下面提供的步骤，一步一步的完成第一个Java入门程序的编写、编译和执行。

**第一步**：新建一个后缀为.java的文本文件`HelloWorld.java`，用记事本编写代码如下。

```java
public class HelloWorld {
   public static void main(String[] args) {
     System.out.println(" HelloWorld ");
    }
}
```

**第二步**：进入`HelloWorld.java`文件所在目录，在地址栏输入cmd回车，即可在此处打开命令行窗口。

![1660146387184](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252330321.png)

编译：在命令行窗口输入编译命令`javac HelloWorld`完成编译，编译后会生成一个`HelloWorld.class`文件。

![1660146644956](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252330570.png)

**第三步**：再接着输入`java HelloWorld`就可以运行了，运行结果如下。

![1660146816170](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252330846.png)

### 2.4 Java程序中常见的问题

刚才在编写第一个HelloWorld程序的时候，是不是很容易报错啊？第一次写代码，90%的同学都会有些小问题的，比如单词写错了！ 括号少写一个！等等！  写错代码都是很正常的，**一个什么错都犯过的程序员，才是真正的程序员**。

下面我们把程序中常见的问题，总结一下。大家在写代码时注意一下这些问题就可以了

- [x] Windows的文件扩展名没有勾选
- [x] 代码写了，但是忘记保存了
- [x] 文件名和类名不一致。
- [x] 英文大小写错误，单词拼写错误，存在中文符号，找不到main方法。
- [x] 括号不配对。
- [x] 编译或执行工具使用不当。

---

> - **文件扩展名没有打开**

下图中文件扩展名的勾勾没有勾选，就会导致你创建的文件是普通的文本文件（.txt）文件，而不是java文件。

**正确做法是把文件扩展名的勾选上**

![1660147216279](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252331520.png)

> - **文件名和类名不一致**

你看下图中，文件名是`HelloWorld`，但是类名是`Helloworld`看出区别了吗？一个是大写的W，一个是小写的w。 不仔细看还真看不出来。 

**正确写法是文件名叫`HelloWorld`，类名也叫`HelloWorld**`

![1660531741851](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252331013.png)

> - **单词大小写错吴**

下图中不是string和system这两个单词都写错了， 这里是严格区分大小写的

**正确写法是String和System**

![1660531915677](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252331414.png)

> - **主方法写错了**

下图所示，主方法的名称写成了`mian`，这是错误的。

主方法正确写法：必须是` public static void main(String[] args){}`，一个字母都不能错。

![1660532147208](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252331661.png)

> - **标点符号写错了**

下图中打印语句最后的分号，写成功中文分号`；`

**正确写法应该是英文分号** `;`  不仔细看还真看不出区别，要小心

![1660532298281](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252332029.png)

### 2.5 JDK的组成

我们已经安装了JDK，并且开发了一个Java入门程序，用javac命令编译，用Java命令运行，但是对于Java程序的执行原理并没有过多的介绍。 

下面我们把JDK的组成，以及跨平台原理给大家介绍一下，有利于同学们理解Java程序的执行过程。 

JDK由JVM、核心类库、开发工具组成，如下图所示

![1660147531310](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252332807.png)

下面分别介绍一下JDK中每一个部分是用来干什么的

```java
- 什么是JVM?
    答：JDK最核心的组成部分是JVM（Java Virtual Machine），它是Java虚拟机，真正运行Java程序的地方。
    
- 什么是核心类库？
	答：它是Java本身写好的一些程序，给程序员调用的。 Java程序员并不是凭空开始写代码，是要基于核心类库提供的一些基础代码，进行编程。
	
- 什么是JRE?
    答：JRE（Java Runtime Enviroment），意思是Java的运行环境；它是由JVM和核心类库组成的；如果你不是开发人员，只需要在电脑上安装JRE就可以运行Java程序。
    
- 什么是开发工具呢？
	答：Java程序员写好源代码之后，需要编译成字节码，这里会提供一个编译工具叫做javac.exe，编写好源代码之后，想要把class文件加载到内存中运行，这里需要用到运行工具java.exe。 
	除了编译工具和运行工具，还有一些其他的反编译工具、文档工具等待...
```

JDK、JRE的关系用一句话总结就是：用JDK开发程序，交给JRE运行

### 2.6 Java的跨平台原理

学完JDK的组成后，我们知道Java程序的执行是依赖于Java虚拟机的。就是因为有了Java虚拟机所以Java程序有一个重要的特性叫做跨平台性。

- **什么是跨平台行呢？**

  所谓跨平台指的是用Java语言开发的程序可以在多种操作系统上运行，常见的操作系统有Windows、Linux、MacOS系统。

  如果没有跨平台性，同一个应用程序，想要在多种操作系统上运行，需要针对各个操作系统单独开发应用。比如微信有Windows版本、MacOS版本、Android版本、IOS版本

- **为什么Java程序可以跨平台呢？**

  跨平台性的原理是因为在**不同版本的操作系统**中安装有**不同版本的Java虚拟机**，Java程序的运行只依赖于Java虚拟机，和操作系统并没有直接关系。**从而做到一处编译，处处运行**。

![1660147588001](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252332673.png)

### 2.7 JDK环境变量配置

JDK安装后，接下我们来学习一个补充知识，叫做Path环境变量

- **什么是Path环境变量？**

  Path环境变量是让系统程序的路径，方便程序员在命令行窗口的任意目录下启动程序；

- **如何配置环境变量呢？**

  比如把QQ的启动程序，配置到Path环境变量下就可以在任意目录下启动QQ，按照一下步骤操作。

  **第一步：**先找到QQ启动程序所在的目录`C:\Program Files (x86)\Tencent\QQ\Bin`，复制这个路径

  ![1660538063180](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252332177.png)

  **第二步：**按照下面的步骤，找到Path环境变量。

  首先找到此电脑，右键点击属性，可以按照下面的界面；点击【高级系统设置】，再点击【环境变量】

  ![1660538424000](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252333118.png)

  双击Path后，点击新建，把QQ启动目录粘贴进来，不要忘记点确定哦^_^

  ![1660538760744](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252333916.png)

  **第三步：**配置好之后，检查是否配置成功

  ```java
  1.Win+R 输入cmd回车，打开命令行窗口
  2.输入QQScLanucher，可以看到QQ启动了
  ```

![1660539146158](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252333780.png)

----

- **将JDK配置到Path路径下**

  上面我们配置了QQ的启动目录到Path环境变量位置，那么接下来，我们把JDK的bin目录配置到Path环境变量下，这样就可以在任意目录下启动javac和java命令来完成编译和运行了。

  **第一步：**找到JDK的bin目录`C:\Program Files\Java\jdk-17.0.3\bin`，复制一下

  **第二步：**将JDK的bin目录粘贴在Path环境变量后面

  ![1660539632325](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252333994.png)

  **第三步：检测否配置成功**

  ```java
  1.按住Win+R输入cmd 回车，打开命令行创建
  2.输入javac -version 看提示信息是否显示你安装JDK的版本号
    输入java -version 看提示信息是否显示你安装JDK的版本号
  【如果显示版本号都是JDK17就表示配置安装成功】
  ```

  ![1660539955302](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202409252333468.png)

按照前面的操作到这里，就说明JDK环境变量已经配置好了，后面使用JDK命令可以在任意目录下运行。
