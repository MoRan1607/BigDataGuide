二、Hive数据类型
---
### 1、基本数据结构
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hive%E6%96%87%E6%A1%A3Pics/%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png"/>  
<p align="center">
</p>
</p>  

&emsp; 对于Hive的String类型相当于数据库的varchar类型，该类型是一个可变的字符串，不过它不能声明其中最多能存储多少个字符，理论上它可以存储2GB的字符数。  

### 2、基本数据类型
<p align="center">
<img src="https://github.com/Dr11ft/BigDataGuide/blob/master/Pics/Hive%E6%96%87%E6%A1%A3Pics/%E9%9B%86%E5%90%88%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png"/>  
<p align="center">
</p>
</p>  

&emsp; Hive有三种复杂数据类型**ARRAY、MAP和STRUCT**。ARRAY和MAP与Java中的Array和Map类似，而STRUCT与C语言中的Struct类似，它封装了一个命名字段集合，复杂数据类型允许任意层次的嵌套。  

**案例实操**  
1）假设某表有如下一行，我们用JSON格式来表示其数据结构。在Hive下访问的格式为  
```json
{
    "name": "songsong",
    "friends": ["bingbing" , "lili"] ,       //列表Array,
    "children": {                      //键值Map,
        "xiao song": 18 ,
        "xiaoxiao song": 19
    }
    "address": {                      //结构Struct,
        "street": "hui long guan" ,
        "city": "beijing"
    }
}
```  

2）基于上述数据结构，创建测试文件test.txt，并导入到Hive表中  
```txt
songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long guan_beijing
yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing
```  

3）Hive上创建测试表test  
```mysql
create table test(
name string,
friends array<string>,
children map<string, int>,
address struct<street:string, city:string>
)
row format delimited fields terminated by ','
collection items terminated by '_'
map keys terminated by ':'
lines terminated by '\n';
```  

**`字段解释`**：  
&emsp; row format delimited fields terminated by ','&emsp; &emsp; —— 列分隔符  
&emsp; collection items terminated by '_'&emsp; &emsp; —— MAP STRUCT 和 ARRAY 的分隔符(数据分割符号)  
&emsp; map keys terminated by ':' &emsp; &emsp; —— MAP中的key与value的分隔符  
&emsp; lines terminated by '\n';&emsp; &emsp; —— 行分隔符  

4）导入文本数据到测试表  
```mysql
load data local inpath ‘/opt/module/datas/test.txt’into table test
```  

5）访问三种集合列里的数据，以下分别是ARRAY，MAP，STRUCT的访问方式  
```mysql
select friends[1],children['xiao song'],address.city from test where name="songsong";
```  





