## 一、面向对象入门

各位同学，为什么说面向对象是Java最核心的课程呢？因为写Java程序是有套路的，而面向对象就是写Java程序的套路；你如果不知道面向对象编程，那么你Java语言就算白学了。

那这种编程套路是咋回事呢？ 接下来，我们通过一个案例快速的认识一下。

现在假设我们需要处理的是学生的姓名、语文成绩、数学成绩这三个数据，要求打印输出这个学生的总成绩，和平均成绩。

<img src="https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022013152.png" alt="1662209848898" style="zoom:50%;" />

遇到这样的需求，我们以前都会定义方法来做，如下图所示

注意：这里每一个方法有**三个参数**

![1662209886046](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022013055.png)

![1662209899008](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022013335.png)

定义好方法之后，我们调用方法的时候，需要给每一个方法**传递三个实际参数**

![1662210110729](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022013208.png)

在上面案例中，这种编程方式是一种面向过程的编程方式。所谓面向过程，就是编写一个的方法，有数据要进行处理就交给方法来处理。

但是实际上**姓名、语文成绩、数学成绩三个数据可以放在一起，组合成一个对象**，然后让对象提供方法对自己的数据进行处理。这种方式称之为面向对象编程。

![1662210587156](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022013758.png)

**总结一些：所谓编写对象编程，就是把要处理的数据交给对象，让对象来处理。**



## 二、深刻认识面向对象

好的各位同学，在上一节课我们已经用面向对象的编程套路，处理了学生数据。接下来我们就要搞清楚，面向对象的几个最核心问题了。

<img src="https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022013412.png" alt="1662211382446" style="zoom: 33%;" />

我们把这三个问题搞明白，那么你对面向对象的理解就很到位。 

### 2.1 面向对象编程有什么好处？

先来看第一个问题，面向对象编程到底有什么好处呢？ 那就不得不谈，Java的祖师爷对这个世界的理解了。

Java的祖师爷，詹姆斯高斯林认为，在这个世界中 **万物皆对象！**任何一个对象都可以包含一些数据，数据属于哪个对象，就由哪个对象来处理。

这样的话，只要我们找到了对象，其实就找到了对数据的处理方式。

![1662211620054](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022014720.png)

所以面向对象编程的好处，用一句话总结就是：面向对象的开发更符合人类的思维习惯，让编程变得更加简单、更加直观。



### 2.2 程序中对象到底是个啥？

说完面向对象编程有什么好处之后，这里有同学可能会有问题了，你刚才举的例子中，“汽车”、“手机”、“蔡徐坤”是一个实实在在的东西，你说是一个对象好理解。那我们程序中的对象到底是个啥呢？

**对象实质上是一种特殊的数据结构**。这种结构怎么理解呢？

你可以把对象理解成一张表格，表当中记录的数据，就是对象拥有的数据。

![1662212402342](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022014434.png)

这就是程序中的对象到底是个啥！ **一句话总结，对象其实就是一张数据表，表当中记录什么数据，对象就处理什么数据。**



### 2.3 对象是怎么出来的？

刚刚我们讲到对象就是一张数据表，那么这个数据表是怎么来的呢？这张表是不会无缘无故存在的，因为Java也不知道你这个对象要处理哪些数据，所以这张表需要我们设计出来。

用什么来设计这张表呢？就是类（class），**类可以理解成对象的设计图**，或者对象的模板。

![1662213156309](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022014321.png)

我们需要按照对象的设计图创造一个对象。**设计图中规定有哪些数据，对象中就只能有哪些数据。**

![1662213268590](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022014087.png)

**一句话总结：对象可以理解成一张数据表，而数据表中可以有哪些数据，是有类来设计的。**



## 三、对象在计算机中的执行原理

各位同学，前面我们已经带同学写了面向对象的代码，也知道对象到底是咋回事。如果我们再搞清楚对象在计算机中的执行原理，那我们对面向对象的理解就更加专业了。

按照我们之前讲的数组的执行原理，数组变量记录的其实数数组在堆内存中的地址。其实面向对象的代码执行原理和数组的执行原理是非常类似的。

其实`Student s1 = new Student();`这句话中的原理如下

- `Student s1`表示的是在栈内存中，创建了一个Student类型的变量，变量名为s1

- 而`new Student()`会在堆内存中创建一个对象，而对象中包含学生的属性名和属性值

  同时系统会为这个Student对象分配一个地址值0x4f3f5b24

- 接着把对象的地址赋值给栈内存中的变量s1，通过s1记录的地址就可以找到这个对象

- 当执行`s1.name=“播妞”`时，其实就是通过s1找到对象的地址，再通过对象找到对象的name属性，再给对象的name属性赋值为`播妞`;  

搞明白`Student s1 = new Student();`的原理之后，`Student s2 = new Student();`原理完全一样，只是在堆内存中重新创建了一个对象，又有一个新的地址。`s2.name`是访问另对象的属性。

![1662213744520](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022014505.png)



## 四、类和对象的一些注意事项

各位同学，前面几节课我们已经入门了。接下来，关于面向对象有一些细枝末节的东西需要给大家交代一下。

我把这些注意事项已经列举在下面了，我们把几个不好理解的解释一下就可以了（标记方框），其他的大大家一看就能理解。

![1662213891968](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022014144.png)

> **第一条**：一个代码文件中，可以写多个class类，但是只能有一个是public修饰，且public修饰的类必须和文件名相同。

假设文件名为`Demo1.java`，这个文件中假设有两个类`Demo1类和Student类`，代码如下

```java
//public修饰的类Demo1，和文件名Demo1相同
public class Demo1{
    
}

class Student{
    
}
```

> **第二条：**对象与对象之间的数据不会相互影响，但是多个变量指向同一个对象会相互影响。

如下图所示，s1和s2两个变量分别记录的是两个对象的地址值，各自修改各自属性值，是互不影响的。

![1662214650611](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022014013.png)

如下图所示，s1和s2两个变量记录的是同一个对象的地址值，s1修改对象的属性值，再用s2访问这个属性，会发现已经被修改了。

![1662215061486](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022014333.png)



## 五、this关键字

各位同学，接下来我们学习几个面向对象的小知识点，这里我们先认识一下this关键字是什么含义，再说一下this的应用场景。

**this是什么呢？**

this就是一个变量，用在方法中，可以拿到当前类的对象。

我们看下图所示代码，通过代码来体会这句话到底是什么意思。**哪一个对象调用方法方法中的this就是哪一个对象**

![1662301823320](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022015402.png)

上面代码运行结果如下

![1662302089326](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022015052.png)

**this有什么用呢？**

通过this在方法中可以访问本类对象的成员变量。我们看下图代码，分析打印结果是多少

![1662303254161](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022015310.png)

分析上面的代码`s3.score=325`，调用方法printPass方法时，方法中的`this.score`也是325； 而方法中的参数score接收的是250。执行结果是

![1662303676092](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022015189.png)

关于this关键字我们就学习到这里，重点记住这句话：**哪一个对象调用方法方法中的this就是哪一个对象**



## 六、构造器

好同学们，接下来我们学习一个非常实用的语法知识——叫做构造器。

关于构造器，我们掌握下面几个问题就可以了：

1. 什么是构造器？
2. 掌握构造器的特点？
3. 构造器的应用场景？
4. 构造器有哪些注意事项？

我们一个问题一个问题的来学习，先来学习什么是构造器？

- **什么是构造器？**

  构造器其实是一种特殊的方法，但是这个方法没有返回值类型，方法名必须和类名相同。

  如下图所示：下面有一个Student类，构造器名称也必须叫Student；也有空参数构造器，也可以有有参数构造器。

![1662304435504](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022015953.png)

认识了构造器之后，接着我们看一下构造器有什么特点。

- **构造器的特点？**

  在创建对象时，会调用构造器。

  也就是说 `new Student()`就是在执行构造器，当构造器执行完了，也就意味着对象创建成功。 

  ![1662304779863](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022015949.png)

  当执行`new Student("播仔",99)`创建对象时，就是在执行有参数构造器，当有参数构造器执行完，就意味着对象创建完毕了。

  ![1662304859276](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022015660.png)

关于构造器的特点，我们记住一句话：**new 对象就是在执行构造方法**

- **构造器的应用场景？**

  其实构造器就是用来创建对象的。可以在创建对象时给对象的属性做一些初始化操作。如下图所示：

![1662305406056](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022015561.png)

- **构造器的注意事项？**

  学习完构造器的应用场景之后，接下来我们再看一下构造器有哪些注意事项。

  ```java
  1.在设计一个类时，如果不写构造器，Java会自动生成一个无参数构造器。
  2.一定定义了有参数构造器，Java就不再提供空参数构造器，此时建议自己加一个无参数构造器。
  ```

关于构造器的这几个问题我们再总结一下。掌握这几个问题，构造方法就算完全明白了。

```java
1.什么是构造器？
	答：构造器其实是一种特殊的方法，但是这个方法没有返回值类型，方法名必须和类名相			同。
	
2.构造器什么时候执行？
	答：new 对象就是在执行构造方法；

3.构造方法的应用场景是什么？
	答：在创建对象时，可以用构造方法给成员变量赋值

4.构造方法有哪些注意事项？
	1)在设计一个类时，如果不写构造器，Java会自动生成一个无参数构造器。
	2)一定定义了有参数构造器，Java就不再提供空参数构造器，此时建议自己加一个无参数构		造器。
```



## 七、封装性

各位同学，接下来我们再学习一个面向对象很重要的特征叫做——封装性。

**1. 什么是封装呢？**

所谓封装，就是用类设计对象处理某一个事物的数据时，应该把要处理的数据，以及处理数据的方法，都设计到一个对象中去。

比如：在设计学生类时，把学生对象的姓名、语文成绩、数学成绩三个属性，以及求学生总分、平均分的方法，都封装到学生对象中来。

![1662305928023](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022015515.png)

现在我们已经知道什么是封装了。那我们学习封装，学习个啥呢？  其实在实际开发中，在用类设计对事处理的数据，以及对数据处理的方法时，是有一些设计规范的。

封装的设计规范用8个字总结，就是：**合理隐藏、合理暴露**

比如，设计一辆汽车时，汽车的发动机、变速箱等一些零件并不需要让每一个开车的知道，所以就把它们隐藏到了汽车的内部。

把发动机、变速箱等这些零件隐藏起来，这样做其实更加安全，因为并不是所有人都很懂发动机、变速箱，如果暴露在外面很可能会被不懂的人弄坏。

![1662306602412](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022015254.png)

在设计汽车时，除了隐藏部分零件，但是还是得合理的暴露一些东西出来，让司机能够操纵汽车，让汽车跑起来。比如：点火按钮啊、方向盘啊、刹车啊、油门啊、档把啊... 这些就是故意暴露出来让司机操纵汽车的。

![1662306879230](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022016712.png)

好了，到现在我们已经理解什么是封装的一些规范了。就是：**合理暴露、合理隐藏**



**2. 封装在代码中的体现**

知道什么是封装之后，那封装在代码中如何体现呢？一般我们在设计一个类时，会将成员变量隐藏，然后把操作成员变量的方法对外暴露。

这里需要用到一个修饰符，叫private，**被private修饰的变量或者方法，只能在本类中被访问。**

如下图所示，`private double score;` 就相当于把score变量封装在了Student对象的内部，且不对外暴露，你想要在其他类中访问score这个变量就，就不能直接访问了；

如果你想给Student对象的score属性赋值，得调用对外暴露的方法`setScore(int score)`，在这个方法中可以对调用者传递过来的数据进行一些控制，更加安全。

![1662307191295](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022016487.png)

当你想获取socre变量的值时，就得调用对外暴露的另一个方法 `getScore()` 

关于封装我们就学习到这里了。



## 八、实体JavaBean

接下来，我们学习一个面向对象编程中，经常写的一种类——叫实体JavaBean类。我们先来看什么是实体类？

**1. 什么是实体类？**

实体类就是一种特殊的类，它需要满足下面的要求：

![1662335204398](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022016433.png)

接下来我们按照要求，写一个Student实体类；

![1662335451401](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022016502.png)

写完实体类之后，我们看一看它有什么特点？ 其实我们会发现实体类中除了有给对象存、取值的方法就没有提供其他方法了。所以实体类仅仅只是用来封装数据用的。

知道实体类有什么特点之后，接着我们看一下它有哪些应用场景？

**2. 实体类的应用场景**

在实际开发中，实体类仅仅只用来封装数据，而对数据的处理交给其他类来完成，以实现数据和数据业务处理相分离。如下图所示

![1662336287570](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022016447.png)

在实际应用中，会将类作为一种数据类型使用。如下图所示，在StudentOperator类中，定义一个Student类型的成员变量student，然后使用构造器给student成员变量赋值。

然后在Student的printPass()方法中，使用student调用Student对象的方法，对Student对象的数据进行处理。

![1662337507608](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022016099.png)

到这里，我们已经学习了JavaBean实体类的是什么，以及它的应用场景，我们总结一下

```java
1.JavaBean实体类是什么？有啥特点
	JavaBean实体类，是一种特殊的；它需要私有化成员变量，有空参数构造方法、同时提供		getXxx和setXxx方法；
	
	JavaBean实体类仅仅只用来封装数据，只提供对数据进行存和取的方法
 
2.JavaBean的应用场景？
	JavaBean实体类，只负责封装数据，而把数据处理的操作放在其他类中，以实现数据和数		据处理相分离。
```



## 九、面向对象综合案例

学习完面向对象的语法知识之后。接下来，我们做一个面向对象的综合案例——模仿电影信息系统。

需求如下图所示

	1. 想要展示系统中全部的电影信息（每部电影：编号、名称、价格）
	2. 允许用户根据电影的编号（id），查询出某个电影的详细信息。

![1662351774659](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022016309.png)

运行程序时，能够根据用户的选择，执行不同的功能，如下图所示

<img src="https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022016740.png" alt="1662351990387" style="zoom:50%;" />

按照下面的步骤来完成需求

### 1. 第一步：定义电影类

首先每一部电影，都包含这部电影的相关信息，比如：电影的编号（id）、电影的名称（name）、电影的价格（price）、电影的分数（score）、电影的导演（director）、电影的主演（actor）、电影的简介（info）。 

为了去描述每一部电影，有哪些信息，我们可以设计一个电影类（Movie），电影类仅仅只是为了封装电影的信息，所以按照JavaBean类的标准写法来写就行。

```java
public class Movie {
    private int id;
    private String name;
    private double price;
    private double score;
    private String director;
    private String actor;
    private String info;

    public Movie() {
    }

    public Movie(int id, String name, double price, double score, String director, String actor, String info) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.score = score;
        this.director = director;
        this.actor = actor;
        this.info = info;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    public double getScore() {
        return score;
    }

    public void setScore(double score) {
        this.score = score;
    }

    public String getDirector() {
        return director;
    }

    public void setDirector(String director) {
        this.director = director;
    }

    public String getActor() {
        return actor;
    }

    public void setActor(String actor) {
        this.actor = actor;
    }

    public String getInfo() {
        return info;
    }

    public void setInfo(String info) {
        this.info = info;
    }
}
```

### 2. 第二步：定义电影操作类

前面我们定义的Movie类，仅仅只是用来封装每一部电影的信息。为了让电影数据和电影数据的操作相分离，我们还得有一个电影操作类（MovieOperator）。

因为系统中有多部电影，所以电影操作类中MovieOperator，需要有一个`Movie[] movies;` 用来存储多部电影对象；

同时在MovieOperator类中，提供对外提供，对电影数组进行操作的方法。如`printAllMovies()`用于打印数组中所有的电影信息，`searchMovieById(int id)`方法根据id查找一个电影的信息并打印。

```java
public class MovieOperator {
    //因为系统中有多部电影，所以电影操作类中，需要有一个Movie的数组
    private Movie[] movies;
    public MovieOperator(Movie[] movies){
        this.movies = movies;
    }

    /** 1、展示系统全部电影信息 movies = [m1, m2, m3, ...]*/
    public void printAllMovies(){
        System.out.println("-----系统全部电影信息如下：-------");
        for (int i = 0; i < movies.length; i++) {
            Movie m = movies[i];
            System.out.println("编号：" + m.getId());
            System.out.println("名称：" + m.getName());
            System.out.println("价格：" + m.getPrice());
            System.out.println("------------------------");
        }
    }

    /** 2、根据电影的编号查询出该电影的详细信息并展示 */
    public void searchMovieById(int id){
        for (int i = 0; i < movies.length; i++) {
            Movie m = movies[i];
            if(m.getId() == id){
                System.out.println("该电影详情如下：");
                System.out.println("编号：" + m.getId());
                System.out.println("名称：" + m.getName());
                System.out.println("价格：" + m.getPrice());
                System.out.println("得分：" + m.getScore());
                System.out.println("导演：" + m.getDirector());
                System.out.println("主演：" + m.getActor());
                System.out.println("其他信息：" + m.getInfo());
                return; // 已经找到了电影信息，没有必要再执行了
            }
        }
        System.out.println("没有该电影信息~");
    }
}
```



### 3. 第三步：定义测试类

最后，我们需要在测试类中，准备好所有的电影数据，并用一个数组保存起来。每一部电影的数据可以封装成一个对象。然后把对象用数组存起来即可。

```java
public class Test {
    public static void main(String[] args) {
        //创建一个Movie类型的数组
        Movie[] movies = new Movie[4];
        //创建4个电影对象，分别存储到movies数组中
        movies[0] = new Movie(1,"水门桥", 38.9, 9.8, "徐克", "吴京","12万人想看");
        movies[1] = new Movie(2, "出拳吧", 39, 7.8, "唐晓白", "田雨","3.5万人想看");
        movies[2] = new Movie(3,"月球陨落", 42, 7.9, "罗兰", "贝瑞","17.9万人想看");
        movies[3] = new Movie(4,"一点就到家", 35, 8.7, "许宏宇", "刘昊然","10.8万人想看");
        
    }
}
```

准备好测试数据之后，接下来就需要对电影数据进行操作。我们已经把对电影操作先关的功能写到了MovieOperator类中，所以接下来，创建MovieOperator类对象，调用方法就可以完成相关功能。

继续再main方法中，接着写下面的代码。

```java
// 4、创建一个电影操作类的对象，接收电影数据，并对其进行业务处理
MovieOperator operator = new MovieOperator(movies);
Scanner sc = new Scanner(System.in);
while (true) {
    System.out.println("==电影信息系统==");
    System.out.println("1、查询全部电影信息");
    System.out.println("2、根据id查询某个电影的详细信息展示");
    System.out.println("请您输入操作命令：");
    int command = sc.nextInt();
    switch (command) {
        case 1:
            // 展示全部电影信息
            operator.printAllMovies();
            break;
        case 2:
            // 根据id查询某个电影的详细信息展示
            System.out.println("请您输入查询的电影id:");
            int id = sc.nextInt();
            operator.searchMovieById(id);
            break;
        default:
            System.out.println("您输入的命令有问题~~");
    }
}
```

到这里，电影信息系统就完成了。 小伙伴们，自己尝试写一下吧！！



## 十、成员变量和局部变量的区别

各位同学，面向对象的基础内容咱们已经学习完了。同学们在面向对象代码时，经常会把成员变量和局部变量搞混。所以现在我们讲一讲他们的区别。

![1662371089114](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022016929.png)

如下图所示，成员变量在类中方法外，而局部变量在方法中。

<img src="https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410022016986.png" alt="1662353340190" style="zoom: 67%;" />

----

到这里，我们关于面向对象的基础知识就学习完了。**面向对象的核心点就是封装，将数据和数据的处理方式，都封装到对象中； 至于对象要封装哪些数据？对数据进行怎样的处理？ 需要通过类来设计。**

需要注意的是，不同的人，对同一个对象进行设计，对象封装那些数据，提供哪些方法，可能会有所不同；只要能够完成需求，符合设计规范，都是合理的设计。


