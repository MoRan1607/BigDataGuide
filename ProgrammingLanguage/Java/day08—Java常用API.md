# day08—Java常用API

## 一、今日内容介绍、API概述

各位同学，我们前面已经学习了面向对象编程，使用面向编程这个套路，我们需要自己写类，然后创建对象来解决问题。但是在以后的实际开发中，更多的时候，我们是利用面向编程这种套路，使用别人已经写好的类来编程的。

这就是我们今天要学习的内容——常用API（全称是Application Program Interface 应用程序接口），说人话就是：**别人写好的一些程序，给咱们程序员直接拿去调用。**

Java官方其实已经给我们写好了很多很多类，每一个类中又提供了一系列方法来解决与这个类相关的问题。

- 比如String类，表示字符串，提供的方法全都是对字符串操作的。
- 比如ArrayList类，表示一个容器，提供的方法都是对容器中的数据进行操作的。

像这样的类还有很多，Java把这些类是干什么用的、类中的每一个方法是什么含义，编写成了文档，我们把这个文档称之为API文档。

![1662602386634](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022141959.png)



**1. 我们为什么要学习别人写好的程序呢？**

​		在行业中有这么一句话：“不要重复造轮子”。这里所说的轮子就是别人已经写过的程序。意思就是不要写重复的程序，因为程序是用来解决问题的，如果这个问题别人已经解决过，并且这个解决方案也得到了市场认可，那就不用再自己重复写这个程序了。

​		Java已经发展了20多年，在这20多年里，已经积累类了很多问题的解决方案，基本上现在我们遇到的问题，在这20多年里，早就有人解决过。

​		所以我们把面向对象的高级知识学习完之后，Java语言的语法知识就已经学习完了。剩下的所有内容都是是学习一个一个的API，通过调用API提供的方法来解决实际问题。

**2. 我们要学习哪些API**

Java的API文档中，有那么多的类，是所有的类都要学习吗？并不是 ，虽然Java提供了很多个类，但是并不是所有类都得到了市场认可，我们只学习一些在工作中常用的就行。

除了Java官方提供的API，还一些第三方的公司或者组织也会提供一些API，甚至比Java官方提供的API更好用，在需要的时候我们也会告诉大家怎么用。



**3. 今天我们主要学习两个类，一个是String类、还有一个是ArrayList类。**

![1662605214383](assets/1662605214383.png)

字符串的应用场景是非常多的，可以说是无处不在。

比如，在用户登录时，需要对用户名和密码进行校验，这里的用户名和密码都是String

![1662605347797](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022141576.png)

再比如，在和网友聊天时，其实输入的文字就是一个一个字符串

![1662605396550](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022141749.png)

再比如，在百度上搜索时，搜素的关键词，也是字符串

![1662605442842](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022141127.png)

学习完String类之后，还会学习一个类ArrayList

![1662605519698](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022141088.png)

大家知道数组是一个容器，有数组干嘛还要集合呢？	因为数字的长度是固定的，一旦创建不可改变。

比如数组的长度为3，想要存储第4个元素就存不进去了。

![1662605575865](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022141052.png)

使用集合就可以解决上面的问题，集合可以根据需要想存多少个元素就存多少个元素。



## 二、包

**1. 什么是包**

在学习API类之前，我们先要学习包。因为Java官方提供的类有很多，为了对这些类进行分门别类的管理，别人把写好的类都是放在不同的包里的。

包其实类似于文件夹，一个包中可以放多个类文件。如下图所示

![1662605881879](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022142185.png)

建包的语法格式：

```java
//类文件的第一行定义包
package com.itheima.javabean;

public class 类名{
    
}
```



**2. 在自己的程序中，调用其他包中的程序，需要注意下面一个问题**

- 如果当前程序中，要调用自己所在包下的其他程序，可以直接调用。（同一个包下的类，互相可以直接调用）

- 如果当前程序中，要调用其他包下的程序，则必须在当前程序中导包, 才可以访问！

  导包格式：` import 包名.类名`

- 如果当前程序中，要调用Java.lang包下的程序，不需要我们导包的，可以直接使用。

- 如果当前程序中，要调用多个不同包下的程序，而这些程序名正好一样，此时默认只能导入一个程序，另一个程序必须带包名访问。



## 三、String类

### 1. String类概述

各位同学，接下来我们学习String这个类，也就是学对字符串进行处理。为什么要学习字符串处理呢？因为在开发中对于字符串的处理还是非常常见的。

比如：在用户登录时，用户输入的用户名和密码送到后台，需要和正确的用户名和密码进行校验，这就需要用到String类提供的比较功能。

![1662605347797](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022142969.png)

再比如：同学们在直播留言时，有些小伙伴可能不太文明说了一些脏话，后台检测到你输入的是脏话，就会用`***`把脏话屏蔽掉。这也需要用到String类提供的替换功能

![1662605396550](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022142199.png)

Java为了方便我们处理字符串，所以给我们提供了一个String类来代表字符串，这个类就是`java.lang`包下。

按照面向对象的编程思想，对于字符串的操作，只需要创建字符串对象，用字符串对象封装字符串数据，然后调用String类的方法就可以了。

![1662607669465](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022142965.png)

----



### 2. String创建对象

接下来我们打开String类的API，看一下String类的对象如何创建。如下图所示

![1662607801186](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022142410.png)

String类的API中，有这么一句话：“Java程序中的所有字符串字面值（如"abc"）都是字符串的实例实现”。这里所说的实例实现，其实指的就是字符串对象。

意思就是：**所有Java的字符串字面值，都是字符串对象。**

- 所以创建String对象的第一种方式就有了

```java
String s1 = "abc"; //这里"abc"就是一个字符串对象，用s1变量接收

String s2 = "黑马程序员"; //这里的“黑马程序员”也是一个字符串对象，用s2变量接收
```

- 创建String对象还有第二种方式，就是利用String类的构造方法创建String类的对象。

![1662608166502](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022142819.png)

我们前面学习过类的构造方法，执行构造方法需要用到new关键字。`new String(参数)`就是在执行String类的构造方法。 

下面我们演示通过String类的构造方法，创建String类的对象

```java
// 1、直接双引号得到字符串对象，封装字符串数据
String name = "黑马666";
System.out.println(name);

// 2、new String创建字符串对象，并调用构造器初始化字符串
String rs1 = new String();
System.out.println(rs1); // ""

String rs2 = new String("itheima");
System.out.println(rs2);

char[] chars = {'a', '黑', '马'};
String rs3 = new String(chars);
System.out.println(rs3);

byte[] bytes = {97, 98, 99};
String rs4 = new String(bytes);
System.out.println(rs4);
```

关于String类是用来干什么的，以及String类对象的创建我们就学习到这里。最后总结一下

```java
1. String是什么，可以做什么？
	答：String代表字符串，可以用来创建对象封装字符串数据，并对其进行处理。

2.String类创建对象封装字符串数据的方式有几种？
	方式一： 直接使用双引号“...” 。
	方式二：new String类，调用构造器初始化字符串对象。
```



### 3. String类的常用方法

各位同学，在上一节课中，我们学习了如何通过字符串对象封装数据，接下来我们学习调用String类的方法对象字符串数据进行处理。

这里已经将String类的常用方法，给同学们挑出来了，我们先快速的认识一下。为什么是快速认识一下呢？因为API真正的作用是来解决业务需求的，如果不解决业务需求，只是记API是很难记住的。

![1662609378727](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022142414.png)

所以API的正确打开方式是，先找到这个类，把这个类中的方法先用代码快速过一遍，有一个大概印象就行。然后再具体的案例中，选择你需要的方法来用就行。

下面我们就把String类中的方法，按照方法的调用规则，先快速过一遍。（注意：第一次调用API方法，都是看着API方法来调用用的，不是背的）

```java
public class StringDemo2 {
    public static void main(String[] args) {
        //目标：快速熟悉String提供的处理字符串的常用方法。
        String s = "黑马Java";
        // 1、获取字符串的长度
        System.out.println(s.length());

        // 2、提取字符串中某个索引位置处的字符
        char c = s.charAt(1);
        System.out.println(c);

        // 字符串的遍历
        for (int i = 0; i < s.length(); i++) {
            // i = 0 1 2 3 4 5
            char ch = s.charAt(i);
            System.out.println(ch);
        }

        System.out.println("-------------------");

        // 3、把字符串转换成字符数组，再进行遍历
        char[] chars = s.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            System.out.println(chars[i]);
        }

        // 4、判断字符串内容，内容一样就返回true
        String s1 = new String("黑马");
        String s2 = new String("黑马");
        System.out.println(s1 == s2); // false
        System.out.println(s1.equals(s2)); // true

        // 5、忽略大小写比较字符串内容
        String c1 = "34AeFG";
        String c2 = "34aEfg";
        System.out.println(c1.equals(c2)); // false
        System.out.println(c1.equalsIgnoreCase(c2)); // true

        // 6、截取字符串内容 (包前不包后的)
        String s3 = "Java是最好的编程语言之一";
        String rs = s3.substring(0, 8);
        System.out.println(rs);

        // 7、从当前索引位置一直截取到字符串的末尾
        String rs2 = s3.substring(5);
        System.out.println(rs2);

        // 8、把字符串中的某个内容替换成新内容，并返回新的字符串对象给我们
        String info = "这个电影简直是个垃圾，垃圾电影！！";
        String rs3 = info.replace("垃圾", "**");
        System.out.println(rs3);

        // 9、判断字符串中是否包含某个关键字
        String info2 = "Java是最好的编程语言之一，我爱Java,Java不爱我！";
        System.out.println(info2.contains("Java"));
        System.out.println(info2.contains("java"));
        System.out.println(info2.contains("Java2"));

        // 10、判断字符串是否以某个字符串开头。
        String rs4 = "张三丰";
        System.out.println(rs4.startsWith("张"));
        System.out.println(rs4.startsWith("张三"));
        System.out.println(rs4.startsWith("张三2"));

        // 11、把字符串按照某个指定内容分割成多个字符串，放到一个字符串数组中返回给我们
        String rs5 = "张无忌,周芷若,殷素素,赵敏";
        String[] names = rs5.split(",");
        for (int i = 0; i < names.length; i++) {
            System.out.println(names[i]);
        }
    }
}
```

演示完String类的这些方法之后，我们对字符串有哪些方法，就已经有一个大致印象了。至少知道String字符串能干哪些事情。

至于String类的这些方法是否都记住了，这个还需要通过一些案例训练，在用的过程中去找哪个方法能够解决你的实际需求，就用哪个方法。同一个方法用的次数多个，自然就记住了。



### 4. String的注意事项

在上一节，我们学习了字符串的一些常用方法，在实际工作中用这些方法解决字符串的常见问题是完全足够的，但是在面试时可能会问一些原理性的东西。

所以把字符串原理性的内容，就当做注意事项来学习一下。一共有下面的2点：

![1662610060051](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022142468.png)

- **注意事项1：String类的对象是不可变的对象**

我们先看一段代码，分析这段代码的结果

![1662610347618](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022142114.png)

以上代码中，先定义了一个String变量 name第一次赋值为`“黑马”;` 然后对`name`变量记录的字符串进行两次拼接，第一次拼接`“程序员”`，第二次拼接`“播妞”`；我们发现得到的结果是：`黑马程序员播妞`

这里问题就来了，你不是是说：**String类的对象是不可变的字符串对象吗？**我看name的值变了呀！！！

![1662610591674](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022142924.png)



下面我们就解释一下，String是不可变对象到底是什么含义。

需要注意的是：只要是以`“”`方式写出的字符串对象，会在堆内存中的**字符串常量池**中存储。

执行第一句话时，会在堆内存的常量池中，创建一个字符串对象`“黑马”`，然后把`“黑马”`的地址赋值给`String name`

![1662610697641](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022143636.png)

当执行第二句话时，又会再堆内存的常量池中创建一个字符串`“程序员”`，和`“黑马”`拼接，拼接之后还会产生一个新的字符串对象`”黑马程序员“`，然后将新产生的`“黑马程序员”`对象的地址赋值给`String name`变量。

![1662610978351](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022143775.png)

此时你会发现，之前创建的字符串对象`“黑马”`内容确实是没有改变的。所以说String的对象是不可变的。



- **注意事项2：字符串字面量和new出来字符串的区别**
  1. 只要是以`“...”`方式写出的字符串对象，会存储到字符串常量池，且相同内容的字符串只存储一份。如下图一所示
  2. 但通过`new`方式创建字符串对象，每new一次都会产生一个新的对象放在堆内存中。如下图二所示

![1662618688215](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022143981.png)

![1662618651517](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022143367.png)

- 总结一下，字符串的注意事项。

```java
1. String是不可变字符串对象
2. 只要是以“...”方式写出的字符串对象，会存储到字符串常量池，且相同内容的字符串只存储一份；
3. 但通过new方式创建字符串对象，每new一次都会产生一个新的对象放在堆内存中。
```



### 5. String案例一：用户登录案例

接下来给大家做一个案例，使用字符串的功能完成登录案例。案例需求如下：

![1662618819077](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022143444.png)

```java
分析一下完成案例的步骤：
	1.首先，从登录界面上可以得出，需要让用户输入登录名和密码
	2.设计一个登录方法，对用户名和密码进行校验
	3.调用登录方法，根据方法的返回结果，判断登录是否成功。
	4.如果登录失败，循环登录3次，结束循环；如果登录成功，跳出循环;
```

案例分析的步骤完成代码

```java
/**
   目标：完成用户的登录案例。
 */
public class StringTest4 {
    public static void main(String[] args) {
        // 1、开发一个登录界面
        for (int i = 0; i < 3; i++) {
            Scanner sc = new Scanner(System.in);
            System.out.println("请您输入登录名称：");
            String loginName = sc.next();
            System.out.println("请您输入登录密码：");
            String passWord = sc.next();

            // 5、开始调用登录方法，判断是否登录成功
            boolean rs = login(loginName, passWord);
            if(rs){
                System.out.println("恭喜您，欢迎进入系统~~");
                break; // 跳出for循环，代表登录完成
            }else {
                System.out.println("登录名或者密码错误，请您确认~~");
            }
        }
    }

    /**
      2、开发一个登录方法，接收用户的登录名和密码，返回认证的结果
     */
    public static boolean login(String loginName, String passWord){
        // 3、准备一份系统正确的登录名和密码
        String okLoginName = "itheima";
        String okPassWord = "123456";

        // 4、开始正式判断用户是否登录成功
        /*if(okLoginName.equals(loginName) && okPassWord.equals(passWord)){
            // 登录成功的
            return true;
        }else {
            return false;
        }*/
        return okLoginName.equals(loginName) && okPassWord.equals(passWord);
    }
}
```

### 6. String案例二：随机产生验证码

接下来学习一个再工作中也比较常见的案例，使用String来开发验证码。需求如下：

![1662619371060](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022143475.png)

```java
根据需求分析，步骤如下：
	1.首先，设计一个方法，该方法接收一个整型参数，最终要返回对应位数的随机验证码。
	2.方法内定义2个字符串变量：
		1个用来记住生成的验证码，1个用来记住要用到的全部字符。
	3.定义for循环控制生成多少位随机字符
	4.每次得到一个字符范围内的随机索引
	5.根据索引提取该字符，把该字符交给code变量连接起
	6.循环结束后，在循环外返回code即可。
	7.在主方法中调用生成验证码的方法
```

根据步骤完成代码

```java
import java.util.Random;
/**
    目标：完成随机产生验证码，验证码的每位可能是数字、大写字母、小写字母
 */
public class StringTest5 {
    public static void main(String[] args) {
        System.out.println(createCode(4));
        System.out.println(createCode(6));
    }
    /**
       1、设计一个方法，返回指定位数的验证码
     */
    public static String createCode(int n){
        // 2、定义2个变量 
        //一个是记住最终产生的随机验证码 
        //一个是记住可能用到的全部字符
        String code = "";
        String data = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

        Random r = new Random();
        // 3、开始定义一个循环产生每位随机字符
        for (int i = 0; i < n; i++) {
            // 4、随机一个字符范围内的索引。
            int index = r.nextInt(data.length());
            // 5、根据索引去全部字符中提取该字符
            code += data.charAt(index); // code = code + 字符
        }
        // 6、返回code即可
        return code;
    }
}
```

关于String的案例，我们先练习到这里。以后遇到对字符串进行操作的需求，优先找String类有没有提供对应的方法。



## 四、ArrayList类

### 1. ArrayList快速入门

学习完String类之后，接下来再学习一个类——叫ArrayList。 

ArrayList表示一种集合，它是一个容器，用来装数据的，类似于数组。那有了数组，为什么要有集合呢？

因为数组一旦创建大小不变，比如创建一个长度为3的数组，就只能存储3个元素，想要存储第4个元素就不行。而集合是大小可变的，想要存储几个元素就存储几个元素，在实际工作中用得更多。

然后集合有很多种，而ArrayList只是众多集合中的一种，跟多的集合我们在就业班的课程中再学习。如下图所示：

![1662620084702](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022143957.png)

集合该怎么学呢？1. 首先你要会创建集合对象，2. 然后能够调用集合提供的方法对容器中的数据进行增删改查，3. 最后知道集合的一些特点就可以了。

![1662620152564](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022143834.png)



### 2. ArrayList常用方法

想要使用ArrayList存储数据，并对数据进行操作：

- 第一步：创建ArrayList容器对象。一般使用空参数构造方法，如下图所示：

- 第二步：调用ArrayList类的常用方法对容器中的数据进行操作。常用方法如下：

![1662620389155](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022143149.png)

接下来我们把ArrayList集合的这些方法快速的熟悉一下：

```java
/**
目标：要求同学们掌握如何创建ArrayList集合的对象，并熟悉ArrayList提供的常用方法。
 */
public class ArrayListDemo1 {
    public static void main(String[] args) {
        // 1、创建一个ArrayList的集合对象
        // ArrayList<String> list = new ArrayList<String>();
        // 从jdk 1.7开始才支持的
        ArrayList<String> list = new ArrayList<>();

        list.add("黑马");
        list.add("黑马");
        list.add("Java");
        System.out.println(list);

        // 2、往集合中的某个索引位置处添加一个数据
        list.add(1, "MySQL");
        System.out.println(list);

        // 3、根据索引获取集合中某个索引位置处的值
        String rs = list.get(1);
        System.out.println(rs);

        // 4、获取集合的大小（返回集合中存储的元素个数）
        System.out.println(list.size());

        // 5、根据索引删除集合中的某个元素值，会返回被删除的元素值给我们
        System.out.println(list.remove(1));
        System.out.println(list);

        // 6、直接删除某个元素值，删除成功会返回true，反之
        System.out.println(list.remove("Java"));
        System.out.println(list);

        list.add(1, "html");
        System.out.println(list);

        // 默认删除的是第一次出现的这个黑马的数据的
        System.out.println(list.remove("黑马"));
        System.out.println(list);

        // 7、修改某个索引位置处的数据，修改后会返回原来的值给我们
        System.out.println(list.set(1, "黑马程序员"));
        System.out.println(list);
    }
}
```

### 3. ArrayList应用案例1

接下来，我们学习一个ArrayList的应用案例，需求如下：

![1662620686208](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022144629.png)

我们分析一下这个案例的步骤该如何实现：

```java
1.用户可以选购多个商品，可以创建一个ArrayList集合，存储这些商品
2.按照需求，如果用户选择了"枸杞"批量删除，应该删除包含"枸杞"的所有元素
	1)这时应该遍历集合中每一个String类型的元素
	2)使用String类的方法contains判断字符串中是否包含"枸杞"
    3)包含就把元素删除
3.输出集合中的元素，看是否包含"枸杞"的元素全部删除
```

按照分析的步骤，完成代码

```java
public class ArrayListTest2 {
    public static void main(String[] args) {
        // 1、创建一个ArrayList集合对象
        ArrayList<String> list = new ArrayList<>();
        list.add("枸杞");
        list.add("Java入门");
        list.add("宁夏枸杞");
        list.add("黑枸杞");
        list.add("人字拖");
        list.add("特级枸杞");
        list.add("枸杞子");
        System.out.println(list);
        //运行结果如下： [Java入门, 宁夏枸杞, 黑枸杞, 人字拖, 特级枸杞, 枸杞子]
       
        // 2、开始完成需求：从集合中找出包含枸杞的数据并删除它
        for (int i = 0; i < list.size(); i++) {
            // i = 0 1 2 3 4 5
            // 取出当前遍历到的数据
            String ele = list.get(i);
            // 判断这个数据中包含枸杞
            if(ele.contains("枸杞")){
                // 直接从集合中删除该数据
                list.remove(ele);
            }
        }
        System.out.println(list);
        //删除后结果如下：[Java入门, 黑枸杞, 人字拖, 枸杞子]
    }
}
```

运行完上面代码，我们会发现，删除后的集合中，竟然还有`黑枸杞`，`枸杞子`在集合中。这是为什么呢？

![1662621705234](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022144845.png)

枸杞子被保留下来，原理是一样的。可以自行分析。

那如何解决这个问题呢？这里打算给大家提供两种解决方案：

- **集合删除元素方式一**：每次删除完元素后，让控制循环的变量`i--`就可以了；如下图所示

![1662622656784](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022144545.png)

具体代码如下：

```java
// 方式一：每次删除一个数据后，就让i往左边退一步
for (int i = 0; i < list.size(); i++) {
    // i = 0 1 2 3 4 5
    // 取出当前遍历到的数据
    String ele = list.get(i);
    // 判断这个数据中包含枸杞
    if(ele.contains("枸杞")){
        // 直接从集合中删除该数据
        list.remove(ele);
        i--;
    }
}
System.out.println(list);
```



- **集合删除元素方式二**：我们只需要倒着遍历集合，在遍历过程中删除元素就可以了

![1662623052476](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022144625.png)

![1662623321970](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022144324.png)

![1662623468659](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022144402.png)

![1662623624269](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022144791.png)

具体代码如下：

```java
// 方式二：从集合的后面倒着遍历并删除
// [Java入门, 人字拖]
//   i
for (int i = list.size() - 1; i >= 0; i--) {
    // 取出当前遍历到的数据
    String ele = list.get(i);
    // 判断这个数据中包含枸杞
    if(ele.contains("枸杞")){
        // 直接从集合中删除该数据
        list.remove(ele);
    }
}
System.out.println(list);
```



### 4. ArrayList应用案例2

各位同学，上一个ArrayList应用案例中，我们往集合存储的元素是String类型的元素，实际上在工作中我们经常往集合中自定义存储对象。

接下来我们做个案例，用来往集合中存储自定义的对象，先阅读下面的案例需求：

![1662623794937](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022145273.png)

分析需求发现：

1. 在外卖系统中，每一份菜都包含，菜品的名称、菜品的原价、菜品的优惠价、菜品的其他信息。那我们就可以定义一个菜品类（Food类），用来描述每一个菜品对象要封装那些数据。
2. 接着再写一个菜品管理类（FoodManager类），提供展示操作界面、上架菜品、浏览菜品的功能。

- 首先我们先定义一个菜品类（Food类），用来描述每一个菜品对象要封装那些数据。

```java
public class Food {
    private String name;	//菜品名称
    private double originalPrice; //菜品原价
    private double specialPrice; //菜品优惠价
    private String info; //菜品其他信息

    public Food() {
    }

    public Food(String name, double originalPrice, double specialPrice, String info) {
        this.name = name;
        this.originalPrice = originalPrice;
        this.specialPrice = specialPrice;
        this.info = info;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getOriginalPrice() {
        return originalPrice;
    }

    public void setOriginalPrice(double originalPrice) {
        this.originalPrice = originalPrice;
    }

    public double getSpecialPrice() {
        return specialPrice;
    }

    public void setSpecialPrice(double specialPrice) {
        this.specialPrice = specialPrice;
    }

    public String getInfo() {
        return info;
    }

    public void setInfo(String info) {
        this.info = info;
    }
}
```

- 接下来写一个菜品管理类，提供**上架菜品的功能、浏览菜品的功能、展示操作界面的功能。**

```java
public class FoodManager{
    //为了存储多个菜品，预先创建一个ArrayList集合；
    //上架菜品时，其实就是往集合中添加菜品对象
    //浏览菜品时，其实就是遍历集合中的菜品对象，并打印菜品对象的属性信息。
    private ArrayList<Food> foods = new ArrayList<>(); 
    //为了在下面的多个方法中，能够使用键盘录入，提前把Scanner对象创建好；
    private Scanner sc = new Scanner(System.in);
   
    /**
     1、商家上架菜品
     */
    public void add(){
        System.out.println("===菜品上架==");
        // 2、提前创建一个菜品对象，用于封装用户上架的菜品信息
        Food food = new Food();
        System.out.println("请您输入上架菜品的名称：");
        String name = sc.next();
        food.setName(name);

        System.out.println("请您输入上架菜品的原价：");
        double originalPrice = sc.nextDouble();
        food.setOriginalPrice(originalPrice);

        System.out.println("请您输入上架菜品的优惠价：");
        double specialPrice = sc.nextDouble();
        food.setSpecialPrice(specialPrice);

        System.out.println("请您输入上架菜品的其他信息：");
        String info = sc.next();
        food.setInfo(info);

        // 3、把菜品对象添加到集合容器中去
        foods.add(food);
        System.out.println("恭喜您，上架成功~~~");
    }

    /**
       2、菜品；浏览功能
     */
    public void printAllFoods(){
        System.out.println("==当前菜品信息如下：==");
        for (int i = 0; i < foods.size(); i++) {
            Food food = foods.get(i);
            System.out.println("菜品名称：" + food.getName());
            System.out.println("菜品原价：" + food.getOriginalPrice());
            System.out.println("菜品优惠价：" + food.getSpecialPrice());
            System.out.println("其他描述信息：" + food.getInfo());
            System.out.println("------------------------");
        }
    }
    /**
    3、专门负责展示系统界面的
    */
    public void start(){
        while (true) {
            System.out.println("====欢迎进入商家后台管理系统=====");
            System.out.println("1、上架菜品（add）");
            System.out.println("2、浏览菜品（query）");
            System.out.println("3、退出系统（exit）");
            System.out.println("请您选择操作命令：");
            String command = sc.next();
            switch (command) {
                case "add":
                    add();
                    break;
                case "query":
                    printAllFoods();
                    break;
                case "exit":
                    return; // 结束当前方法！
                default:
                    System.out.println("您输入的操作命令有误~~");
            }
        }
	}
}
```

- 最后在写一个测试类Test，在测试类中进行测试。其实测试类，只起到一个启动程序的作用。

```java
public class Test {
    public static void main(String[] args) {
        FoodManager manager = new FoodManager();
        manager.start();
    }
}
```

运行结果如下：需要用户输入add、query或者exit，选择进入不同的功能。

![1662624841469](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022145834.png)

好了，如果你能够把这个案例写出来，说明你对面向对象的思维封装数据，以及使用ArrayList容器存储数据，并对数据进行处理这方面的知识已经运用的很熟悉了。

