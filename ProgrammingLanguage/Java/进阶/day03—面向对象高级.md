# day03—面向对象高级

## 一、内部类

 内部类是类中的五大成分之一（成员变量、方法、构造器、内部类、代码块），如果一个类定义在另一个类的内部，这个类就是内部类。

当一个类的内部，包含一个完整的事物，且这个事物没有必要单独设计时，就可以把这个事物设计成内部类。

比如：汽车、的内部有发动机，发动机是包含在汽车内部的一个完整事物，可以把发动机设计成内部类。

```java
public class Car{
	//内部类
    public class Engine{
        
    }
}
```

内部类有四种形式，分别是成员内部类、静态内部类、局部内部类、匿名内部类。

我们先来学习成员内部类

### 1.1 成员内部类

成员内部类就是类中的一个普通成员，类似于成员变量、成员方法。

```java
public class Outer {
    private int age = 99;
    public static String a="黑马";

    // 成员内部类
    public class Inner{
        private String name;
        private  int age = 88;

        //在内部类中既可以访问自己类的成员，也可以访问外部类的成员
        public void test(){
            System.out.println(age); //88
            System.out.println(a);   //黑马

            int age = 77;
            System.out.println(age); //77
            System.out.println(this.age); //88
            System.out.println(Outer.this.age); //99
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }
    }
}
```

成员内部类如何创建对象，格式如下

```java
//外部类.内部类 变量名 = new 外部类().new 内部类();
Outer.Inner in = new Outer().new Inner();
//调用内部类的方法
in.test();
```

总结一下内部类访问成员的特点

- 既可以访问内部类成员、也可以访问外部类成员
- 如果内部类成员和外部类成员同名，可以使用**`类名.this.成员`**区分



### 1.2 静态内部类

静态内部类，其实就是在成员内部类的前面加了一个static关键字。静态内部类属于外部类自己持有。

```java
public class Outer {
    private int age = 99;
    public static String schoolName="黑马";

    // 静态内部类
    public static class Inner{
        //静态内部类访问外部类的静态变量，是可以的；
        //静态内部类访问外部类的实例变量，是不行的
        public void test(){
            System.out.println(schoolName); //99
            //System.out.println(age);   //报错
        }
    }
}
```

静态内部类创建对象时，需要使用外部类的类名调用。

```java
//格式：外部类.内部类 变量名 = new 外部类.内部类();
Outer.Inner in = new Outer.Inner();
in.test();
```



### 1.3 局部内部类

局部内部类是定义在方法中的类，和局部变量一样，只能在方法中有效。所以局部内部类的局限性很强，一般在开发中是不会使用的。

```java
public class Outer{
    public void test(){
        //局部内部类
        class Inner{
            public void show(){
                System.out.println("Inner...show");
            }
        }
        
        //局部内部类只能在方法中创建对象，并使用
        Inner in = new Inner();
        in.show();
    }
}
```



### 1.4 匿名内部类

**1.4.1 认识匿名内部类，基本使用**

各位同学，接下来学习一种再实际开发中用得最多的一种内部类，叫匿名内部类。相比于前面几种内部类，匿名内部类就比较重要的。

我们还是先认识一下什么是匿名内部类？

匿名内部类是一种特殊的局部内部类；所谓匿名，指的是程序员不需要为这个类声明名字。

下面就是匿名内部类的格式：

```java
new 父类/接口(参数值){
    @Override
    重写父类/接口的方法;
}
```

匿名内部类本质上是一个没有名字的子类对象、或者接口的实现类对象。

比如，先定义一个Animal抽象类，里面定义一个cry()方法，表示所有的动物有叫的行为，但是因为动物还不具体，cry()这个行为并不能具体化，所以写成抽象方法。

```java
public abstract class Animal{
    public abstract void cry();
}
```

接下来，我想要在不定义子类的情况下创建Animal的子类对象，就可以使用匿名内部类

```java
public class Test{
    public static void main(String[] args){
        //这里后面new 的部分，其实就是一个Animal的子类对象
        //这里隐含的有多态的特性： Animal a = Animal子类对象;
        Animal a = new Animal(){
            @Override
            public void cry(){
                System.out.println("猫喵喵喵的叫~~~");
            }
        }
        a.eat(); //直线上面重写的cry()方法
    }
}
```

需要注意的是，匿名内部类在编写代码时没有名字，编译后系统会为自动为匿名内部类生产字节码，字节码的名称会以`外部类$1.class`的方法命名

![1665658585267](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131624739.png)

**匿名内部类的作用：简化了创建子类对象、实现类对象的书写格式。**



**1.4.2 匿名内部类的应用场景**

学习完匿名内部类的基本使用之后，我们再来看一下匿名内部类在实际中的应用场景。其实一般我们会主动的使用匿名内部类。

**只有在调用方法时，当方法的形参是一个接口或者抽象类，为了简化代码书写，而直接传递匿名内部类对象给方法。**这样就可以少写一个类。比如，看下面代码

```java
public interface Swimming{
    public void swim();
}
```

```java
public class Test{
    public static void main(String[] args){
        Swimming s1 = new Swimming(){
            public void swim(){
                System.out.println("狗刨飞快");
            }
        };
        go(s1);
        
        Swimming s1 = new Swimming(){
            public void swim(){
                System.out.println("猴子游泳也还行");
            }
        };
        go(s1);
    }
    //形参是Swimming接口，实参可以接收任意Swimming接口的实现类对象
    public static void go(Swimming s){
        System.out.println("开始~~~~~~~~");
        s.swim();
        System.out.println("结束~~~~~~~~");
    }
}
```

## 二、枚举

### 2.1 认识枚举

**2.1.1 认识枚举、枚举的原理**

同学们，接下来我们学习一个新的知识点，枚举。枚举是我们以后在项目开发中偶尔会用到的知识。话不多说，我们还是先来认识一下枚举。

枚举是一种特殊的类，它的格式是：

```java
public enum 枚举类名{
    枚举项1,枚举项2,枚举项3;
}
```

其实枚举项就表示枚举类的对象，只是这些对象在定义枚举类时就预先写好了，以后就只能用这几个固定的对象。

我们用代码演示一下，定义一个枚举类A，在枚举类中定义三个枚举项X, Y, Z

```java
public enum A{
    X,Y,Z;
}
```

想要获取枚举类中的枚举项，只需要用类名调用就可以了

```java
public class Test{
    public static void main(String[] args){
        //获取枚举A类的，枚举项
        A a1 = A.X;
        A a2 = A.Y;
        A a3 = A.Z;
    }
}
```

刚才说，枚举项实际上是枚举类的对象，这一点其实可以通过反编译的形式来验证（需要用到反编译的命令，这里不能直接将字节码拖进idea反编译）

![1665669996020](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131625231.png)

我们会看到，枚举类A是用class定义的，说明枚举确实是一个类，而且X，Y，Z都是A类的对象；而且每一个枚举项都是被`public static final `修饰，所以被可以类名调用，而且不能更改。

**2.1.2 枚举深入**

既然枚举是一个类的话，我们能不能在枚举类中定义构造器、成员变量、成员方法呢？答案是可以的。来看一下代码吧

```java
public enum A{
    //定义枚举项
    X,Y,Z("张三"); //枚举项后面加括号，就是在执行枚举类的带参数构造方法。
    
    //定义空构造器
    public A(){
        
    }
    
    //成员变量
    private String name;
    //定义带参数构造器
    public A(String name){
        this.name=name;
    }
    
    //成员方法
    public String getName(){
        return name;
    }
    ...
}
```

虽然枚举类中可以像类一样，写一些类的其他成员，但是一般不会这么写，如果你真要这么干的话，到不如直接写普通类来的直接。



### 2.2 枚举的应用场景

刚才我们认识了一下什么是枚举，接下来我们看一下枚举在实际中的运用，枚举的应用场景是这样的：**枚举一般表示一组信息，然后作为参数进行传输。**

我们来看一个案例。比如我们现在有这么一个应用，用户进入应用时，需要让用户选择是女生、还是男生，然后系统会根据用户选择的是男生，还是女生推荐不同的信息给用户观看。

![1665670887179](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131625388.png)

这里我们就可以先定义一个枚举类，用来表示男生、或者女生

```java
public class Constant{
    BOY,GRIL
}
```

再定义一个测试类，完成用户进入系统后的选择

```java
public class Test{
    public static void main(String[] args){
        //调用方法，传递男生
        provideInfo(Constant.BOY);
    }
    
    public static void provideInfo(Constant c){
        switch(c){
            case BOY:
                System.out.println("展示一些信息给男生看");
                break;
            case GRIL:
                System.out.println("展示一些信息给女生看");
                break;
        }
    }
}
```

最终再总结一下枚举的应用场景：**枚举一般表示几个固定的值，然后作为参数进行传输**。



## 三、泛型

### 3.1 认识泛型

所谓泛型指的是，在定义类、接口、方法时，同时声明了一个或者多个类型变量（如：<E>），称为泛型类、泛型接口、泛型方法、它们统称为泛型。

比如我们前面学过的ArrayList类就是一个泛型类，我们可以打开API文档看一下ArrayList类的声明。

![1665671616852](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131625493.png)

ArrayList集合的设计者在定义ArrayList集合时，就已经明确ArrayList集合时给别人装数据用的，但是别人用ArrayList集合时候，装什么类型的数据他不知道，所以就用一个`<E>`表示元素的数据类型。

当别人使用ArrayList集合创建对象时，`new ArrayList<String> `就表示元素为String类型，`new ArrayList<Integer>`表示元素为Integer类型。

![1665671987771](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131625046.png)

我们总结一下泛型的作用、本质：

- **泛型的好处：在编译阶段可以避免出现一些非法的数据。**

- **泛型的本质：把具体的数据类型传递给类型变量。**



### 3.2 自定义泛型类

接下来我们学习一下自定义泛型类，但是有一些话需要给大家提前交代一下：泛型类，在实际工作中一般都是源代码中写好，我们直接用的，就是ArrayList<E>这样的，自己定义泛型类是非常少的。

自定义泛型类的格式如下

```java
//这里的<T,W>其实指的就是类型变量，可以是一个，也可以是多个。
public class 类名<T,W>{
    
}
```

接下来，我们自己定义一个MyArrayList<E>泛型类，模拟一下自定义泛型类的使用。注意这里重点仅仅只是模拟泛型类的使用，所以方法中的一些逻辑是次要的，也不会写得太严谨。

```java
//定义一个泛型类，用来表示一个容器
//容器中存储的数据，它的类型用<E>先代替用着，等调用者来确认<E>的具体类型。
public class MyArrayList<E>{
    private Object[] array = new Object[10];
    //定一个索引，方便对数组进行操作
    private int index;
    
    //添加元素
    public void add(E e){
        array[index]=e;
        index++;
    }
    
    //获取元素
    public E get(int index){
        return (E)array[index];
    }
}
```

接下来，我们写一个测试类，来测试自定义的泛型类MyArrayList是否能够正常使用

```java
public class Test{
    public static void main(String[] args){
        //1.确定MyArrayList集合中，元素类型为String类型
        MyArrayList<String> list = new MyArrayList<>();
        //此时添加元素时，只能添加String类型
        list.add("张三");
        list.add("李四");
        
         //2.确定MyArrayList集合中，元素类型为Integer类型
        MyArrayList<Integer> list1 = new MyArrayList<>();
        //此时添加元素时，只能添加String类型
        list.add(100);
        list.add(200);
        
    }
}
```

关于自定义泛型类，你们把这个案例理解，对于初学者来说，就已经非常好了。



### 3.3 自定义泛型接口

在上一节中，我们已经学习了自定义泛型类，接下来我们学习一下泛型接口。泛型接口其实指的是在接口中把不确定的数据类型用`<类型变量>`表示。定义格式如下：

```java
//这里的类型变量，一般是一个字母，比如<E>
public interface 接口名<类型变量>{
    
}
```

比如，我们现在要做一个系统要处理学生和老师的数据，需要提供2个功能，保存对象数据、根据名称查询数据，要求：这两个功能处理的数据既能是老师对象，也能是学生对象。

首先我们得有一个学生类和老师类

```java
public class Teacher{

}
```

```java
public class Student{
    
}
```

我们定义一个`Data<T>`泛型接口，T表示接口中要处理数据的类型。

```java
public interface Data<T>{
    public void add(T t);
    
    public ArrayList<T> getByName(String name);
}
```

接下来，我们写一个处理Teacher对象的接口实现类

```java
//此时确定Data<E>中的E为Teacher类型，
//接口中add和getByName方法上的T也都会变成Teacher类型
public class TeacherData implements Data<Teacher>{
   	public void add(Teacher t){
        
    }
    
    public ArrayList<Teacher> getByName(String name){
        
    }
}
```

接下来，我们写一个处理Student对象的接口实现类

```java
//此时确定Data<E>中的E为Student类型，
//接口中add和getByName方法上的T也都会变成Student类型
public class StudentData implements Data<Student>{
   	public void add(Student t){
        
    }
    
    public ArrayList<Student> getByName(String name){
        
    }
}
```

再啰嗦几句，在实际工作中，一般也都是框架底层源代码把泛型接口写好，我们实现泛型接口就可以了。



### 3.4 泛型方法

同学们，接下来我们学习一下泛型方法。下面就是泛型方法的格式

```java
public <泛型变量,泛型变量> 返回值类型 方法名(形参列表){
    
}
```

下图中在返回值类型和修饰符之间有<T>定义的才是泛型方法。

![1665750638693](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131625951.png)

接下我们看一个泛型方法的案例

```java
public class Test{
    public static void main(String[] args){
        //调用test方法，传递字符串数据，那么test方法的泛型就是String类型
        String rs = test("test");
    
        //调用test方法，传递Dog对象，那么test方法的泛型就是Dog类型
    	Dog d = test(new Dog()); 
    }
    
    //这是一个泛型方法<T>表示一个不确定的数据类型，由调用者确定
    public static <T> test(T t){
        return t;
    }
}
```

### 3.5 泛型限定

接着，我们来学习一个泛型的特殊用法，叫做泛型限定。泛型限定的意思是对泛型的数据类型进行范围的限制。有如下的三种格式

- <?> 表示任意类型
- <? extends 数据类型> 表示指定类型或者指定类型的子类
- <? super 数据类型> 表示指定类型或者指定类型的父类

下面我们演示一下，假设有Car作为父类，BENZ，BWM两个类作为Car的子类，代码如下

```java
class Car{}
class BENZ extends Car{}
class BWN extends Car{}

public class Test{
    public static void main(String[] args){
        //1.集合中的元素不管是什么类型，test1方法都能接收
        ArrayList<BWM> list1 = new ArrayList<>();
        ArrayList<Benz> list2 = new ArrayList<>();
        ArrayList<String> list3 = new ArrayList<>();
        test1(list1);
        test1(list2);
        test1(list3);
        
        //2.集合中的元素只能是Car或者Car的子类类型，才能被test2方法接收
        ArrayList<Car> list4 = new ArrayList<>();
        ArrayList<BWM> list5 = new ArrayList<>();
        test2(list4);
        test2(list5);
        
        //2.集合中的元素只能是Car或者Car的父类类型，才能被test3方法接收
        ArrayList<Car> list6 = new ArrayList<>();
        ArrayList<Object> list7 = new ArrayList<>();
        test3(list6);
        test3(list7);
    }
    
    public static void test1(ArrayList<?> list){
        
    }
    
    public static void test2(ArrayList<? extends Car> list){
        
    }
    
   	public static void test3(ArrayList<? super Car> list){
        
    }
}
```

### 3.6 泛型擦除

最后，关于泛型还有一个特点需要给同学们介绍一下，就是泛型擦除。什么意思呢？**也就是说泛型只能编译阶段有效，一旦编译成字节码，字节码中是不包含泛型的**。而且泛型只支持引用数据类型，不支持基本数据类型。

把下面的代码的字节码进行反编译

![1665752105271](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131625310.png)

下面是反编译之后的代码，我们发现ArrayList后面没有泛型

![1665752037764](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131625681.png)



## 四、常用API

各位同学，恭喜大家，到目前位置我们关于面向对象的语法知识就全部学习完了。接下来我们就可以拿着这些语法知识，去学习一个一个的API方法，掌握的API方法越多，那么Java的编程能力就越强。

 API（Application Programming interface）意思是应用程序编程接口，说人话就是Java帮我们写好的一些程序，如：类、方法等，我们直接拿过来用就可以解决一些问题。

![1665752705389](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131626714.png)

我们要学习那些API呢？把下面一种图中的所有类的常用方法学会了，那我们JavaSE进阶的课程就算你全学会了。

![1665752813753](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131626773.png)

很多初学者给我反应的问题是，这些API一听就会，但是就是记住不！送同学们一句话**，**

**“千里之行始于足下，多记、多查、多些代码、孰能生巧！”**

![1665752883617](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131626250.png)

### 4.1 Object类

各位小伙伴，我们要学习的第一个API就是Object类。Object类是Java中所有类的祖宗类，因此，Java中所有类的对象都可以直接使用Object类中提供的一些方法。

按照下图的提示，可以搜索到你想要找的类

![1665753230409](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131626940.png)

我们找到Object类的下面两个方法

![1665753282718](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131626605.png)

**4.1.1 toString()方法**

我们先来学习toString()方法。

```java
public String toString()
    调用toString()方法可以返回对象的字符串表示形式。
    默认的格式是：“包名.类名@哈希值16进制”
```

假设有一个学生类如下

```java
public class Student{
    private String name;
    private int age;
    
    public Student(String name, int age){
        this.name=name;
        this.age=age;
    }
}
```

再定义一个测试类

```java
public class Test{
    public static void main(String[] args){
        Student s1 = new Student("赵敏",23);
        System.out.println(s1.toString()); 
    }
}
```

打印结果如下

![1665753662732](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131626361.png)

如果，在Student类重写toString()方法，那么我们可以返回对象的属性值，代码如下

```java
public class Student{
    private String name;
    private int age;
    
    public Student(String name, int age){
        this.name=name;
        this.age=age;
    }
    
    @Override
    public String toString(){
        return "Student{name=‘"+name+"’, age="+age+"}";
    }
}
```

运行测试类，结果如下

![1665754067446](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131626487.png)



**4.1.2 equals(Object o)方法**

接下来，我们学习一下Object类的equals方法

```java
public boolean equals(Object o)
    判断此对象与参数对象是否"相等"
```

我们写一个测试类，测试一下

```java
public class Test{
	public static void main(String[] args){
        Student s1 = new Student("赵薇",23);
        Student s2 = new Student("赵薇",23);
        
        //equals本身也是比较对象的地址，和"=="没有区别
        System.out.println(s1.equals(s2)); //false
         //"=="比较对象的地址
        System.out.println(s1==s2); //false
    }
}
```

但是如果我们在Student类中，把equals方法重写了，就按照对象的属性值进行比较

```java
public class Student{
    private String name;
    private int age;
    
    public Student(String name, int age){
        this.name=name;
        this.age=age;
    }
    
    @Override
    public String toString(){
        return "Student{name=‘"+name+"’, age="+age+"}";
    }
    
    //重写equals方法，按照对象的属性值进行比较
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Student student = (Student) o;

        if (age != student.age) return false;
        return name != null ? name.equals(student.name) : student.name == null;
    }
}
```

再运行测试类，效果如下

![1665754859931](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131626459.png)

总结一下Object的toString方法和equals方法

```java
public String toString()
   	返回对象的字符串表示形式。默认的格式是：“包名.类名@哈希值16进制”
   	【子类重写后，返回对象的属性值】
   	
public boolean equals(Object o)
    判断此对象与参数对象是否"相等"。默认比较对象的地址值，和"=="没有区别
    【子类重写后，比较对象的属性值】
```



**4.1.3 clone() 方法**

接下来，我们学习Object类的clone()方法，克隆。意思就是某一个对象调用这个方法，这个方法会复制一个一模一样的新对象，并返回。

```java
public Object clone()
    克隆当前对象，返回一个新对象
```

想要调用clone()方法，必须让被克隆的类实现Cloneable接口。如我们准备克隆User类的对象，代码如下

```java
public class User implements Cloneable{
    private String id; //编号
    private String username; //用户名
    private String password; //密码
    private double[] scores; //分数

    public User() {
    }

    public User(String id, String username, String password, double[] scores) {
        this.id = id;
        this.username = username;
        this.password = password;
        this.scores = scores;
    }

    //...get和set...方法自己加上

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

接着，我们写一个测试类，克隆User类的对象。并观察打印的结果

```java
public class Test {
    public static void main(String[] args) throws CloneNotSupportedException {
        User u1 = new User(1,"zhangsan","wo666",new double[]{99.0,99.5});
		//调用方法克隆得到一个新对象
        User u2 = (User) u1.clone();
        System.out.println(u2.getId());
        System.out.println(u2.getUsername());
        System.out.println(u2.getPassword());
        System.out.println(u2.getScores()); 
    }
}
```

我们发现，克隆得到的对象u2它的属性值和原来u1对象的属性值是一样的。

![1665757008178](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131627635.png)

上面演示的克隆方式，是一种浅克隆的方法，浅克隆的意思：**拷贝出来的对象封装的数据与原对象封装的数据一模一样（引用类型拷贝的是地址值）**。如下图所示

![1665757187877](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131627849.png)

还有一种拷贝方式，称之为深拷贝，拷贝原理如下图所示

![1665757265609](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131627296.png)

下面演示一下深拷贝User对象

```java
public class User implements Cloneable{
    private String id; //编号
    private String username; //用户名
    private String password; //密码
    private double[] scores; //分数

    public User() {
    }

    public User(String id, String username, String password, double[] scores) {
        this.id = id;
        this.username = username;
        this.password = password;
        this.scores = scores;
    }

    //...get和set...方法自己加上

	@Override
    protected Object clone() throws CloneNotSupportedException {
        //先克隆得到一个新对象
        User u = (User) super.clone();
        //再将新对象中的引用类型数据，再次克隆
        u.scores = u.scores.clone();
        return u;
    }
}
```

![1665757536274](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131627947.png)



### 4.2 Objects类

Objects是一个工具类，提供了一些方法可以对任意对象进行操作。主要方法如下

![1665760840329](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131627309.png)

下面写代码演示一下这几个方法

```java
public class Test{
    public static void main(String[] args){
        String s1 = null;
        String s2 = "itheima";
        
        //这里会出现NullPointerException异常，调用者不能为null
        System.out.println(s1.equals(s2));
        //此时不会有NullPointerException异常，底层会自动先判断空
        System.out.println(Objects.equals(s1,s2));
        
        //判断对象是否为null，等价于==
        System.out.println(Objects.isNull(s1)); //true
        System.out.println(s1==null); //true
        
        //判断对象是否不为null，等价于!=
        System.out.println(Objects.nonNull(s2)); //true
        System.out.println(s2!=null); //true
    }
}
```



### 4.3 基本类型包装类

同学们，接下来我们学习一下包装类。为什么要学习包装类呢？因为在Java中有一句很经典的话，万物皆对象。Java中的8种基本数据类型还不是对象，所以要把它们变成对象，变成对象之后，可以提供一些方法对数据进行操作。

Java中8种基本数据类型都用一个包装类与之对一个，如下图所示

<img src="https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202410131627964.png" alt="1665758797003" style="zoom:50%;" />

我们学习包装类，主要学习两点：

- 1. 创建包装类的对象方式、自动装箱和拆箱的特性；
- 2. 利用包装类提供的方法对字符串和基本类型数据进行相互转换



**4.2.1 创建包装类对象**

我们先来学习，创建包装类对象的方法，以及包装类的一个特性叫自动装箱和自动拆箱。我们以Integer为例，其他的可以自己学，都是类似的。

```java
//1.创建Integer对象，封装基本类型数据10
Integer a = new Integer(10);

//2.使用Integer类的静态方法valueOf(数据)
Integer b = Integer.valueOf(10);

//3.还有一种自动装箱的写法（意思就是自动将基本类型转换为引用类型）
Integer c = 10;

//4.有装箱肯定还有拆箱（意思就是自动将引用类型转换为基本类型）
int d = c;

//5.装箱和拆箱在使用集合时就有体现
ArrayList<Integer> list = new ArrayList<>();
//添加的元素是基本类型，实际上会自动装箱为Integer类型
list.add(100);
//获取元素时，会将Integer类型自动拆箱为int类型
int e = list.get(0);
```



**4.2.2 包装类数据类型转换**

在开发中，经常使用包装类对字符串和基本类型数据进行相互转换。

- 把字符串转换为数值型数据：包装类.parseXxx(字符串)

```java
public static int parseInt(String s)
    把字符串转换为基本数据类型
```

- 将数值型数据转换为字符串：包装类.valueOf(数据);

```java
public static String valueOf(int a)
    把基本类型数据转换为
```

- 写一个测试类演示一下

```java
//1.字符串转换为数值型数据
String ageStr = "29";
int age1 = Integer.parseInt(ageStr);

String scoreStr = 3.14;
double score = Double.prarseDouble(scoreStr);

//2.整数转换为字符串，以下几种方式都可以（挑中你喜欢的记一下）
Integer a = 23;
String s1 = Integer.toString(a);
String s2 = a.toString();
String s3 = a+"";
String s4 = String.valueOf(a);
```

