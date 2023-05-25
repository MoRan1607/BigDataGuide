### Hive的用户自定义函数实现步骤与流程

可回答：1）有写过UDF吗？如何实现UDF？2）UDF用什么语言开发的？开发完了怎么用？   

参考答案：

**1、如何构建UDF？**

用户创建的UDF使用过程如下：

第一步：继承UDF或者UDAF或者UDTF，实现特定的方法；

第二步：将写好的类打包为jar，如hivefirst.jar；

第三步：进入到Hive外壳环境中，利用add jar /home/hadoop/hivefirst.jar注册该jar文件；

第四步：为该类起一个别名，create temporary function mylength as 'com.whut.StringLength'，这里注意UDF只是为这个Hive会话临时定义的；

第五步：在select中使用mylength()。

**2、函数自定义实现步骤**

1）继承Hive提供的类

```java
org.apache.hadoop.hive.ql.udf.generic.GenericUDF 
org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
```

2）实现类中的抽象方法

3）在 hive 的命令行窗口创建函数添加 jar

```mysql
add jar linux_jar_path 	
# 创建 function
create [temporary] function [dbname.]function_name AS class_name; 
```

4）在 hive 的命令行窗口删除函数

```mysql
drop [temporary] function [if exists] [dbname.]function_name;
```

**3、自定义UDF案例**

1）需求

自定义一个UDF实现计算给定字符串的长度，例如：

```sql
hive(default)> select my_len("abcd"); 
4
```

2）导入依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.apache.hive</groupId>
		<artifactId>hive-exec</artifactId>
		<version>3.1.2</version>
	</dependency>
</dependencies>
```

3）创建一个类，继承于Hive自带的UDF

```java
/**
* 自定义 UDF 函数，需要继承 GenericUDF 类
* 需求: 计算指定字符串的长度
*/
public class MyStringLength extends GenericUDF {
	/**
	*
	*@param arguments 输入参数类型的鉴别器对象
	* @return 返回值类型的鉴别器对象
	*@throws UDFArgumentException
	*/
	@Override
	public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {
		// 判断输入参数的个数
		if(arguments.length !=1) {
			throw new UDFArgumentLengthException("Input Args Length Error!!!");
		}
// 判断输入参数的类型

		if(!arguments[0].getCategory().equals(ObjectInspector.Category.PRIMITIVE)
		  ) {
			throw new UDFArgumentTypeException(0,"Input Args Type Error!!!");
		}
//函数本身返回值为 int，需要返回 int 类型的鉴别器对象
		return PrimitiveObjectInspectorFactory.javaIntObjectInspector;
	}

	/**
	* 函数的逻辑处理
	*@param arguments 输入的参数
	*@return 返回值
	*@throws HiveException
	*/
	@Override
	public Object evaluate(DeferredObject[] arguments) throws HiveException {
		if(arguments[0].get() == null) {
			return 0;
		}
		return arguments[0].get().toString().length();
	}

	@Override
	public String getDisplayString(String[] children) {
		return "";
	}
}
```

4）打成jar包上传到服务器/opt/module/data/myudf.jar

5）将jar包添加到hive的classpath

```mysql
hive (default)> add jar /opt/module/data/myudf.jar; 
```

6）创建临时函数与开发好的java class关联

7）即可在hql中使用自定义的函数

```mysql
hive (default)> select ename,my_len(ename) ename_len from emp;
```

**欢迎加入知识星球，查阅《大数据面试题 V4.0》以及更多大数据开发内容**   
<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/Pics/%E6%98%9F%E7%90%83%E4%BC%98%E6%83%A0%E5%88%B8%20(21).png"  width="290" height="387"/>  
<p align="center">
</p>   
