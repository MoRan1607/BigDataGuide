# Python环境搭建与输入输出

## 一、Python概述

### 1.1 计算机资源

在开发领域，计算机资源可以分为两部分：软件资源 + 硬件资源

==软件资源：看得见，摸不着==

==硬件资源：看得见，摸得着==

硬件资源（CPU、内存、硬盘、风扇、电源、键盘、鼠标...）

软件资源（Office办公软件、网易云音乐、各种各样的计算机游戏）



思考：我们发现，软硬件之间其实是可以交互的，这是什么原理呢？

答：使用操作系统，==操作系统==是计算机软硬件之间的桥梁

### 1.2 操作系统分类

在日常的应用中，操作系统大概可以分为三大类：

① Windows操作系统 

② MacOS操作系统 

③ Linux操作系统（服务器端使用量最大的操作系统）

### 1.3 为什么要学习Python

**① 技术趋势**

Python自带明星属性，热度稳居编程语言界前三

![image-20210306090039676](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112221752.png)

**② 简单易学**

开发代码少，精确表达需求逻辑；==33个关键字，7种基本数据类型==；语法规则简单，接近自然语。

![image-20210306090337310](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112221457.png)

**③ 应用广泛**

Python语言涉及IT行业70%以上的技术领域

![image-20210306090727147](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112221777.png)

### 1.4 Python语言的缺点

① Python其运行速度相对于C/C++/Java要略慢一些

② Python由于语言的特性，无法对代码进行加密

③ Python的版本之间，兼容性不太理想（Python2和Python3）

### 1.5 Python语言介绍

Python是一种==跨平台==的计算机程序设计语⾔。 是一个高层次的结合了==解释性、编译性、互动性和面向对象==的脚本语⾔。最初被设计用于编写自动化脚本Shell（适用于Linux操作系统），随着版本的不断更新和语言新功能的添加，逐渐被用于独立的、大型项目的开发。

其实目前很多知名的机器学习、⼈⼯智能以及深度学习框架也都是基于Python语⾔进⾏开发的：

Google开源机器学习框架：TensorFlow

开源社区主推学习框架：Scikit-learn

百度开源深度学习框架：Paddle

### 1.6 Python2.x和Python3.x版本的区别

在目前的Python领域，其主要应用版本有两个：Python2和Python3

![image-20210306091509401](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112221123.png)

主要区别可以理解为：==输入、输出以及编码格式的不同==

**Python2.x与Python3.x版本对比**

- Python2.x

- Python3.x
  - Python3.6、==Python3.7==、Python3.8、Python3.9...

在生产环境中，我们⼀般不会选择最新版本的Python，因为可能会存在未知Bug，所以⼀般强烈建议大家在选择软件版本时，向前推1 ~ 2个版本。所以咱们课程主要讲解Python3.7版本。

## 二、Python解析器（☆）

### 2.1 Python解析器的作用

```python
print('Hello World')
```

由于Python属于高级语言，其并不能直接在计算机中运行，因为缺少Python语言的运行环境：Python解析器

![image-20210306092814499](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112221013.png)

Python解析器的作用：==就是把Python代码转换为计算机底层可以识别的机器语言==，如0101...

### 2.2 Python解析器的种类

① CPython，C语言开发的解释器[官方]，应⽤广泛的解释器。

② IPython，基于CPython的一种交互式解释器。

③ 其他解释器

PyPy，基于Python语言开发的解释器。

JPython，运⾏在Java平台的解释器，直接把Python代码编译成Java字节码执⾏。

IronPython，运⾏在微软.Net平台上的Python解释器，可直接把Python代码编译成.Net的字节码。

### 2.3 下载Python解析器

下载地址：https://www.python.org/downloads/release/python-379/

[单击上述链接] -- 查找目标文件：Windows x86-64 executable installer -- 单击即可下载。

![image-20210306093337458](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112222448.png)

### 2.4 Python解析器的安装

第一步：双击运行Python的解析器，选择==自定义安装==以及==添加Python到环境变量==

![image-20210306095227329](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112222304.png)

第二步：选择所有要安装的功能菜单，默认全部勾选

![image-20210306095439595](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112222433.png)

> pip：Python的包管理工具，可以用来安装未来我们项目中需要使用的各种模块

第三步：设置Python解析器的安装路径，强烈建议安装在除C盘以外的盘符

![image-20210306095909408](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112222437.png)

第四步：测试Python解析器是否可以使用

按Windows + R，输入cmd字符，打开Windows的DOS窗口，输入python（全部小写），如下图所示：

![image-20210306100236471](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112222586.png)

出现了以上界面，就代表Python3.7的解析器已经安装成功了。如何从这个窗口中退出到DOS模式呢？

答：使用exit()方法

```python
>>> exit() 回车
```

## 三、Python开发工具PyCharm（☆）

### 3.1 为什么要安装PyCharm

工欲善其事必先利其器

在Python的开发领域，其开发工具非常非常多，EditPlus、Notepad++、Sublime Text3、Visual Studio Code、PyCharm（目前功能最强大的IDE）

![image-20210306102520443](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112222993.png)

### 3.2 PyCharm的主要作用

PyCharm是⼀种Python IDE （集成开发环境），带有一整套可以帮助用户在使用Python语言开发时提高其效率的⼯具，内部集成的功能如下：

Project管理

智能提示

语法高亮

代码跳转

调试代码

解释代码(解释器)

框架和库

......

### 3.3 PyCharm的分类

PyCharm一共有两个版本：专业版（收费） 与 社区版（免费、开源）

![image-20210306102803654](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112224280.png)

在基础班，PyCharm社区版足够我们使用，绰绰有余。

### 3.4 下载PyCharm

下载地址：**https://www.jetbrains.com/pycharm/download/#section=windows**

![image-20210306103210207](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112224740.png)

### 3.5 PyCharm安装

第一步：双击PyCharm软件安装包，进行软件安装

![image-20210306104505660](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112224523.png)

第二步：设置软件的安装路径，理论上没有任何要求，但是建议放在除C盘以外的盘符

![image-20210306105046370](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112224638.png)

第三步：PyCharm基本设置，创建桌面图标与.py文件关联

![image-20210306105223088](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112225027.png)

### 3.6 PyCharm软件的使用

**PyCharm 2017.3版本下载链接：**

链接：https://pan.baidu.com/s/1D2AN9pdbyfpFAv-T_RW9QA?pwd=mr66 

提取码：mr66

#### 3.6.1 创建Python项目

什么是项目？其实我们在实际开发中，每次参与一个工作的开发都是一个项目的开发过程。所以使用PyCharm的第一件事就是学习Python项目的创建过程。

第一步：创建项目

![image-20210306110324245](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112225817.png)

第二步：设置项目路径，必须放在C盘以外的盘符（非常重要！！！）

![image-20210306110916916](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112225813.png)

配置完成后，单机Create创建Python项目。

#### 3.6.2 新建文件与代码书写

![image-20210306111656942](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112225670.png)

> 如果将来要上传到服务器的文件，那么文件名切记不能使用中文。

 编写Hello World

```python
print('Hello World')
```

#### 3.6.3 运行代码

![image-20210306112159796](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112225870.png)

运行结果：

![image-20210306112245497](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112225988.png)

#### 3.6.4 设置或更换Python解析器

打开File文件，找到Settings设置，如下图所示：更换Python解析器

![image-20210306113159846](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112233155.png)

#### 3.6.5 PyCharm软件本身设置

① 软件主题（软件未来的样式）

② 编码字体的设置

③ 代码字号的设置（文字大小）

打开File文件 => Settings设置，找到界面设置：

![image-20210306115108007](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112233525.png)

主题设置：

![image-20210306115322452](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112233231.png)

字体与字号设置：

![image-20210306115516870](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112233123.png)

字体设置：

![image-20210306115611339](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112233645.png)

字号设置：

![image-20210306115735435](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112233657.png)

#### 3.6.6 打开项目与关闭项目

打开项目：本身项目已经存在了，我们直接打开。

![image-20210306120615122](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112233782.png)

选择项目目录（文件夹）即可，如下图所示：

![image-20210306120712306](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112233862.png)

① This Window => 覆盖当前项⽬，从⽽打开目标项目

② New Window => 在新窗⼝打开，则打开两次PyCharm，每个PyCharm负责一个项⽬

③ Attach => 把两个项目合并在一起，放在同一个窗口中

关闭项目：对已经运行项目进行关闭操作。

![image-20210306120425927](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112234386.png)

### 3.7 PyCharm 2017.3永久激活

**1、下载jar包**

此jar包的目的就是让截获截止时间并骗过PyCharm

下载链接：

链接：https://pan.baidu.com/s/1lj5Ce0JrTql0TQQeq497TA?pwd=mr66 

提取码：mr66

**2、修改相关配置文件**

下载完毕后， 将其放入PyCharm在你本地的安装目录bin下。

并且修改两个以vmoptions为结尾的启动文件如图所示：

并且在**两个文件**后追加：红色部分为自己的安装目录

-javaagent:<font color=red>D:\CodingSoft\PyCharm2017.3.2\bin</font>\JetbrainsCrack-2.6.10-release-enc.jar

**3、重启PyCharm**

注意：如果之前已经存在注册码，可以直接跳到第4步，如果没有注册码，则填写下面的注册码！

```
BIG3CLIK6F-eyJsaWNlbnNlSWQiOiJCSUczQ0xJSzZGIiwibGljZW5zZWVOYW1lIjoibGFuIHl1IiwiYXNzaWduZWVOYW1lIjoiIiwiYXNzaWduZWVFbWFpbCI6IiIsImxpY2Vuc2VSZXN0cmljdGlvbiI6IkZvciBlZHVjYXRpb25hbCB1c2Ugb25seSIsImNoZWNrQ29uY3VycmVudFVzZSI6ZmFsc2UsInByb2R1Y3RzIjpbeyJjb2RlIjoiQUMiLCJwYWlkVXBUbyI6IjIwMTctMTEtMjMifSx7ImNvZGUiOiJETSIsInBhaWRVcFRvIjoiMjAxNy0xMS0yMyJ9LHsiY29kZSI6IklJIiwicGFpZFVwVG8iOiIyMDE3LTExLTIzIn0seyJjb2RlIjoiUlMwIiwicGFpZFVwVG8iOiIyMDE3LTExLTIzIn0seyJjb2RlIjoiV1MiLCJwYWlkVXBUbyI6IjIwMTctMTEtMjMifSx7ImNvZGUiOiJEUE4iLCJwYWlkVXBUbyI6IjIwMTctMTEtMjMifSx7ImNvZGUiOiJSQyIsInBhaWRVcFRvIjoiMjAxNy0xMS0yMyJ9LHsiY29kZSI6IlBTIiwicGFpZFVwVG8iOiIyMDE3LTExLTIzIn0seyJjb2RlIjoiREMiLCJwYWlkVXBUbyI6IjIwMTctMTEtMjMifSx7ImNvZGUiOiJEQiIsInBhaWRVcFRvIjoiMjAxNy0xMS0yMyJ9LHsiY29kZSI6IlJNIiwicGFpZFVwVG8iOiIyMDE3LTExLTIzIn0seyJjb2RlIjoiUEMiLCJwYWlkVXBUbyI6IjIwMTctMTEtMjMifSx7ImNvZGUiOiJDTCIsInBhaWRVcFRvIjoiMjAxNy0xMS0yMyJ9XSwiaGFzaCI6IjQ3NzU1MTcvMCIsImdyYWNlUGVyaW9kRGF5cyI6MCwiYXV0b1Byb2xvbmdhdGVkIjpmYWxzZSwiaXNBdXRvUHJvbG9uZ2F0ZWQiOmZhbHNlfQ==-iygsIMXTVeSyYkUxAqpHmymrgwN5InkOfeRhhPIPa88FO9FRuZosIBTY18tflChACznk3qferT7iMGKm7pumDTR4FbVVlK/3n1ER0eMKu2NcaXb7m10xT6kLW1Xb3LtuZEnuis5pYuEwT1zR7GskeNWdYZ0dAJpNDLFrqPyAPo5s1KLDHKpw+VfVd4uf7RMjOIzuJhAAYAG+amyivQt61I9aYiwpHQvUphvTwi0X0qL/oDJHAQbIv4Qwscyo4aYZJBKutYioZH9rgOP6Yw/sCltpoPWlJtDOcw/iEWYiCVG1pH9AWjCYXZ9AbbEBOWV71IQr5VWrsqFZ7cg7hLEJ3A==-MIIEPjCCAiagAwIBAgIBBTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTE1MTEwMjA4MjE0OFoXDTE4MTEwMTA4MjE0OFowETEPMA0GA1UEAwwGcHJvZDN5MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxcQkq+zdxlR2mmRYBPzGbUNdMN6OaXiXzxIWtMEkrJMO/5oUfQJbLLuMSMK0QHFmaI37WShyxZcfRCidwXjot4zmNBKnlyHodDij/78TmVqFl8nOeD5+07B8VEaIu7c3E1N+e1doC6wht4I4+IEmtsPAdoaj5WCQVQbrI8KeT8M9VcBIWX7fD0fhexfg3ZRt0xqwMcXGNp3DdJHiO0rCdU+Itv7EmtnSVq9jBG1usMSFvMowR25mju2JcPFp1+I4ZI+FqgR8gyG8oiNDyNEoAbsR3lOpI7grUYSvkB/xVy/VoklPCK2h0f0GJxFjnye8NT1PAywoyl7RmiAVRE/EKwIDAQABo4GZMIGWMAkGA1UdEwQCMAAwHQYDVR0OBBYEFGEpG9oZGcfLMGNBkY7SgHiMGgTcMEgGA1UdIwRBMD+AFKOetkhnQhI2Qb1t4Lm0oFKLl/GzoRykGjAYMRYwFAYDVQQDDA1KZXRQcm9
```

**4、验证**

验证是否是有效期是否截止到2099年  

点击 PyCharm的help->about查看：

![image-20220723084720952](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112234016.png)

## 四、Python注释

### 4.1 注释的作用

首先强调一件事：Python代码 => Python解析器 => 机器语言，但是注释经过了Python的解释器并不会解析与执行。因为其主要就是进行代码的注释。

注释作用：==提高代码的阅读性==

![image-20210306143714495](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112234505.png)

在我们编写Python程序时，为了了提高程序的可读性，强烈建议大家为核心代码添加注释信息。

### 4.2 Python注释的基本语法

#### 4.2.1 单行注释

单行注释，以"#"(Shift + 3)号开头，只能注释一行内容

```python
# 注释内容
```

示例代码：

第一种：代码行的上面

```python
# 输出Hello World字符串
print('Hello World')
```

第二种：放在代码的后面(代码后面保留2个空格)

```python
print('Hello World')  # 输出Hello World字符串
```

#### 4.2.1 多行注释

多行注释：可以同时注释多行代码或程序，常用于代码块的注释

基本语法：

```python
"""
注释内容
第一行
第二行
第三行
"""
```

或

```
'''
注释内容
第一行
第二行
第三行
'''
```

示例代码：

```python
"""
Hi, 大家好
我是黑马程序员的小伙伴
从今天开始，我们将一起学习Python这门语言
"""

'''
Hi, 大家好
我是黑马程序员的小伙伴
从今天开始，我们将一起学习Python这门语言
'''
print('Hi, 大家好')
print('我是黑马程序员的小伙伴')
print('从今天开始，我们将一起学习Python这门语言')
```

#### 4.2.3 PyCharm注释小技巧（快捷键）

在PyCharm中，我们可以使用`Ctrl + /斜杠`来对代码或程序进行快速注释。

## 五、PyCharm常用快捷键

### 5.1 代码提示

在PyCharm中，当我们输入Python关键字中的前2~3个字符，其会自动进行代码提示。这个时候，我们只需要按回车即可以快速的输入某个内容。

![image-20210306150352389](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112238916.png)

### 5.2 代码保存

编写代码时，一定要养成一个好的习惯，使用`Ctrl + S`快速对代码进行保存操作。

个人建议，当写完一行代码时，就按一次。

### 5.3 撤销与恢复

如果不小心删除了某行代码，这个时候我们可以快速按`Ctrl + Z`就可以快速进行恢复。每按一次就撤销一次，如果撤销多了，怎么办？

答：还可以通过`Ctrl + Y`进行恢复操作

## 六、Python中的变量（重点）（☆☆☆☆☆）

### 6.1 变量的学习目标（案例）

案例：实现两个变量的交换

![image-20210306152222362](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112238349.png)

1号杯：可乐

2号杯：牛奶

经过一系列Python操作以后

1号杯：牛奶

2号杯：可乐

### 6.2 引入变量的概念

什么是量：量是程序中的最小单元。

那什么是变量呢？

==① 变量是存储数据的容器==

==② 变量在程序运行过程中是可以发生改变的量== 

==③ 变量存储的数据是临时的==

### 6.3 变量的作用（举个栗子）

淘宝注册案例：

① 写入用户名、密码

==② Python程序要接收用户名和密码（临时存储）==

③ 把刚才接收的用户名和密码永久的存储起来（数据库）

![image-20210306153106909](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112238793.png)

为了解决以上问题，Python开发了变量这样一个概念，可以把用户输入的一些信息，临时的保存起来，保存的这个容器就是Python变量。

### 6.4 变量的定义

基本语法：

```python
变量名称 = 变量的值
注：等号的两边都要保留一个空格，其实Python中建议符号的两边尽量都要保留一个空格
```

> 说明：在Python程序中，这个等号和日常生活中的等号不太一样，其有一个专业名词：赋值运算符，其读法：要从右向左读，把变量的值通过 = 赋值给左边的变量。

### 6.5 变量的命令规则

标识符命名规则是Python中定义变量名称时一种命名规范，具体如下：

==① 由数字、字母、下划线(_)组成==

==② 不能数字开头==

==③ 严格区分⼤小写==

==④ 不能使⽤内置关键字作为变量名称==

![image-20210306155908564](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112238171.png)

> 下划线 => Shift + -减号

举个栗子：

① abc、abc123、_abc、hello（合理）

② 123abc、@abc、abc-123（不合理）

③ _（下划线） => 请问这可以是一个变量名称么？

答：可以

```python
for _ in range(10):
    ...
```

④ 变量abc和变量ABC是同一个变量么？

答：不一样，这是两个完全不同的变量

⑤ 记不住33个关键字怎么办？

答：借助于help()方法

```python
>>> help('keywords')
```

### 6.6 推荐变量的命名规则

① 变量命名一定要做到见名知义。

② 大驼峰：即每个单词首字母都大写，例如： MyName 。

③ 小驼峰：第二个（含）以后的单词首字母大写，例例如： myName 。

④ 下划线：例如： my_name 。

### 6.7 变量的定义与调用

在Python中，记住：变量一定要先定义，后使用，否则会报错。

定义：

```python
name = 'itheima'
address = '北京市顺义区京顺路99号'
```

调用：

```python
print(name)
print(address)
或
print(name, address)
```

### 6.8 变量的定义与使用常见问题

① 变量与字符串如何区别：

==在Python中，如果要赋值的内容添加了单引号或者双引号，其就是Python中的一种数据类型：叫做字符串（日常生活中的文本信息）==

② print打印变量时，喜欢为其添加引号

```python
print(name)  # 输出变量name对应的值
与
print('name')  # 输出'name'这个字符串
```

## 七、Python中变量的数据类型

### 7.1 为什么要学习数据类型

变量的定义非常的简单，但是很多小伙伴可能会想：变量除了存储这种字符类型的数据以外，还能存储其他类型的数据么？其实，在 Python中，我们为了应对不同的业务需求，也会把数据分为不同的类型，如下图所示：

![image-20210306162601034](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112238803.png)

面试题：请手写出Python中的7种数据类型？

答：数值类型、布尔类型、字符串类型、列表类型、元组类型、集合类型、字典类型

今天我们只需要了解前3种即可。



问题：如何判断一个变量到底是什么类型？

答：① 使用type(变量名称)方法，返回变量的数据类型 ② isinstance(变量名称,数据类型)，只能返回True或False（真的还是假的）

### 7.2 数值类型

数值类型就是我们日常生活中的数字，数字又分为两种形式：整数 与 小数（带小数点）

整数类型：int类型

小数类型：float类型

案例1：定义一个人的信息，姓名：Tom、年龄18岁

```python
name = 'Tom'
age = 18
print(type(age))
```

案例2：定义一个超市收银系统，写入一个名称：大白菜，价格：3.5

```python
name = '大白菜'
price = 3.5
print(type(price))
```

### 7.3 布尔类型

布尔类型是与逻辑相关一种数据类型，只有两个值：True（真）与False（假）

案例1：手工定义一个flag变量，其值为True

```python
flag = True
print(flag)
print(type(flag))
```

其实在Python中，很多程序的返回结果也可以是True或False，比如isinstance()

```python
num = 10
print(isinstance(num, int))  # True
print(isinstance(num, bool))  # False
```

### 7.4 字符串类型

在Python变量定义中，如果其赋值的内容是通过单引号或双引号引起来的内容就是字符串str类型。

```python
msg = '这家伙很懒，什么都没有留下...'
print(type(msg))
```

### 7.5 其他类型(了解)

```python
# 1、list列表类型
list1 = [10, 20, 30, 40]
print(type(list1))

# 2、tuple元组类型
tuple1 = (10, 20, 30, 40)
print(type(tuple1))

# 3、set集合类型：去重
set1 = {10, 20, 30}
print(type(set1))

# 4、dict字典类型：查询、搜索
dict1 = {'name':'itheima', 'age':18}
print(type(dict1))
```

## 八、了解Python中的Bug

### 8.1 认识一下bug

所谓bug，就是程序中的错误。如果程序有错误，就需要咱们程序员来进行问题排查，及时纠正错误。

![image-20210306171244287](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112239140.png)

### 8.2 解决bug三步走

第一步：查看错误页面

第二步：看错误的行号

第三步：根据具体的错误，具体分析

### 8.3 PyCharm代码调试（重点）

Debug工具是PyCharm IDE中集成的专门用来调试程序的工具，在这里程序员可以查看程序的执行细节和流程，以方便我们快速找出程序的Bug！

Debug工具使⽤二步走：==① 打断点 ② Debug调试==

### 8.4 下断点

断点应该放在哪个位置：答：代码可能出错的代码段的第一行

![image-20210306171719774](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112239523.png)

### 8.5 Debug调试

![image-20210306171833494](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112239962.png)

### 8.6 单步调试

![image-20210306172007078](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112239601.png)

遇到小闪电图标就代表这一行，可能出错了。

![image-20210306172338345](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112239219.png)

## 九、Python中的格式化输出（☆☆☆）

### 9.1 格式化输出

目前为止，我们所有的输出都是直接通过print(变量名称)形式直接打印的。但是实际工作中，我们可能需要对变量的输出进行格式化操作（按照一定格式进行输出）。

### 9.2 百分号格式化输出

基本语法：

```python
...
print(变量名称)
print('字符串%格式' % (变量名称))
print('字符串%格式 %格式 %格式' % (变量名称1, 变量名称2, 变量名称3))
```

%格式常见形式如下：

| **格式符号** | **转换**               |
| ------------ | ---------------------- |
| ==%s==       | 字符串                 |
| ==%d==       | 有符号的十进制整数     |
| ==%f==       | 浮点数                 |
| %c           | 字符                   |
| %u           | 无符号十进制整数       |
| %o           | 八进制整数             |
| %x           | 十六进制整数（小写ox） |
| %X           | 十六进制整数（大写OX） |
| %e           | 科学计数法（小写'e'）  |
| %E           | 科学计数法（大写'E'）  |
| %g           | %f和%e的简写           |
| %G           | %f和%E的简写           |

案例：定义两个变量name='itheima', age=18，按照如下格式进行输出：我的名字是itheima，今年18岁了。

![image-20210306175326815](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112239358.png)

案例：定义两个变量title='大白菜'，price=3.5，按照如下格式进行输出：今天蔬菜特价了，大白菜只要3.5元/斤。

```python
title = '大白菜'
price = 3.5
# 格式化输出“今天蔬菜特价了，大白菜只要3.5元/斤。"
print("今天蔬菜特价了，%s只要%.2f元/斤。" % (title, price))
```

其实除了%f可以设置小数点位数以外，%d也可以填充序号。

案例：定义两个变量id=1，name='itheima'，按照如下格式进行输出：姓名itheima，学号000001

```python
id = 1
name = 'itheima'
print("姓名%s，学号%06d" % (name, id))
```

### 9.3 format方法格式化输出

基本语法：

```python
...
print('字符串{}'.format(变量名称1))
print('{}字符串{}'.format(变量名称1, 变量名称2))
```

案例：定义两个变量，name='孙悟空'，mobile='18878569090'，按照以下格式进行输出"姓名：孙悟空，联系方式：18878569090"

```python
name = '孙悟空'
mobile = '18878569090'
print("姓名：{}，联系方式：{}".format(name, mobile))
```

### 9.4 format方法简写形式格式化输出（推荐）

在Python3.6以后版本，为了简化format输出操作，引入了一个简写形式：

```python
name = '孙悟空'
mobile = '18878569090'
print(f'姓名：{name}，联系方式：{mobile}')
```

### 9.5 格式化输出中的转义符号

在字符串中，如果出现了\t和\n，其代表的含义就是两个转义字符

```python
\t ：制表符，一个tab键（4个空格）的距离
\n ：换行符
```

案例：

```python
print('*\t*\t*')
print('hello\nworld')
```

特别说明：==默认情况下，每个print()方法执行完毕后，都会输出一个\n换行符。如果不想让print()方法换行，可以添加一个end参数==

```python
print('*', end='')
```

## 十、Python中的标准输入（☆☆☆）

### 10.1 为什么需要输入

到目前为止，我们所有的程序都只能把数据输出给用户。但是实际工作中，我们经常输入获取用户的输入信息，如银行系统中的密码输入、淘宝中的用户登录验证。

![image-20210306182224429](https://raw.githubusercontent.com/qiye0716/picture_typora/main/img/202305112240059.png)

### 10.2 input()输入方法

在Python中，如果想让Python程序接受用户的输入信息，可以使用input()方法

基本语法：

```python
input()
```

但是往往只有input()方法，其意义不大，我们还应该使用一个变量来临时接受用户的输入，已方便后期的操作。

```python
变量名称 = input('提示信息：')
```

案例：银行系统中的，输入密码的过程

```python
password = input('请输入您的银行卡密码：')
print(f'您输入的银行卡密码为：{password}')
```

### 10.3 input()方法重要事项

记住：所有由input()方法获取的数据都是==“字符串”==类型

```python
name = input('请输入您的姓名：')
age = input('请输入您的年龄：')

print(type(name))  # <class 'str'>
print(type(age))  # <class 'str'>
```

